# S3 Interview questions

### Basic Level

<details>

<summary><strong>Question 1: Access Denied Error</strong> — In production, users report "Access Denied" when accessing objects in an S3 bucket via a web app. Walk through your troubleshooting steps.</summary>

Answer: First, verify the bucket and object exist using the AWS CLI:

```bash
aws s3 ls s3://bucket-name --recursive
```

Check the IAM identity used by the app with:

```bash
aws sts get-caller-identity
```

and test permissions with the IAM policy simulator. Then examine bucket policy, ACLs, and S3 Block Public Access settings. Finally, confirm encryption settings if SSE-KMS is used — ensure the role/user has required KMS permissions (e.g., kms:Decrypt). This resolved a similar issue by fixing a missing kms:Decrypt permission.

</details>

<details>

<summary><strong>Question 2: High S3 Costs Suddenly</strong> — Your team's S3 bill spiked 3x without increased usage. How do you investigate and optimize?</summary>

Answer: Use S3 Storage Lens or Cost Explorer to break down costs by prefix, storage class, and requests. Identify unneeded Standard storage and consider lifecycle policies or Intelligent-Tiering to auto-move infrequent data. Example outcome: combined with CloudFront OAI restrictions and tiering, costs dropped from $250 to $5/month. Consider enabling Requester Pays if partners are responsible for access costs.

</details>

<details>

<summary><strong>Question 3: Enable Versioning Issue</strong> — After enabling versioning on a production bucket with 1M objects, delete operations fail. Explain why and fix.</summary>

Answer: Deletes create delete markers rather than removing object data, so object count appears to double. List versions with:

```bash
aws s3api list-object-versions --bucket my-bucket
```

Clean up using lifecycle rules such as "Permanently delete noncurrent versions after X days" and "Remove expired delete markers" to prevent storage bloat.

</details>

### Intermediate Level

<details>

<summary><strong>Question 4: Lifecycle Policy Not Transitioning</strong> — Logs in S3/logs/ prefix aren't transitioning to IA after 30 days per policy. Bucket has versioning. Troubleshoot.</summary>

Answer: Get the lifecycle config:

```bash
aws s3api get-bucket-lifecycle-configuration --bucket your-bucket
```

With versioning enabled, objects become current/noncurrent versions; ensure your lifecycle rules target current and/or noncurrent versions appropriately (e.g., transition noncurrent to IA at 30 days). Verify the rule filters the correct prefix ("logs/") and test in staging.

</details>

<details>

<summary><strong>Question 5: Cross-Account Read Access</strong> — Dev account needs read access to prod S3 bucket in another account for CI/CD. How to set up securely?</summary>

Answer: In the prod account, create an IAM role granting s3:GetObject and s3:ListBucket and a trust policy that allows the dev account to assume the role. Add a bucket policy permitting that role ARN (or use aws:SourceAccount conditions). In the dev account, let your CI role/user assume that role. Prefer cross-account roles over sharing keys.

</details>

<details>

<summary><strong>Question 6: Slow Downloads in App</strong> — Users complain of slow S3 object downloads >100MB during peak hours. Optimize performance.</summary>

Answer: Implement exponential backoff for retries. Use key prefix partitioning (e.g., date-based keys like 2026/01/13/) to distribute request load and avoid per-prefix request limits. Front S3 with CloudFront for edge caching. For large uploads/downloads, use multipart uploads and consider S3 Transfer Acceleration for global users. These steps reduced latency \~70% in past projects.

</details>

<details>

<summary><strong>Question 7: Block Public Access Override</strong> — Legacy app broke after enabling Block Public Access; it relied on public bucket policy. Fix without disabling Block Public Access.</summary>

Answer: Since Block Public Access blocks ACLs/policies that allow public access, switch to CloudFront with Origin Access Control (OAC) and update the bucket policy to allow only CloudFront (or the CloudFront OAC) to access the bucket. Add policy conditions such as "aws:SecureTransport": "true" to enforce HTTPS. This preserves Block Public Access while allowing controlled access via CloudFront.

</details>

### Advanced Level

<details>

<summary><strong>Question 8: Replication Lag in CRR</strong> — CRR setup for compliance replicates 99% objects but lags >15min on deletes during high churn. Fix.</summary>

Answer: Enable S3 Replication Time Control (RTC) to get a 15-minute SLA (99.99%) and metrics/events. For deletes, ensure "Replicate delete markers" is configured if you need deletes replicated; otherwise manage deletions via lifecycle rules. If using KMS, ensure keys are valid/symmetric in both regions. Monitor via CloudWatch and adjust replication rules/tags.

</details>

<details>

<summary><strong>Question 9: Encryption Mismatch KMS</strong> — App fails uploading to S3 with SSE-KMS due to "Access Denied" on kms:GenerateDataKey*. Prod uses custom KMS key.</summary>

Answer: Grant the IAM role kms:GenerateDataKey and kms:Decrypt in the KMS key policy (or via IAM policies referencing the key ARN). Ensure the bucket policy and upload requests align with "s3:x-amz-server-side-encryption": "aws:kms" and the correct key ARN. Audit with Access Analyzer to detect gaps.

</details>

<details>

<summary><strong>Question 10: Batch Operations Fail Throttling</strong> — S3 Batch copy job on 10M objects fails with SlowDown. Prod env has concurrent jobs. Troubleshoot.</summary>

Answer: SlowDown indicates exceeding request rate. Split the job into smaller batches (by prefix), add exponential backoff when generating manifests and during retries, and poll job status less aggressively (e.g., every 5 min instead of every few seconds). Ensure manifest CSV is UTF-8 without BOM and keys <1024 chars. Retry with \~1M object batches.

</details>

<details>

<summary><strong>Question 11: Cost Optimization Multi-Tier Logs</strong> — Processing 10TB daily logs; costs high from Standard + retrievals. Design lifecycle for optimization.</summary>

Answer: Example lifecycle rules:

* Transition >30 days to IA (or Intelligent-Tiering)
* Transition >90 days to Glacier Flexible
* Transition >365 days to Glacier Deep Archive
* Expire after 7 years

Separate small objects (<128 KB) into a different prefix for specialized handling. Use S3 Analytics/Storage Lens to refine policies. This approach saved \~60% vs Standard in a production case.

</details>

<details>

<summary><strong>Question 12: Secure Static Site Hosting</strong> — Host React app on S3 with custom domain, HTTPS, invalidate cache on deploy, restrict to CloudFront.</summary>

Answer: Host the build artifacts in an S3 bucket (no public ACLs), create a CloudFront distribution with an ACM certificate, and configure Origin Access Control (OAC) so only CloudFront can access the bucket. Use CloudFront behaviors and custom error pages to support SPA routing (serve index.html on 404). Invalidate CloudFront cache on deploy via:

```bash
aws cloudfront create-invalidation --distribution-id DIST_ID --paths "/*"
```

Block direct S3 access entirely via bucket policy.

</details>

<details>

<summary><strong>Question 13: Versioning Cleanup Explosion</strong> — Bucket object count jumped 2x after mass deletes; noncurrent versions piling up. Prod impact? Clean.</summary>

Answer: Configure lifecycle rules:

* NoncurrentVersionExpiration after 30 days
* ExpiredObjectDeleteMarker: true

For one-time cleanup, use S3 Inventory + S3 Batch Delete (manifest from inventory CSV). Query inventory with Athena to plan deletes; this avoided large bills in a production case.

</details>

### Additional Scenarios

<details>

<summary><strong>Question 14: VPC Endpoint for S3 Access</strong> — EC2 in private subnet can't reach S3; public internet blocked. Costs rising from NAT Gateway.</summary>

Answer: Create a VPC Gateway Endpoint for S3 and add it to the subnet route tables (free). Use an endpoint policy to restrict access (e.g., condition "aws:SourceVpce"). For KMS, create Interface Endpoints if needed. This reduced NAT Gateway egress costs significantly.

</details>

<details>

<summary><strong>Question 15: Multipart Upload Cleanup</strong> — Prod ETL leaves orphaned multipart uploads, consuming space. How to detect/clean?</summary>

Answer: Use a lifecycle rule:

```xml
<AbortIncompleteMultipartUpload>
  <DaysAfterInitiation>1</DaysAfterInitiation>
</AbortIncompleteMultipartUpload>
```

Use S3 Inventory + Athena to find multipart uploads older than X days and then S3 Batch Abort for large-scale cleanup. This reclaimed substantial space in production.

</details>

<details>

<summary><strong>Question 16: S3 Analytics for Perf</strong> — High latency in EU region for game data. Use analytics to optimize.</summary>

Answer: Enable S3 Storage Class Analysis to export usage patterns and identify infrequently accessed data. Transition those objects to cheaper storage classes or use regional caching (CloudFront) to reduce latency. Observed \~50% reduction in peak latency after changes.

</details>

<details>

<summary><strong>Question 17: Requester Pays for Sharing</strong> — Third-party submits 1TB data to your bucket daily, high transfer costs. Mitigate.</summary>

Answer: Enable Requester Pays on the bucket so requesters pay GET/PUT costs, and document requirements for partners. Combine with Transfer Acceleration if partners need faster uploads. This shift reduced your monthly bill significantly.

</details>

<details>

<summary><strong>Question 18: Object Lock Compliance</strong> — Regulatory req: logs immutable 7y for WORM. Implement for new/existing.</summary>

Answer: Enable Object Lock in the bucket and use GOVERNANCE mode for testing, then set default retention to 7 years. For existing objects, use S3 Batch Copy with object lock retention options to apply retention. Use Compliance mode after audits to prevent deletions even by root.

</details>

<details>

<summary><strong>Question 19: Metrics Alarm for Throttling</strong> — Prod app throttled on S3 PUTs during spikes. Proactive monitoring?</summary>

Answer: Create CloudWatch alarms on ThrottledRequests metrics (e.g., >5 in 5 min) and send notifications via SNS/PagerDuty. Parse access logs with metric filters for 503 SlowDown. Combine alarms with application changes (prefix partitioning) to mitigate throttling.

</details>

<details>

<summary><strong>Question 20: Disaster Recovery Test Fail</strong> — CRR to DR region lags; test restore fails on encrypted objects. Fix.</summary>

Answer: Use symmetric multi-region KMS keys and ensure replication grants are present in both regions. Enable Replication Time Control (RTC) for SLA and test via controlled chaos (delete subset in source and verify destination within 15 min). Consider MFA Delete for critical data if required.

</details>

***

References / Further reading:
