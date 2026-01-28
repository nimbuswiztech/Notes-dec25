# cicd flow

I've worked on various Jenkins pipelines to streamline CI/CD processes within our organization. Given the different range of applications, I've created CI/CD pipelines to suit the specific needs of each application. These pipelines incorporate build tools such as Maven, Sonar, Docker, Tomcat and others, ensuring seamless integration and deployment workflows.

{% stepper %}
{% step %}
### CI process — stages

* Check out the source code in the Jenkins workspace.
* Perform the Sonar scan; if the scan result is successful, the build will proceed with other activities in the job; otherwise, abort the job.
* Create the binaries (war, docker image, .exe) according to the build tools used.
* Publish the binaries (war, docker image, .exe) to the JFrog Artifactory.

Once these activities are completed, the CI part is complete and the CD part will start. The CI job will initiate the CD job from the post-build section, triggering the CD job with parameters.
{% endstep %}

{% step %}
### CD job for deploying to Kubernetes clusters

* Checkout the Git repository that contains the manifest file.
* Update the manifest file with the latest image tag.
* Login to the Kubernetes cluster with an IAM user.
* Deploy the application using kubectl commands on the cluster.
* Verify the deployment.
* If deployment fails:
  * Check application logs and try to fix the issue.
  * If unable to fix, roll back to the previous version.
{% endstep %}

{% step %}
### CD job for deploying to Tomcat (Ansible playbook tasks)

The Ansible playbook contains the following tasks:

* Download the binaries to the Tomcat server and extract them.
* Stop the Tomcat service.
* Take a backup of existing binaries and configuration files.
* Copy the latest binaries, configuration files and dependencies to the targeted path.
* Start the Tomcat service and verify the deployment.
* If deployment fails:
  * Examine application logs to identify and address issues.
  * If possible, fix the problem; otherwise, roll back to the previous version.
{% endstep %}
{% endstepper %}

***

### Kubernetes deployment steps

Jenkinsfile to define a pipeline for deploying a Spring Boot application to an EKS cluster using Jenkins.

The pipeline should include the following high-level steps:

* Checkout the Git repository
* Build a JAR
* Build a Docker image
* Push the image to ECR
* Integrate Jenkins with the EKS cluster
* Deploy the app to EKS

To use this in Jenkins, select “Pipeline script” under the pipeline section and specify the necessary steps.

{% code title="Jenkinsfile" %}
```
```
{% endcode %}

If you look in detail, the pipeline contains five stages. The following stepper breaks down each stage and its content.

{% stepper %}
{% step %}
### Checkout

* Retrieves code from a Git repository.
* Specifies the repository URL and branch (example uses `main`).
* Code is checked out to the Jenkins workspace for subsequent stages.
{% endstep %}

{% step %}
### Maven Build (Build Jar)

* Cleans previous build artifacts:
  * sh 'mvn clean'
* Builds the JAR:
  * sh 'mvn package'
* Produces an executable JAR used later in the pipeline.

Note: JARs may be rebuilt when code changes or for new releases.
{% endstep %}

{% step %}
### Docker Image Build

* Builds a Docker image using the Dockerfile in the project:
  * sh 'docker build -t \<IMAGE\_NAME> .'
* The -t option names the image; the build context is the current directory.
{% endstep %}

{% step %}
### Push Docker Image to ECR

* Authenticate and push the image to Amazon ECR using the AWS CLI:
  * aws ecr get-login-password --region \<AWS\_REGION> | docker login --username AWS --password-stdin \<ECR\_REGISTRY\_ID>
  * docker tag \<IMAGE\_NAME>:latest \<ECR\_REGISTRY\_ID>/\<IMAGE\_NAME>:latest
  * docker push \<ECR\_REGISTRY\_ID>/\<IMAGE\_NAME>:latest
* Uses AWS credentials configured in Jenkins (e.g., withAWS block).
{% endstep %}

{% step %}
### Integrate Jenkins with EKS and Deploy

* Update kubeconfig to connect to the EKS cluster:
  * sh 'aws eks update-kubeconfig --name \<EKS\_CLUSTER\_NAME> --region \<AWS\_REGION>'
* Deploy resources to the cluster:
  * sh "kubectl apply -f \<K8S\_DEPLOY\_FILE>.yaml"
* Uses Jenkins’ AWS integration to run the commands securely.
{% endstep %}
{% endstepper %}

***

#### Trigger the pipeline

Start the pipeline by clicking “Build Now” in the Jenkins Dashboard.

![](<../../.gitbook/assets/AD_4nXfafcTB0_kJamzesOW4G8TZKBjBsZB2sC3piBw9ewRg1utizqOLINZDNpbamr2uiXEKbdBMs8UDC QCnf26Rzk7WoNn3a2KySUaDfOmT60vgzhs44 uUTpCBPlJ44RnjngAnvCJeDjH490kbTCj8kt1hvwC>)

#### Monitor the pipeline

Monitor pipeline progress and view build logs to identify issues.

![](<../../.gitbook/assets/AD_4nXfIUeT_P2Qblfc4tDwXz8oRP5gwfiShvm0hRtP_2_15AxJ3BchhsxK kjWd4t7qRrEpvao8cezjSUepwo9KbXcTe6GL0ZfYcoxlBbWUWZjzjyzLAzlYtSFiLpgF99Y_geldcu3t_3vIPyGmVGLmy8HaB o>)

***

#### Interact with a cluster from terminal

Retrieve the status of an EKS cluster:

`aws eks describe-cluster --region <region-name> --name <cluster-name> --query cluster.status`

#### Update the kubeconfig file

The kubeconfig file manages communication between kubectl and the cluster. Update it with:

`aws eks --region <region-name> update-kubeconfig --name <cluster-name>`

#### Retrieve data from the cluster

Common kubectl commands:

`kubectl get nodes`\
`kubectl get pods`\
`kubectl get services`\
`kubectl get deployments`

#### Expose the service

To make a service accessible outside the cluster, expose it as a LoadBalancer. If your service is not accessible yet, change the service type to LoadBalancer before deploying.

![](<../../.gitbook/assets/AD_4nXdRV5WRxJ42v0VFJJ0B2M0zVGIhszxRxiDS6SbxsuTqVfDDlW4qsLmejgBeHqCV6RlecDU0fYh_9ErzfhyLetLyT45UeUlpHD8zG2CqY 0JoyYMI2QlCDx7mjnj_Q_hOtaVqviiUGsYbMNGhqwPTmYwNJQ0>)

#### Service type LoadBalancer

Change your Service type to LoadBalancer before deploying to make it accessible from the internet.

#### Get the external IP

Run:

`kubectl get svc`

This will list services and include the external IP assigned to LoadBalancer-type services.

![](<../../.gitbook/assets/AD_4nXdP6ES6gYRfprOVj0DQzmu8cPcgdM1dY55zXToipcahwmMi2uRUgAAVD9W3miA qzwmVTXXekulWJgb3sm0Oiyr_cPmV q6ry3weE__FqG_Bl4waCihJI4jTHFm6D0yc4CKzlCwwtdUzmvamjSLq0pfAoA>)

#### Example

Open a web browser and enter: : — ensure the port is open and accessible.

#### Allow the required ports

Configure the security group associated with the worker nodes in the EKS cluster to allow incoming traffic on the port used by the service.
