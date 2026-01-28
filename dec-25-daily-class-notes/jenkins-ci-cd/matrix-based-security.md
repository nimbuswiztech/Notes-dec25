# matrix based security

Matrix-based security allows Jenkins administrators to assign specific permissions per user or group using a matrix layout. Each row is a user or group, and each column is a permission like:

| User/Group | Job                          | Overall      | Agent       | Credentials |
| ---------- | ---------------------------- | ------------ | ----------- | ----------- |
| admin      | ✅ Build ✅ Configure ✅ Delete | ✅ Administer | ✅ Configure | ✅ Create    |
| developer  | ✅ Build ✅ Read               | ❌            | ❌           | ❌           |
| viewer     | ✅ Read                       | ❌            | ❌           | ❌           |

***

✅ Step-by-step demo setup

{% stepper %}
{% step %}
### Enable Jenkins Security

* Go to: **Manage Jenkins → Configure Global Security**
*   Under **Access Control → Authorization**, select:

    ✅ **Matrix-based security**
{% endstep %}

{% step %}
### Add Users (or Groups)

Under the matrix:

* Click **"Add user or group"**
* Add the following:
  * `admin`
  * `developer`
  * `viewer`
{% endstep %}

{% step %}
### Assign Permissions

demo permission setup:

| User      | Overall | Job Read | Job Build | Job Configure | Job Delete | Credentials |
| --------- | ------- | -------- | --------- | ------------- | ---------- | ----------- |
| admin     | ✅ All   | ✅        | ✅         | ✅             | ✅          | ✅           |
| developer | ❌       | ✅        | ✅         | ✅             | ❌          | ❌           |
| viewer    | ❌       | ✅        | ❌         | ❌             | ❌          | ❌           |

{% hint style="info" %}
Explain to students: each permission column gives access to specific Jenkins functions.
{% endhint %}
{% endstep %}
{% endstepper %}

***

🔬 Real-time testing scenario

{% stepper %}
{% step %}
#### Log in as `admin`

* Full access to everything.
* Can create, configure, delete jobs.
* Can access system configuration.
{% endstep %}

{% step %}
#### Log in as `developer`

* Can view and build jobs.
* Cannot delete or create new credentials.
{% endstep %}

{% step %}
#### Log in as `viewer`

* Can only view jobs and build history.
* Cannot build or configure.
{% endstep %}
{% endstepper %}

***

✅ Jenkinsfile for testing access

{% code title="Jenkinsfile" %}
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo "Building app..."
            }
        }
        stage('Deploy') {
            steps {
                echo "Deploying to dev environment..."
            }
        }
    }
}
```
{% endcode %}

***

Per-project security

{% stepper %}
{% step %}
Go to **Manage Jenkins → Configure Global Security**
{% endstep %}

{% step %}
Check **"Enable project-based security"**
{% endstep %}

{% step %}
Inside each job → **Configure → Enable project-based security**
{% endstep %}
{% endstepper %}
