# github webhook

## Run a Jenkins job with GitHub webhook

### Pre-requisites

* A Jenkins Server [Click here for help](https://github.com/siddeshpm1/DevOps/blob/main/jenkins)
* A GitHub repository [Click here for help](https://github.com/siddeshpm1/DevOps/blob/main/jenkins)

### Integration Steps

Log into the Jenkins.

{% stepper %}
{% step %}
### Create a Freestyle Project

* Create a new job
  * Job Name: `WebhookTestJob`
  * Source code management
    * Git URL: [get URL here](https://github.com/yankils/hello-world.git)
  * Build Triggers
    * Enable: `GitHub hook trigger for GITScm polling`
* Build
  * Click `Add Build Step` → `Invoke Top Level Maven Targets`
    * Maven Version: `Maven`
    * Goals: `clean install`
{% endstep %}

{% step %}
### Configure the GitHub repository webhook

On your GitHub repository:

* Go to `Settings` → `Webhooks`
* Add a webhook:
  * Payload URL: `http://<Jenkins_IP>:8080/github-webhook/`
  * Content type: `application/json`
{% endstep %}

{% step %}
### Trigger the job

* Update the repository (push a new commit). This should trigger the Jenkins job via the webhook.
{% endstep %}
{% endstepper %}
