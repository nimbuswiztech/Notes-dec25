# HTTPD Scenarios

{% stepper %}
{% step %}
### Sudden 403s after deploy

Production users get 403 on `/static/*` right after a config change; app was fine 5 minutes ago. What do you check first?

**Answer:** Check `error_log` for messages like “client denied by server configuration” and confirm any `<Directory>` / `<Location>` rules didn’t tighten (e.g., `Require all denied`). Validate filesystem permissions and SELinux context (if enabled). Confirm no new rewrite rule is sending requests into a restricted path. If the change came from a recent config deploy, roll back via config version history if needed. \[^3]
{% endstep %}

{% step %}
### 404 for existing file behind rewrite

`/health` returns 404 but the file exists; there’s a rewrite setup for the app. How do you debug quickly?

**Answer:** Temporarily increase rewrite logging with `LogLevel rewrite:trace3` for that vhost, reproduce once, then revert (to avoid noisy logs). Confirm `DocumentRoot`, `Alias`, and rewrite rule ordering; often a catch-all rule routes `/health` to the upstream app instead of serving the static file. Add an early exception rule for `/health` so the health check remains static and local. \[^4]
{% endstep %}

{% step %}
### Too many open connections / slow clients

During peak, CPU is low but connections are high and users see slowness. What Apache settings do you check?

**Answer:** Check `KeepAlive`, `KeepAliveTimeout`, and `MaxKeepAliveRequests` because long keep-alives with slow clients can consume workers. Validate MPM limits (`MaxRequestWorkers`, `ServerLimit`) against instance memory/CPU. Use `server-status` (restricted) to inspect worker states (e.g., “W”/“R”/“K”) and tune accordingly. \[^6]\[^5]
{% endstep %}

{% step %}
### TLS handshake failures after enabling HTTPS

Some clients can’t connect after a “security hardening” change. What’s the likely cause and fix?

**Answer:** Likely older clients were blocked by tightened `SSLProtocol`/`SSLCipherSuite`. Check `ssl.conf` for `SSLProtocol` and `SSLCipherSuite` settings. Decide policy: internet-facing apps can require TLSv1.2+, while legacy internal clients may need a temporary allowance with a deprecation plan. Test with `openssl s_client` from representative client environments. \[^7]\[^1]
{% endstep %}

{% step %}
### Sensitive headers leaking

Security reports server version/banner is visible and TRACE is enabled. What do you change?

**Answer:** Disable TRACE and minimize information leakage: `TraceEnable off`, set `ServerTokens Prod`, and `ServerSignature Off`. Apply per-vhost if needed and validate with `curl -I` and a basic scanner. Ensure changes are consistent across all vhosts to avoid surprises. \[^7]
{% endstep %}

{% step %}
### Need basic auth for an admin path

Expose `/admin` only to a small on-call group quickly, without touching the application. How?

**Answer:** Implement Apache-level authz/authn in the vhost: add a `<Location "/admin">` block with `Require` rules backed by LDAP/OIDC (preferred) or `htpasswd` for emergency access. Add source-IP restrictions (VPN CIDR) as an extra layer. Ensure access attempts are logged for audit. \[^2]
{% endstep %}
{% endstepper %}

{% stepper %}
{% step %}
### 502 Bad Gateway from reverse proxy

Apache is a reverse proxy to Tomcat/Node; users see 502 intermittently under load. How do you troubleshoot and stabilize?

**Answer:** Correlate timestamps between Apache `error_log` and upstream logs; 502 typically means invalid/empty response from upstream or connection issues. Check `ProxyTimeout`/`Timeout`, upstream keepalive settings, and backend saturation (thread pools, GC). Implement health checks and proper balancer configuration so Apache marks failing nodes and fails over instead of repeatedly sending traffic. \[^9]\[^8]
{% endstep %}

{% step %}
### Large file uploads failing via proxy

Uploads >1GB fail with 502/503 through Apache reverse proxy, but smaller files work. What do you adjust?

**Answer:** Check proxy buffering/timeouts and backend max body size. Increase `ProxyTimeout` and any backend-specific upload limits, verify network path timeouts (ALB/NLB idle timeouts if using AWS), and confirm any fronting proxies aren’t truncating requests. For scale and cost, consider offloading large uploads to object storage (pre-signed URLs) to reduce Apache load. \[^10]\[^4]
{% endstep %}

{% step %}
### Performance tuning: prefork vs event

A legacy LAMP app uses mod\_php and Apache is memory heavy; what’s your production approach?

**Answer:** If mod\_php is in use you may need prefork MPM, but it’s memory-intensive. Short-term: lower `MaxRequestWorkers` to avoid swapping and right-size instances. Long-term: migrate PHP to PHP-FPM and switch to event/worker MPM for better concurrency and lower memory per connection. Measure memory per worker before and after to justify the change. \[^5]\[^11]
{% endstep %}

{% step %}
### Hotlinking and bandwidth costs

Traffic costs spike; logs show many image requests from third-party domains embedding your assets. How do you mitigate?

**Answer:** Prefer offloading static assets to a CDN (CloudFront) with signed URLs/cookies. At Apache level: implement referer-based controls (best-effort), caching headers to reduce repeat hits, and WAF/rate-limiting to block abusive clients. Analyze access logs by referer and user-agent to validate impact. \[^2]
{% endstep %}

{% step %}
### Need reliable log retention (“durability”)

Compliance requires access logs retained 1 year and searchable; disk fills up. What’s your solution?

**Answer:** Rotate and compress logs locally (logrotate) and ship them off-host to durable storage like S3 or to a logging cluster (OpenSearch) via an agent (Fluent Bit, Vector). Keep short local retention (7–14 days). Enforce centralized retention and integrity controls (e.g., S3 Object Lock) if compliance requires append-only retention. \[^12]
{% endstep %}

{% step %}
### Troubleshooting worker saturation

Users see intermittent 503, and `top` shows Apache processes but requests hang. How do you prove it’s worker exhaustion?

**Answer:** Expose a restricted `mod_status` and inspect the scoreboard: all workers busy and a growing request queue indicate exhaustion. Tune MPM (`MaxRequestWorkers`, `ServerLimit`) based on memory headroom, reduce `KeepAliveTimeout`, and find/fix slow upstream calls causing worker hold. Add dashboards and alerts on busy worker counts and request durations. \[^5]\[^6]
{% endstep %}

{% step %}
### Multi-tenant isolation (“cross-account”)

One Apache hosts multiple customer vhosts; a misconfig exposed another tenant’s content. How do you design isolation?

**Answer:** Enforce strict per-vhost `DocumentRoot` and `<Directory>` permissions; avoid shared writable paths. For strong isolation, run separate Apache instances/containers per tenant or at minimum separate Unix users and filesystem ACLs. Implement automated config validation in CI (lint + integration tests) before reloads. \[^2]
{% endstep %}
{% endstepper %}

{% stepper %}
{% step %}
### WAF rollout without breaking traffic

Need WAF-like protection quickly for OWASP Top 10 attacks, but must avoid false positives. What’s your plan?

**Answer:** Deploy ModSecurity in detection-only mode initially, feed audit logs into SIEM, and tune exclusions for legitimate traffic patterns. Progressively enable blocking per path/vhost using vetted rule sets (OWASP CRS). Keep a rollback mechanism and a documented incident runbook for rapid rule adjustments. \[^14]\[^13]
{% endstep %}

{% step %}
### Sticky sessions broken behind Apache balancer

Users randomly lose sessions after scaling app servers. How do you fix and verify stickiness?

**Answer:** Configure `mod_proxy_balancer` with a stable route (cookie-based or URL parameter) and ensure the app preserves that cookie. Increase balancer logging to observe route assignments and check backend session timeouts that can force rebalancing. For robust scaling, move session state to a shared store (Redis) instead of relying solely on stickiness. \[^16]\[^15]
{% endstep %}

{% step %}
### Zero-downtime config changes (“versioning”)

How do you roll out Apache config changes with safety, auditability, and no outage?

**Answer:** Keep configs in Git with PR reviews, run `apachectl configtest` in CI, and perform a canary rollout to one node before fleet-wide change. Use `apachectl graceful` (or `graceful` reload) to avoid dropping connections and validate with synthetic checks. Tag releases so rollback is a simple revert+redeploy. \[^2]
{% endstep %}

{% step %}
### End-to-end encryption and re-encryption boundaries

In AWS, TLS terminates at ALB but compliance requires encryption to the instance too. What do you do?

**Answer:** Terminate TLS at the ALB and also enable HTTPS on Apache (TLS between ALB and instance). Manage certificates via ACM on ALB and automate instance certificate management (e.g., with a cert manager) or use TLS passthrough with an NLB if appropriate. Validate ciphers and protocols and test with `openssl s_client` from within the VPC. Document the termination points and rationale. \[^1]
{% endstep %}

{% step %}
### Incident: sudden spike in 401/403 from one region

After a CloudFront/ALB change, only one geo/ISP sees auth failures. How do you troubleshoot?

**Answer:** Correlate request IDs across CloudFront/ALB/Apache and compare headers (Host, X-Forwarded-Proto, Authorization). Check any auth flow dependent on forwarded headers or host/scheme—common root causes include missing `ProxyPreserveHost` or dropped `X-Forwarded-*` headers breaking redirect/callback URLs. Apply a quick config fix, then add regression tests for forwarded headers. \[^4]
{% endstep %}

{% step %}
### Defense against slowloris / abusive clients

Attackers open many slow connections and exhaust workers, but traffic volume isn’t huge. What controls do you apply?

**Answer:** Harden connection handling: lower `KeepAliveTimeout`, set a reasonable global `Timeout`, and use the event MPM if possible for better connection scalability. Enable `mod_reqtimeout` and/or `mod_evasive` to limit slow or abusive clients. In AWS, combine with WAF at ALB/CloudFront for L7 filtering. Monitor connection states via `mod_status` to confirm improvements. \[^6]\[^5]\[^2]
{% endstep %}

{% step %}
### Cost optimization: right-sizing and caching strategy

Apache fleet cost is high; most traffic is static assets and repeated API responses. What’s your production optimization approach?

**Answer:** Offload static assets to S3+CloudFront to remove them from Apache traffic, reducing instance counts. Add caching headers and edge caching for safe dynamic responses, enable compression where appropriate, and instrument APM/access logs to ensure correctness. Validate cost savings by comparing request volume and instance scaling before and after the change. \[^2]
{% endstep %}
{% endstepper %}

References and further reading:

* https://httpd.apache.org/docs/2.4/ssl/ssl\_howto.html
* https://httpd.apache.org/docs/2.4/misc/security\_tips.html
* https://wetopi.com/error-code-403-forbidden/
* https://squareops.com/blog/comprehensive-guide-to-http-errors-in-devops-causes-scenarios-and-troubleshooting-steps/
* https://www.goranstimac.com/blog/apache-performance-tuning-configuring-mpm-directives/
* https://support.site24x7.com/portal/en/kb/articles/configure-status-url-apache
* https://docs.miarec.com/admin-guide/security/security-hardening-for-apache-web-server/
* https://stackoverflow.com/questions/169453/bad-gateway-502-error-with-apache-mod-proxy-and-tomcat
* https://www.digitalocean.com/community/tutorials/how-to-troubleshoot-common-http-error-codes
* https://lists.apache.org/thread/m7lh83xh7po68f20fwxlhsp6xkrz2902
* https://www.infiflex.com/optimize-web-server-performance-by-tuning-apache
* https://httpd.apache.org/docs/current/programs/log\_server\_status.html
* https://www.digitalocean.com/community/tutorials/how-to-set-up-mod\_security-with-apache-on-debian-ubuntu
* https://github.com/owasp-modsecurity/ModSecurity
* https://httpd.apache.org/docs/2.4/mod/mod\_proxy\_balancer.html

If you prefer a strictly “Apache-only” set (no AWS edge services like ALB/CloudFront) I can regenerate these to focus purely on httpd modules/config/runtime on EC2 or on-prem.
