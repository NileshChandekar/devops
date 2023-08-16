### What is Jenkins 

### Jenkins Master 

### Jenkins Slave

### Installation of Jenkins Master:- 

* Setup a VM first:
* Using Vagrat I am creating the environment
  * Assumption:-
  * Libvirt Servcie
  * Libvirt-Vagrant Plugin
  * Vagrant Service is installed and running
* Run the below ``vagrant`` script to create the environemnt.
```
mkdir /cicdproject/vagrant/
cd /cicdproject/vagrant/
mkdir jenkins
cd jenkins
```
```
root@816484-logging01:/cicdproject/vagrant/jenkins# ls -lhrt 
total 8.0K
-rw-r--r-- 1 root root 440 Aug  1 12:01 bootstrap.sh
-rw-r--r-- 1 root root 863 Aug  1 12:03 Vagrantfile
root@816484-logging01:/cicdproject/vagrant/jenkins#
```
```
root@816484-logging01:/cicdproject/vagrant/jenkins# cat Vagrantfile 
# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_NO_PARALLEL'] = 'yes'
VAGRANT_BOX         = "generic/ubuntu2004"
VAGRANT_BOX_VERSION = "4.2.10"
CPUS_JENKINS_NODE    = 2
MEMORY_JENKINS_NODE  = 1024
JENKINS_NODES_COUNT  = 1

Vagrant.configure(2) do |config|
  config.vm.provision "shell", path: "bootstrap.sh"
  (1..JENKINS_NODES_COUNT).each do |i|
    config.vm.define "jenkins#{i}" do |node|
      node.vm.box               = VAGRANT_BOX
      node.vm.box_check_update  = false
      node.vm.box_version       = VAGRANT_BOX_VERSION
      node.vm.hostname          = "jenkins#{i}.example.com"
      node.vm.network "private_network", ip: "192.168.100.10#{i}"
  
    node.vm.provider :libvirt do |v|
      v.memory  = MEMORY_JENKINS_NODE
      v.nested  = true
      v.cpus    = CPUS_JENKINS_NODE
    end
  end  
 end
end
root@816484-logging01:/cicdproject/vagrant/jenkins#
```
```
root@816484-logging01:/cicdproject/vagrant/jenkins# cat bootstrap.sh 
#!/bin/bash


# Enable ssh password authentication
echo "[TASK 2] Enable ssh password authentication"
sed -i 's/^PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config

# Enable root ssh login
echo "[TASK 3] Enable ssh root login"
sed -i 's/.*PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config

systemctl reload sshd

# Set Root password
echo "[TASK 4] Set root password"
echo -e '0\n0\n' | sudo passwd root

root@816484-logging01:/cicdproject/vagrant/jenkins#
```
```diff
+ vagrant up
```

```
root@816484-logging01:/cicdproject/vagrant/jenkins# vagrant status
Current machine states:

jenkins1                  running (libvirt)

root@816484-logging01:/cicdproject/vagrant/jenkins#
```

```diff
+ vagrant ssh
```

```
root@jenkins1:~# apt install openjdk-17-jdk/focal-updates -y
root@jenkins1:~# wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
root@jenkins1:~# sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
root@jenkins1:~# apt update -y
root@jenkins1:~# apt install jenkins docker.io -y
```

```
root@jenkins1:~# systemctl status jenkins.service
● jenkins.service - Jenkins Continuous Integration Server
     Loaded: loaded (/lib/systemd/system/jenkins.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2023-08-03 15:17:59 UTC; 1 weeks 5 days ago
   Main PID: 16511 (java)
      Tasks: 59 (limit: 1065)
     Memory: 417.4M
     CGroup: /system.slice/jenkins.service
             └─16511 /usr/bin/java -Djava.awt.headless=true -jar /usr/share/java/jenkins.war --webroot=/var/cache/jenkins/war --httpPort=8080
```

```
root@jenkins1:~# systemctl status docker
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-08-15 13:52:39 UTC; 18h ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 104761 (dockerd)
      Tasks: 11
     Memory: 35.6M
     CGroup: /system.slice/docker.service
             └─104761 /usr/bin/dockerd -H tcp://0.0.0.0:4243 -H unix:///var/run/docker.sock
```
             
```
root@jenkins1:~# netstat -planetx | grep -i 8080
```

* Admin Password is here:-

```
root@jenkins1:~# cat /var/lib/jenkins/secrets/initialAdminPassword
```

* Complete the basic configuration which is required.


### Installation of Jenkins Slave:- 

* We will create the ``SLAVE`` node as a container,
* Build the Dockerfile first:-
* On the same Master Jenkins node I am creating the Jenkins Slave node as a container.

```
root@jenkins1:~# mkdir slavejenkins
root@jenkins1:~# cd slavejenkins
root@jenkins1:~/slavejenkins#
```

```
root@jenkins1:~/slavejenkins# cat Dockerfile 
# Use the official Ubuntu base image
FROM ubuntu:20.04
USER root
# Install required tools
RUN apt-get update && \
    apt-get install -y  wget curl && \
    apt-get clean 
# Add the Jenkins repository key
RUN curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null

# Add the Jenkins repository
RUN echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | tee /etc/apt/sources.list.d/jenkins.list > /dev/null

# Install OpenSSH server, Jenkins, and Docker
RUN apt-get update && \
    apt-get install -y openssh-server && \
    apt-get install docker.io -y && \
    apt-get install openjdk-11-jre -y && \
    apt-get install jenkins -y && \
    usermod -aG docker jenkins && \
    mkdir /var/run/sshd
# Set root password
RUN  echo 'root:0' | chpasswd
# Allow root login via SSH and enable password authentication
RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config && \
    sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
# Expose SSH port
EXPOSE 22
EXPOSE 8080
# Script for entrypoint
COPY docker-entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-entrypoint.sh
# Start SSH service when the container starts
ENTRYPOINT ["docker-entrypoint.sh"]
root@jenkins1:~/slavejenkins# cd slavejenkins/
```

```
root@jenkins1:~/slavejenkins# cat docker-entrypoint.sh 
#!/bin/bash
/usr/sbin/sshd  -D &
java -jar /usr/share/java/jenkins.war &
root@jenkins1:~/slavejenkins# 
```

```
root@jenkins1:~/slavejenkins# docker build -t jenkins-with-docker .
root@jenkins1:~/slavejenkins# docker tag jenkins-with-docker:latest  nileshc85/jenkins-with-docker:20.04
root@jenkins1:~/slavejenkins# docker push nileshc85/jenkins-with-docker:20.04
```

### Configuration:-

* Dashboard
  * Manage Jenkins
    * Nodes and Clouds
      * Cloud ### On your left side cornet

[!image](https://github.com/NileshChandekar/devops/blob/main/Jenkins/images/Screenshot%202023-08-16%20at%201.43.29%20PM.png)
[!image]()
[!image]()
[!image]()


### Working Pipeline 

* A Working Github account
* Jenkins installation.
* Credentials, for DockerHub
* Credentials for Git

```
pipeline {
  environment {
    registry = "nileshc85/cicd-test-1"
    registryCredential = 'jenkins-auth-docker1'
    dockerImage = ''
  }
  
  agent any

    
    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }


        stage("Clone Git Repository") {
            steps {
                git(
                    url: "http://gitlab.example.com/root/np-1.git",
                    branch: "main",
                    changelog: true,
                    poll: true
                )
            }
        }
        
            stage('Building image') {
              steps{
                script {
                  dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
              }
            }
        
        stage('Deploy Image') {
          steps{
             script {
                docker.withRegistry( '', registryCredential ) {
                dockerImage.push()
              }
            }
          }
        }
        
        stage('Remove Unused docker image') {
          steps{
            sh "docker rmi $registry:$BUILD_NUMBER"
          }
        }
        
        stage('Run Script') {
            steps {
                sh 'chmod +x updatemanifest.sh && ./updatemanifest.sh'
            }
        }

        stage('Git Push') {
            steps {
                sh 'git add .'
                sh 'git commit -m "push to git $BUILD_NUMBER"'
                sh 'git push http://root:haIvVP+K7nXhr0+I8gK6VxYtWFtaWkM7kOkenHhrsGs=@gitlab.example.com/root/np-1.git main'
            }
        }
        
        
    }
}
```
