# excercices

## Attaching and Expanding an EBS Volume on an EC2 Instance

{% hint style="info" %}
Main Takeaway\
You can seamlessly add storage to a running EC2 instance by attaching a new EBS volume and then resizing the partition and filesystem—all without downtime.
{% endhint %}

{% stepper %}
{% step %}
### Prerequisites

* An existing EC2 instance in the desired Availability Zone.
* AWS CLI configured with sufficient IAM permissions (ec2:DescribeVolumes, ec2:AttachVolume, ec2:ModifyVolume).
* SSH access to the EC2 instance as a user with sudo privileges.
{% endstep %}

{% step %}
### Attaching an Additional EBS Volume

#### Create a New EBS Volume (if not already created)

```
aws ec2 create-volume \
  --region us-east-1 \
  --availability-zone us-east-1a \
  --size 20 \
  --volume-type gp3
```

Note the resulting `VolumeId` (e.g., `vol-0abcd1234efgh5678`).

#### Attach the Volume to the EC2 Instance

```
aws ec2 attach-volume \
  --volume-id vol-0abcd1234efgh5678 \
  --instance-id i-0123456789abcdef0 \
  --device /dev/xvdf
```

* Replace `vol-…` and `i-…` with your volume and instance IDs.
* On Nitro instances, the device may appear as `/dev/nvme1n1`; check with `lsblk` after attaching.
{% endstep %}

{% step %}
### Preparing and Mounting the New Volume on Linux

#### Verify Attachment

```
sudo lsblk
```

You should see a new device (e.g., `xvdf` or `nvme1n1`) without partitions.

#### Create a Filesystem

```
sudo mkfs -t xfs /dev/xvdf
```

(Use `ext4` if preferred: `sudo mkfs.ext4 /dev/xvdf`.)

#### Mount the Volume

```
sudo mkdir /data
sudo mount /dev/xvdf /data
```

#### Persist Mount Across Reboots

1.  Retrieve the UUID:

    ```
    sudo blkid /dev/xvdf
    ```
2.  Edit `/etc/fstab` and add:

    ```
    UUID=<your-uuid>  /data  xfs  defaults,nofail  0  2
    ```
{% endstep %}

{% step %}
### Expanding an Existing EBS Volume

#### Modify the Volume Size

```
aws ec2 modify-volume \
  --volume-id vol-0abcd1234efgh5678 \
  --size 40
```

Monitor until the modification state is `completed`:

```
aws ec2 describe-volumes-modifications \
  --volume-id vol-0abcd1234efgh5678 \
  --filters Name=modification-state,Values=completed
```

#### Extend Partition (if applicable)

1.  Check for partition on the volume:

    ```
    lsblk
    ```
2.  If there’s a partition (e.g., `/dev/xvdf1`), grow it:

    ```
    sudo growpart /dev/xvdf 1
    ```

(Install `cloud-utils` if `growpart` is unavailable.)

#### Extend the Filesystem

For XFS:

```
sudo xfs_growfs /data
```

For ext4:

```
sudo resize2fs /dev/xvdf1
```

#### Verify the New Size

```
df -hT /data
```
{% endstep %}

{% step %}
### Summary of Commands

| Task                   | Command Snippet                                            |
| ---------------------- | ---------------------------------------------------------- |
| Create EBS volume      | `aws ec2 create-volume … --size 20 …`                      |
| Attach volume          | `aws ec2 attach-volume … --device /dev/xvdf`               |
| Verify block devices   | `sudo lsblk`                                               |
| Format filesystem      | `sudo mkfs -t xfs /dev/xvdf`                               |
| Mount volume           | `sudo mount /dev/xvdf /data`                               |
| Persist in fstab       | Add `UUID=… /data xfs defaults,nofail 0 2` to `/etc/fstab` |
| Modify volume size     | `aws ec2 modify-volume … --size 40`                        |
| Grow partition         | `sudo growpart /dev/xvdf 1`                                |
| Grow XFS filesystem    | `sudo xfs_growfs /data`                                    |
| Grow ext4 filesystem   | `sudo resize2fs /dev/xvdf1`                                |
| Verify filesystem size | `df -hT /data`                                             |

Attaching and resizing EBS volumes in AWS can be accomplished without downtime by leveraging the AWS CLI and Linux filesystem tools. Following these steps ensures your EC2 instance seamlessly gains additional storage and adapts to increased capacity requirements.
{% endstep %}
{% endstepper %}
