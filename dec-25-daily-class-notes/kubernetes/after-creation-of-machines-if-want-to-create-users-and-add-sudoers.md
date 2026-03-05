# After creation of machines, if want to create users and add sudoers

{% stepper %}
{% step %}
### Create a new user and set its password

Run:

```
useradd user_name
```

Then set the password:

```
passwd user_name
```
{% endstep %}

{% step %}
### Grant the user sudo privileges

Open the sudoers file (for example with vi):

```
vi /etc/sudoers
```

Add this line to the file:

```
user_name    ALL=(ALL:ALL) ALL
```
{% endstep %}

{% step %}
### Enable password authentication for SSH

On many cloud images, password authentication is disabled by default. Edit the SSH daemon config:

```
vi /etc/ssh/sshd_config
```

Change the line:

```
PasswordAuthentication no
```

to:

```
PasswordAuthentication yes
```
{% endstep %}

{% step %}
### Restart the SSH service

Run:

```
systemctl restart sshd
```
{% endstep %}

{% step %}
### Resize GKE cluster nodes

Run the gcloud command to resize the cluster:

```
gcloud container clusters resize cluster-1 --num-nodes=0 --zone us-central1-a --quiet
```
{% endstep %}
{% endstepper %}
