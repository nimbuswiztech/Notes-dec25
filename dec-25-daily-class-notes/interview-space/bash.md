# Bash

#### Basic Level Questions (Questions 1-7)

<details>

<summary><strong>Question 1: In a production EC2 instance, you're seeing high CPU usage from a cron job running a Bash script. How do you identify the script and debug it without stopping the cron?</strong></summary>

Answer: First, check active cron jobs with `crontab -l` for the user or `sudo crontab -l -u root`. For system-wide, grep `/var/log/cron` or `/var/log/syslog` for the job PID. Use `top -p $(pgrep -f script_name)` or `ps aux | grep script` to find the process. Tail the script's log if it outputs one, or strace the PID with `strace -p PID -o trace.log` to see stuck syscalls. In AWS, check CloudWatch Logs if integrated, then edit crontab to add logging like `*/5 * * * * /path/script.sh >> /var/log/myscript.log 2>&1`.

</details>

<details>

<summary><strong>Question 2: How do you securely read AWS credentials in a Bash script running on an EC2 instance without hardcoding them?</strong></summary>

Answer: Use IAM roles for EC2—attach a role with least-privilege policies, then `aws sts get-caller-identity` verifies it. In the script, export `AWS_PROFILE=default` or rely on instance metadata: `curl -s http://169.254.169.254/latest/meta-data/iam/security-credentials/ROLE_NAME`. Never hardcode; for temp creds, use `aws configure get` if profile-based. Best practice: script checks `if [ -z "$AWS_ACCESS_KEY_ID" ]; then echo "Use IAM role"; fi`.

</details>

<details>

<summary><strong>Question 3: Your Bash script fails intermittently with "Argument list too long" when grepping a massive log file in /var/log. What's the cause and fix?</strong></summary>

Answer: It's `Argument list too long` from `execve` limit when passing huge file lists to grep. Don't use `grep pattern $(find /var/log -name "*.log")`; instead, pipe find: `find /var/log -name "*.log" -exec grep -l pattern {} +` or `find ... | xargs grep pattern`. For single huge file, use `grep -a pattern hugefile.log` if binary. In prod, rotate logs with logrotate to avoid this.

</details>

<details>

<summary><strong>Question 4: Explain how you'd loop through S3 bucket objects listed via AWS CLI in a Bash script and delete old ones.</strong></summary>

Answer: Use `aws s3api list-objects-v2 --bucket mybucket --query 'Contents[?LastModified<\`date -d '30 days ago' -Iseconds\`].Key' --output text`to get keys, then loop:`while IFS= read -r key; do aws s3 rm s3://mybucket/$key; done < <(aws s3api ... )`. Handle pagination with` --max-items\` or script a loop with ContinuationToken. Enable versioning first; use Lifecycle policies in AWS for automation over scripts.

</details>

<details>

<summary><strong>Question 5: In Linux, a script needs root but you're non-root. How do you run it elevated securely in CI/CD like Jenkins on EKS?</strong></summary>

Answer: Use `sudo` in script with NOPASSWD in sudoers: `username ALL=(ALL) NOPASSWD: /path/to/script.sh`. In Jenkins/EKS, use service accounts or pods with `securityContext: privileged: false, runAsUser: 1000`. Better: Docker ENTRYPOINT with capabilities or AWS SSM Run Command. Script starts with `if [[ $EUID -ne 0 ]]; then exec sudo "$0" "$@"; fi`.

</details>

<details>

<summary><strong>Question 6: How do you make a Bash script idempotent for deploying configs to multiple EC2s via SSH?</strong></summary>

Answer: Check existence before acting: `if [[ ! -f /etc/myapp.conf ]] || ! grep -q "key=value" /etc/myapp.conf; then echo "key=value" >> /etc/myapp.conf; fi`. For multi-host, use `parallel-ssh -H hosts.txt -l ec2-user 'bash -s' < deploy.sh`. In Ansible/Terraform, prefer modules, but for Bash, hash file contents and compare MD5.

</details>

<details>

<summary><strong>Question 7: Your script parses JSON from `aws ec2 describe-instances` but fails on large output. Fix it.</strong></summary>

Answer: AWS CLI paginates; use `--no-paginate --output text --query 'Reservations[*].Instances[*].[InstanceId,State.Name]'`. Parse with `jq`: `aws ec2 describe-instances --query '...' --output json | jq -r '.[] | select(.State.Name=="running") | .InstanceId'`. Install jq via `apt`; for no jq, use `grep -oP '(?<=i-)[a-z0-9]{17,}'`.

</details>

***

#### Intermediate Level Questions (Questions 8-14)

<details>

<summary><strong>Question 8: Troubleshoot a Bash script in prod that's hanging on `scp` to an EKS node during CI/CD artifact deploy.</strong></summary>

Answer: Check netstat/ss for timeouts: `ss -tuln | grep :22`. Script trap: `trap 'echo "Timeout"; exit 1' SIGTERM; timeout 30s scp file user@node:/path`. Logs: `scp -v`. In EKS, use `kubectl cp` instead: `kubectl cp artifact pod:/path`. AWS SSM for secure copy without keys; increase ulimit -n if many files.

</details>

<details>

<summary><strong>Question 9: Write a snippet to monitor disk usage on EKS worker nodes and alert via SNS if >80%.</strong></summary>

Answer:

```
#!/bin/bash
nodes=$(kubectl get nodes -o jsonpath='{.items[*].metadata.name}')
for node in $nodes; do
  usage=$(kubectl exec -n kube-system ds/daemonset-node-exporter --df -h / | awk 'NR==2{print $5}' | sed 's/%//')
  if (( usage > 80 )); then
    aws sns publish --topic-arn arn:aws:sns:region:account:alerts --message "Node $node: $usage%"
  fi
done
```

Run via cron; use Prometheus for prod scale.

</details>

<details>

<summary><strong>Question 10: In a Jenkins pipeline Bash step, handle secret from AWS Secrets Manager without logging it.</strong></summary>

Answer: Use `aws secretsmanager get-secret-value --secret-id mysecret --query SecretString --output text` and pipe to var: `secret=$(aws ... )`; export masked in Jenkins `withCredentials`. Mask in console: Jenkins `maskPasswords()`. Secure: IAM role on Jenkins EC2, rotate secrets. Script: `if aws secretsmanager ... >/dev/null; then read -s secret; fi`.

</details>

<details>

<summary><strong>Question 11: Your script processes Terraform outputs post-apply, but `terraform output -json` changes format. Make it robust.</strong></summary>

Answer: Use jq for stability: `outputs=$(terraform output -json) instance_id=$(echo $outputs | jq -r '.instance_id.value') echo $instance_id | aws ec2 tag-resources --resource-arn-list arn:aws:ec2:... --tags Key=Name,Value=Prod` Handle nulls: `jq -r '.instance_id.value // empty'`. Version pin Terraform; use tfoutput tool.

</details>

<details>

<summary><strong>Question 12: During EKS upgrade, a script restarts DaemonSets but some pods OOMKilled. Debug and fix.</strong></summary>

Answer: `kubectl get events --sort-by='.lastTimestamp' | grep OOMKilled`. Script: `kubectl rollout restart ds/my-daemonset; kubectl rollout status ds/my-daemonset --timeout=300s`. Check limits: `kubectl describe pod | grep Limits`. Add to script: `kubectl top pods; kubectl logs pod -c container --previous`. AWS: Use Cluster Autoscaler, set resource requests properly.

</details>

<details>

<summary><strong>Question 13: Automate cleanup of old EBS snapshots >90 days across regions using Bash.</strong></summary>

Answer:

```
for region in us-east-1 us-west-2; do
  aws ec2 describe-snapshots --owner-ids $(aws sts get-caller-identity --query Account --output text) --region $region --query 'Snapshots[?StartTime<`date -d "90 days ago" -Iso`].SnapshotId' --output text | xargs -n1 -I {} aws ec2 delete-snapshot --snapshot-id {} --region $region
done
```

Dry-run with `--query`. Use AWS Backup policies for compliance.

</details>

<details>

<summary><strong>Question 14: A Bash script deploys Helm charts but fails on Tiller TLS in EKS. Modern fix?</strong></summary>

Answer: Helm 3 is tillerless. Script: `helm upgrade --install myrelease chart/ --set image.tag=$TAG --kubeconfig /path --namespace prod`. Verify: `helm list -n prod`. Trap errors: `helm upgrade ... || { echo "Failed"; exit 1; }`. In CI, use `helm repo add` securely with OIDC.

</details>

***

#### Advanced Level Questions (Questions 15-20)

<details>

<summary><strong>Question 15: Implement a Bash function for blue-green deploy on ECS with zero downtime, handling health checks.</strong></summary>

Answer:

```
deploy_blue_green() {
  blue=tg-blue; green=tg-green
  aws elbv2 create-target-group ... # if new
  aws ecs update-service --cluster prod --service app --task-definition app:$REVISION --load-balancers TargetGroupArn=arn:tg-green
  aws elbv2 modify-rule --rule-arn arn:rule --actions Type=forward,TargetGroupArn=arn:tg-green
  sleep 60  # health check
  aws elbv2 deregister-targets --tg-arn arn:tg-blue ...
  aws elbv2 delete-target-group --tg-arn arn:tg-blue
}
```

Test traffic shift with weights first.

</details>

<details>

<summary><strong>Question 16: Troubleshoot Bash script race condition in parallel Jenkins stages updating shared S3 state file.</strong></summary>

Answer: Use atomic ops: `aws s3 cp s3://bucket/state.json /tmp/state.json; edit; aws s3 cp /tmp/state.json s3://bucket/state.json --metadata-directive REPLACE`. Better: S3 versioning + DynamoDB for locks: `aws dynamodb update-item --table locks --key '{"id":{"S":"deploy-lock"}}' --update-expression "SET expires=..." --condition-expression "attribute_not_exists(id)"`. Script retries with exponential backoff.

</details>

<details>

<summary><strong>Question 17: Write a script to scan Docker images in ECR with Trivy, fail CI if high vulns, integrate with GitHub Actions.</strong></summary>

Answer:

```
#!/bin/bash
repo=ecr-repo; tag=latest
aws ecr get-login-password | docker login ...
docker pull $AWS_ACCOUNT.dkr.ecr.region.amazonaws.com/$repo:$tag
trivy image --exit-code 1 --no-progress --severity HIGH,CRITICAL $repo:$tag
aws inspector2 start-image-scan --resource-scan-arn arn:...
```

In GH Actions: `if [ $? -ne 0 ]; then echo "::error::Vulns found"; exit 1; fi`.

</details>

<details>

<summary><strong>Question 18: In multi-region DR, script fails over RDS read replicas to primary with minimal RPO. How?</strong></summary>

Answer: Promote replica: `aws rds promote-read-replica --db-instance-identifier replica-id`. Update Route53: `aws route53 change-resource-record-sets --hosted-zone-id Z1 --change-batch file://failover.json`. Pre-script: check lag `aws rds describe-db-instances --db-instance-identifier replica --query ReplicaLag`. RPO <1min with multi-AZ; script verifies `aws rds wait db-instance-available`.

</details>

<details>

<summary><strong>Question 19: Optimize Bash script looping 10k Lambda invocations via CLI for cost in burst traffic.</strong></summary>

Answer: Batch with `--cli-input-json` and `xargs`:

```
seq 1 10000 | xargs -P10 -n100 aws lambda invoke --function-name handler --payload file://batch.json --invocation-type Event
```

Parallel with GNU parallel. Cost: use Provisioned Concurrency. Monitor: `aws logs filter-log-events --log-group /aws/lambda/handler --filter-pattern "ERROR" | jq`. Prefer Step Functions for orchestration.

</details>

<details>

<summary><strong>Question 20: Secure a Bash script for GitOps ArgoCD sync with webhook secrets, handling token rotation.</strong></summary>

Answer:

```
secret=$(aws secretsmanager get-secret-value --secret-id argocd-token --query SecretString --output text)
curl -H "Authorization: Bearer $secret" -X POST https://argocd.example.com/api/v1/applications/myapp/sync

# Rotate:
aws secretsmanager put-secret-value --secret-id ... --secret-string "$(curl -s ... | jq -r .token)"
```

Use OIDC for ArgoCD; script validates JWT expiry: `exp=$(echo $secret | jwt decode ... )`; if <5min, rotate. Audit logs to CloudTrail.

</details>
