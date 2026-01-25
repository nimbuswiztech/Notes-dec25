# Basic Git and GitHub Scenarios

<details>

<summary>Question 1: In a team CI/CD pipeline on GitHub, a developer accidentally pushes a secret API key to the main branch. How do you handle this immediately in production?</summary>

Answer:\
First, rotate the secret right away via AWS Secrets Manager or wherever it's stored to prevent exposure. Then, on GitHub, use the web UI to edit the commit and remove the secret—go to the repo, click the commit, edit the file, and commit directly to main. Follow up by adding a .gitignore for that file type and enable GitHub's secret scanning. To force-push a clean history if needed:

{% code title="Force-remove last commit from main" %}
```bash
git reset --hard HEAD~1 && git push --force-with-lease origin main
```
{% endcode %}

In production, this minimizes downtime since secrets are rotated first.

</details>

<details>

<summary>Question 2: Your team uses GitHub for a monorepo with 50 microservices. A junior dev creates a branch with 100 commits but wants to squash them before PR. What's your command and why?</summary>

Answer:\
Use interactive rebase to squash:

{% code title="Interactive rebase to squash last 100 commits" %}
```bash
git rebase -i HEAD~100
```
{% endcode %}

Mark all but the last as "squash" or "fixup". This keeps a clean linear history, which is easier for reviewers in a monorepo where noisy histories slow down PR merges. Push with --force-with-lease to avoid overwriting others. Example push:

```
```

</details>

<details>

<summary>Question 3: During a hotfix on production branch, Git complains about divergent histories. How do you resolve without losing changes?</summary>

Answer:\
Fetch latest and inspect history:

```
```

Then rebase your branch onto production:

```
```

git rebase production

## resolve conflicts, then:

git add git rebase --continue

```
{% endcode %}

If rebase is messy, abort and merge instead:

{% code %}
git rebase --abort
git merge --no-ff production
{% endcode %}

Push with --force-with-lease if you rewrote history. This keeps history clean for GitHub Actions triggers in prod deploys.
</details>

---

### Intermediate Git and GitHub Scenarios

<details>
<summary>Question 4: GitHub Actions workflow fails because a PR from a fork can't access a private AWS ECR repo. How do you fix securely?</summary>

Answer:  
Forks don't inherit secrets for security. Use GitHub OIDC to grant short-lived AWS credentials: configure an IAM role trusting GitHub's OIDC provider (sts:AssumeRoleWithWebIdentity) and use audience sts.amazonaws.com. In the workflow, use `aws-actions/configure-aws-credentials` with role-to-assume. Add branch protection and required approvals to limit risky merges.
</details>

<details>
<summary>Question 5: Team hits Git LFS lock issues on large Docker images in repo during parallel PRs. Real-time fix?</summary>

Answer:  
Unlock via GitHub UI: Settings > Manage LFS locks, or CLI:

<div data-gb-custom-block data-tag="code">

git lfs unlock <file>

</div>

Prevent with branch protection rules requiring status checks. For production, migrate heavy assets to GitHub Packages or AWS S3 and use references (symlinks). Example tracking in repo:

<div data-gb-custom-block data-tag="code">

git lfs track "*.tar"

</div>

Lock files during builds to avoid conflicts.
</details>

<details>
<summary>Question 6: Recover a deleted GitHub branch used in a merged PR after 90 days—repo is 10GB. Steps?</summary>

Answer:  
Branches may be restorable via API or CLI. Example with gh:

<div data-gb-custom-block data-tag="code">

gh api /repos/:owner/:repo/git/refs/heads/deleted-branch --method PATCH -F ref=refs/heads/recovered

</div>

Or use the API restore endpoints. Locally, fetch refs:

<div data-gb-custom-block data-tag="code">

git fetch origin '+refs/heads/*:refs/remotes/origin/*'

</div>

For large repos, use a shallow clone first to reduce data transfer.
</details>

<details>
<summary>Question 7: In GitHub, a PR shows conflicts from upstream changes. How do you resolve in CLI for team review?</summary>

Answer:  
Checkout the PR branch:

<div data-gb-custom-block data-tag="code">

gh pr checkout 123
git fetch origin main
git rebase main
# or:
git merge main

</div>

Resolve conflicts, then:

<div data-gb-custom-block data-tag="code">

git add .
git commit --no-edit   # or --amend if appropriate
git push --force-with-lease

</div>

Enables a tidy PR history and keeps reviewers focused.
</details>

<details>
<summary>Question 8: Prod deploy via GitHub Actions tags fail due to duplicate annotated tags. How to overwrite safely?</summary>

Answer:  
Annotated tags are effectively immutable—delete remotely first, then recreate:

<div data-gb-custom-block data-tag="code">

git tag -d v1.0
git push origin --delete v1.0
git tag -a v1.0 -m "Release"
git push origin v1.0

</div>

Use GPG-signed tags for security and protect tag creation via repo rules.
</details>

---

### Advanced Git and GitHub Scenarios

<details>
<summary>Question 9: Monorepo with 200 devs hits GitHub rate limits on API calls from CI during peak hours. Mitigation?</summary>

Answer:  
Mitigations include:
- Use GitHub Enterprise or self-hosted runners closer to your cloud region to reduce latency.
- Implement exponential backoff and retries (e.g., curl --retry).
- Use the gh CLI or GraphQL API for more efficient queries.
- Cache API responses in Redis to reduce repeated calls.
</details>

<details>
<summary>Question 10: Troubleshoot Git repo corruption in GitHub mirror after AWS S3 outage—clone fails with "packfile corrupt".</summary>

Answer:  
Try a shallow clone to get a usable repo:

<div data-gb-custom-block data-tag="code">

git clone --depth 1 --no-tags <url>
git fsck --full

</div>

If mirror is corrupt, open a support ticket with GitHub and restore from backups (GitHub Archive, AWS backups, or EFS mirrors).
</details>

<details>
<summary>Question 11: Secure GitHub repo for AWS IAM roles with OIDC in multi-org setup—avoid key rotation hell.</summary>

Answer:  
Add GitHub as an OIDC provider in IAM. Create roles with conditions restricting token use to specific repositories and workflows (e.g., requiring repository claim). Use short-lived tokens via OIDC in workflows instead of long-lived access keys.
</details>

<details>
<summary>Question 12: Git history rewrite in shared repo—how to notify team and sync without breaking pipelines?</summary>

Answer:  
After rewrite (git filter-branch or BFG), push with --force-with-lease. Tag or record old commit hashes, notify the team (Slack/webhook), and update CI pipelines to fetch with forced updates. Prefer pinning pipelines to tags rather than mutable branches after a rewrite.
</details>

<details>
<summary>Question 13: GitHub Dependabot alerts flood for 500 deps in monorepo. Production-grade handling?</summary>

Answer:  
Use dependabot configuration to group updates (e.g., .github/dependabot.yml groups: security-updates). Auto-merge safe minor/patch updates. Use GitHub Advanced Security and integrate additional scanners (Trivy) to gate PRs before merge.
</details>

<details>
<summary>Question 14: Migrate SVN to GitHub with 5-year history, preserving authors/blame for AWS infra code.</summary>

Answer:  
Use svn2git:

<div data-gb-custom-block data-tag="code">

svn2git file:///svn/repo --authors authors.txt
git push --mirror <github-url>

</div>

Provide an authors.txt mapping to preserve commit attribution. Clean up empty commits with interactive rebase where needed.
</details>

<details>
<summary>Question 15: Cherry-pick a commit across 10 branches in GitHub for hotfix, handling conflicts efficiently.</summary>

Answer:  
Use cherry-pick with an automation loop:

<div data-gb-custom-block data-tag="code">

git cherry-pick -x <commit-hash>
# Script example:
for branch in $(git branch -r | grep hotfix); do
  git checkout $branch
  git cherry-pick -x <commit-hash>
  # resolve conflicts if any, commit, push, etc.
done

</div>

Open PRs per branch for review. You can automate with a GitHub Actions composite action to apply the same patch across branches.
</details>

---

### Expert Troubleshooting and Architecture Scenarios

<details>
<summary>Question 16: GitHub Pages deploy fails with "API rate limit exceeded" from large static site gen in Actions.</summary>

Answer:  
Use a self-hosted runner (e.g., EC2 Spot) for heavy builds, minimize GITHUB_TOKEN permissions, and cache repo artifacts. Consider deploying the static site to AWS S3 + CloudFront instead of Pages for scale and lower cost.
</details>

<details>
<summary>Question 17: Detect and block large files >100MB in GitHub pre-push hook for prod repo.</summary>

Answer:  
Client-side pre-push hook example:

<div data-gb-custom-block data-tag="code">

# .git/hooks/pre-push
git rev-list --objects --bytes | awk '$2 && $3 > 100*1024*1024 {print}'

</div>

Also rely on GitHub's server-side limits, and integrate pre-commit frameworks (husky) to enforce locally.
</details>

<details>
<summary>Question 18: Implement GitOps with GitHub + FluxCD for EKS—handle drift detection in prod.</summary>

Answer:  
Bootstrap Flux:

<div data-gb-custom-block data-tag="code">

flux bootstrap github --owner=org --repository=clusters --branch=main --path=./clusters/prod

</div>

Flux watches repo paths for Kustomize/HelmReleases. Use flux reconcile and enable webhooks for automated reconciliation. Restrict Flux permissions with RBAC to minimize blast radius.
</details>

<details>
<summary>Question 19: Repo fork bomb via shallow clones in CI—optimize for 1000+ PRs/day.</summary>

Answer:  
Avoid full clones in CI. Use:
- actions/checkout@v4 with fetch-depth: 1
- git clone --no-checkout --filter=blob:none + sparse-checkout
Only run full clones on merge or when necessary. This reduces build time and storage.
</details>

<details>
<summary>Question 20: Audit GitHub org for SSH keys with high perms post-breach—remediate at scale.</summary>

Answer:  
Use the gh API to list and revoke deploy keys:

<div data-gb-custom-block data-tag="code">

gh api /orgs/org/deploy-keys --paginate | jq
gh api -X DELETE /repos/repo/keys/{id}

</div>

Migrate to per-repo deploy keys or OIDC, enforce 2FA and required code reviews. Scripted audits help at scale during incident response.
</details>
```

</details>
