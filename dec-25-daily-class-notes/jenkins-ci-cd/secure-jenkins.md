# secure jenkins

{% stepper %}
{% step %}
### Enable Global Security

To secure Jenkins, enable global security.

Steps:

* Go to Manage Jenkins → Configure Global Security
* Enable “Enable Security”
* Choose an authentication method:
  * Jenkins own user database
  * LDAP / Active Directory
  * GitHub OAuth / Google SSO (via plugin)
{% endstep %}

{% step %}
### Use Role-Based Access Control (RBAC)

Best Practice: Use Matrix-based or Role Strategy plugin

* Install **Role-based Authorization Strategy** plugin
* Define roles like:
  * **admin**: all permissions
  * **developer**: limited to job creation and build
  * **viewer**: read-only

{% hint style="warning" %}
Disable anonymous access: remove all permissions from the “anonymous” user.
{% endhint %}
{% endstep %}

{% step %}
### Use Credentials Plugin Securely

Do:

* Store secrets (API keys, tokens, SSH keys) in **Jenkins Credentials Store**
* Use `withCredentials` block in Jenkinsfile

{% code title="Jenkinsfile" %}
```groovy
withCredentials([string(credentialsId: 'my-token', variable: 'TOKEN')]) {
    sh 'curl -H "Authorization: Bearer $TOKEN" https://api.prod.com/deploy'
}
```
{% endcode %}

{% hint style="danger" %}
Don’t hardcode secrets in pipeline scripts or environment variables.
{% endhint %}
{% endstep %}

{% step %}
### Secure Jenkins URL and Network

* Use HTTPS:
  * Run Jenkins behind Nginx/Apache with SSL
  * Use Let’s Encrypt or internal SSL certificate
* Firewall rules:
  * Only allow Jenkins access from trusted IPs or VPN
  * Block public access to port 8080
{% endstep %}

{% step %}
### Restrict Script and Plugin Execution

* Sandbox Groovy scripts:
  * Enforce Groovy sandbox for non-admins
  * Review scripts in In-Process Script Approval
* Update plugins regularly:
  * Manage Jenkins → Plugin Manager → Updates
  * Uninstall unused plugins
{% endstep %}

{% step %}
### Monitoring, Auditing & Backups

Install and configure:

* Audit Trail Plugin → tracks UI actions
* Monitoring Plugin → view JVM memory, threads
* ThinBackup Plugin or back up /var/lib/jenkins/

Log important events:

* Configure jenkins.log
* Monitor who logs in, what builds are run, config changes, etc.
{% endstep %}

{% step %}
### Disable Unused Features

| Feature                  | Disable via                            |
| ------------------------ | -------------------------------------- |
| CLI                      | Global Security → Uncheck “Enable CLI” |
| Remoting/Agent Protocols | `jenkins.args` or security settings    |
| Script Console           | Restrict to admin only                 |
{% endstep %}

{% step %}
### Harden Jenkins on OS Level

| OS Hardening Step          | Description                               |
| -------------------------- | ----------------------------------------- |
| Run as a **non-root** user | Use dedicated `jenkins` user              |
| Limit file permissions     | Jenkins home should not be world-readable |
| Secure `jenkins.service`   | Use systemd to restrict system access     |
{% endstep %}

{% step %}
### Regular Maintenance & Alerts

* Set up Slack or Email notifications for failed builds or login attempts
* Rotate credentials regularly
* Periodically audit users and permissions
{% endstep %}
{% endstepper %}

***

<details>

<summary>Summary Checklist</summary>

| Task                                        | Status |
| ------------------------------------------- | ------ |
| \[ ] Enable authentication and RBAC         | ✅      |
| \[ ] Disable anonymous access               | ✅      |
| \[ ] Use credentials plugin (no hardcoding) | ✅      |
| \[ ] Set up HTTPS and reverse proxy         | ✅      |
| \[ ] Install Audit Trail & Monitoring       | ✅      |
| \[ ] Regularly update Jenkins/plugins       | ✅      |
| \[ ] Limit who can run Groovy/Script        | ✅      |
| \[ ] Backup Jenkins (e.g., ThinBackup)      | ✅      |

</details>
