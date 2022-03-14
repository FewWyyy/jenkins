# Installation and Configuration Jenkins

Prerequisites
Minimum hardware requirements:
•	256 MB of RAM
•	1 GB of drive space (although 10 GB is a recommended minimum if running Jenkins as a Docker container)
Recommended hardware configuration for a small team:
•	4 GB+ of RAM
•	50 GB+ of drive space

# Install Jenkins
```
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
sudo yum upgrade
sudo yum install epel-release java-11-openjdk-devel
sudo yum install jenkins
sudo systemctl daemon-reload
```
Start Jenkins

You can start the Jenkins service with the command:
```
sudo systemctl start jenkins
```
You can check the status of the Jenkins service using the command:
```
sudo systemctl status jenkins
```
Unlocking Jenkins
When you first access a new Jenkins instance, you are asked to unlock it using an automatically-generated password.
1.	Browse to http://localhost:8080 (or whichever port you configured for Jenkins when installing it) and wait until the Unlock Jenkins page appears.
 
2.	From the Jenkins console log output, copy the automatically-generated alphanumeric password (between the 2 sets of asterisks).
 
Note:
o	The command: sudo cat /var/lib/jenkins/secrets/initialAdminPassword will print the password at console.
o	If you are running Jenkins in Docker using the official jenkins/jenkins image you can use sudo docker exec ${CONTAINER_ID or CONTAINER_NAME} cat /var/jenkins_home/secrets/initialAdminPassword to print the password in the console without having to exec into the container.
3.	On the Unlock Jenkins page, paste this password into the Administrator password field and click Continue.
Notes:
o	You can always access the Jenkins console log from the Docker logs (above).
o	The Jenkins console log indicates the location (in the Jenkins home directory) where this password can also be obtained. This password must be entered in the setup wizard on new Jenkins installations before you can access Jenkins’s main UI. This password also serves as the default admininstrator account’s password (with username "admin") if you happen to skip the subsequent user-creation step in the setup wizard.

 
Jenkins Kubernetes
1.	Install Kubernetes plugins
2.	Use Credential secrets file
3.	Add credential in Jenkins with kube-config file
4.	Run Command kubectl in Jenkins file
## see syntax in pipeline syntax in Jenkins


Use kube-config helper Node with ServiceAccount 

	Use Service Account Token 
```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: < /etc/kubernetes/pki/ca.crt>
    server: https://10.9.98.46:6443
  name: qas1
contexts:
- context:
    cluster: qas1
    user: jenkins
  name: jenkins@qas1
current-context: jenkins@qas1
kind: Config
preferences: {}
users:
- name: jenkins
  user:
    token: <kubectl describe secrets ### user secrets>
```

Config Master and Slave

เครื่อง Master คือเครื่องที่ติดตั้ง Jenkins ไว้ ส่วนเครื่อง Slave ไม่ต้องติดตั้ง Jenkins
เริ่มจากการสร้าง Add User “Jenkins” ที่เครื่อง Slave ไม่จำเป็นต้องใช้ชื่อ Jenkins
เข้าไปที่ Web UI ของ Jenkins 
เข้าไปที่ menu “Manage Jenkins”  “Manage Credential”  “Add credential”
 

ใช้เป็น Username และ Password
 
 
หลังจากนั้นกลับไปที่ Menu “Manage Jenkins”  “Manage Node and  Clouds”  “New Node”
แล้วใส่ข้อมูลของเครื่อง Slave เข้าไป

 

 
Run Pipeline

Create Pipeline เข้าไปที่ Pipeline syntax  เลือกเป็น With kubeconfig
Example pipeline script
```
pipeline {
    agent {label "Slave1"}
    stages {
        stage('Hello') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'JenkinsK8s', namespace: '', serverUrl: 'https://192.168.1.74:6443') {
    script {sh "kubectl create deployment hello --image=busybox"}
                }
            }
        }
    }
}
```


 
Best Practice Jenkins pull File from Gitlab and Apply to k8s Cluster
Env : Use Jenkins pull file from Gitlab  Run Command Apply file to k8s
Create Pipeline Project
Use “Script from SCM”
Create Credential ( GitLab Username: password )
Create JenkinsFile ใส่ไว้ที่ GitLab
 
 
 

Example Jenkins file
```
pipeline {
    agent {label "Slave1"}

    stages {
        stage('Hello') {
            steps {
                sh 'pwd'
                sh 'ls'
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'JenkinsK8s', namespace: '', serverUrl: 'https://192.168.1.74:6443') {
                script {sh "kubectl apply -f deploy.yml"}
                }
            }
        }
    }
}
```
 
# Setup HA Jenkins
 
Install GitLab
Ref: Download and install GitLab | GitLab
Install and configure the necessary dependencies
sudo yum install -y curl policycoreutils-python openssh-server perl
# Enable OpenSSH server daemon if not enabled: sudo systemctl status sshd
```
sudo systemctl enable sshd
sudo systemctl start sshd
```

# Check if opening the firewall is needed with: sudo systemctl status firewalld
```
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo systemctl reload firewalld
```
```
sudo yum install postfix
sudo systemctl enable postfix
sudo systemctl start postfix
```

Add the GitLab package repository and install the package
```
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash
```

Next, install the GitLab package. Make sure you have correctly set up your DNS, and change https://gitlab.example.com to the URL at which you want to access your GitLab instance. Installation will automatically configure and start GitLab at that URL.
For https:// URLs, GitLab will automatically request a certificate with Let's Encrypt, which requires inbound HTTP access and a valid hostname. You can also use your own certificate or just use http:// (without s).
If you would like to specify a custom password for the initial administrator user (root), check the documentation. If a password is not specified, a random password will be automatically generated.
```
sudo EXTERNAL_URL="https://gitlab.example.com" yum install -y gitlab-ee
```
Browse to the hostname and login
Unless you provided a custom password during installation, a password will be randomly generated and stored for 24 hours in /etc/gitlab/initial_root_password. Use this password with username root to login.
See our documentation for detailed instructions on installing and configuration.

Jenkins to GitLab
Ref: GitLab | Jenkins plugin
Jenkins-to-GitLab authentication
PLEASE NOTE: This auth configuration is only used for accessing the GitLab API for sending build status to GitLab. It is not used for cloning git repos. The credentials for cloning (usually SSH credentials) should be configured separately, in the git plugin.
This plugin can be configured to send build status messages to GitLab, which show up in the GitLab Merge Request UI. To enable this functionality:
1.	Create a new user in GitLab
2.	Give this user 'Maintainer' permissions on each repo you want Jenkins to send build status to
3.	Log in or 'Impersonate' that user in GitLab, click the user's icon/avatar and choose Settings
4.	Click on 'Access Tokens'
5.	Create a token named e.g. 'jenkins' with 'api' scope; expiration is optional
6.	Copy the token immediately, it cannot be accessed after you leave this page
7.	On the Global Configuration page in Jenkins, in the GitLab configuration section, supply the GitLab host URL, e.g. https://your.gitlab.server
8.	Click the 'Add' button to add a credential, choose 'GitLab API token' as the kind of credential, and paste your GitLab user's API key into the 'API token' field
9.	Click the 'Test Connection' button; it should succeed

 
Jenkins SonarQube
1.	Install Jenkins plugins “SonarQube Scanner for Jenkins”
2.	Config Sonarqube server in Jenkins manage server “Configure System ” 
3.	Config Sonarqube in Global tools Configuration 
4.	Write JenkinsFile
Example Jenkins File
```
pipeline {
    agent {label "master"}
    stages {
        stage('Code scan') {
            environment {
                scannerHome = tool 'SonarQube'# ชื่อ Tools ที่ Install ไว้ที่ Global tools
            }
            steps {
                withSonarQubeEnv('SonarQube'#ชื่อ Server SonarQube) {
                    sh "${scannerHome}/bin/sonar-scanner " +
                        "-Dsonar.scm.disabled=True " +
                        "-Dsonar.projectKey=hello " +
                        "-Dsonar.projectName=hello " +
                        "-Dsonar.java.libraries=target/*.jar " +
                        "-Dsonar.sourceEncoding=UTF-8 " +
                        "-Dsonar.sources=. "# Path Source Code “Current is workspace path” +
                        "-Dsonar.java.binaries=. " +
                        '-Dsonar.exclusions="node_modules/**, k8s/**, .git/**, .vscode/**, .mvn/**"'
                }
            }
        }
        stage('Hello') {
            steps {
                sh 'pwd'
                sh 'ls'
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'JenkinsK8s', namespace: '', serverUrl: 'https://192.168.1.74:6443') {
                script {sh "kubectl apply -f deploy.yml"}
                }
            }
        }
    }
}
```


