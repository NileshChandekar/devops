### GitOps - ArgoCD

* Single source of truth to deliver application and infrastructre. 
* With use of source code repository we will have a full track of user as well what changes made in cluster. 
* you will put a declarative manifest in gitrepo 
* after commit, someone from the senior resource will verify and merge the code in the main branch. 
* and once the main branch is updated, argocd will pull it and measure the actaul and desired state and will make the change accordingly. 

User---->Git[KubernetesYaml]--->K8s[ArgoCD]

* Management of application is little crucuial 
* So you need a way to manage the application. 
* Declarative
* Version COntrol 
* Fully Automate application deployment and lifecycle management 

Summarized form of ArgoCD:- Is:- 

* Open-Source gitops tool for kubernetes app delivery
* Uses git repos as a single source of truth for app config. 
* Automate deployment, sync states, detect drift. 
* Offer web UI, Cli, API for easy management. 
* Simplefies kubernetes app deployment and tracking. 

### Pre-Req:- 
* You should be aware of:- 

a) Git repo
b) Containers app
d) Kubernetes Cluster
d) Docker 

*NOTE* ArgoCD is build on top of Docker and Kubernetes. 

### Intro to GitOps. 

* GitOps is a DevOps methodology utilizing the Git repo for automated, version-control application and infrastructure deployment. 

* An operative model for cloud native applications. 
* Monitoring of container deployment. 
* Apply CI/CD practives for both code as well as infrastructure management. 

Operations -----> Developer ----> Code Repo----> CI ----> Container registry ----> State repo

### Git Repo:- 

* Configuration files for infrastructure or manifest files are stored here, 
* Define the desired end state of the application. 

### ArgoCD for CI of infrastructure. 

Developer --<write the code and commit to git repo> Git repo <CI Pipeline> Docker registry
Developer --<Update the code and commit to git repo> Git repo <CI Pipeline> Docker registry <ArgoCD> -->Manifest

### Make sure you have K8s cluster running

### ArgoCD Arch review

* API Server
	* Exposes Grpc calls so clients can connects
	* Application management and Status reporting
	* App operations , like, sync, rollback and user-define actions. 
	* Credentials management. 
	* Authentication and Auth delegations to external identify providers
	* RBAC enforcement
* Repo Server
	* Connect and Clone git repo 
	* Local cache of repo
	* Contains app manifests. 
* App Controller 
	* Manage Desired and Actual state of the app. 
	* Monitoring of the apps state. 
	* Any user-defined hooks. 


*NOTE* --> Can handlt with GUI or CI or API. 


# Working woth ArgoCD

### Installing ArgoCD 
#### Manual Installation:-
##### GUI

* Check namespaces:- 

```
root@0c737c8c793e:/# kubectl get namespaces
NAME              STATUS   AGE
default           Active   5d21h
ingress-nginx     Active   2d20h
kube-node-lease   Active   5d21h
kube-public       Active   5d21h
kube-system       Active   5d21h
root@0c737c8c793e:/# 
```

* create ``argocd`` namespace:- 

```
kubectl create namespace argocd
```

* Install ArgoCD

```
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.7.10/manifests/ha/install.yaml```
```

```
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
```

* Check and Validate

```diff
root@0c737c8c793e:/# kubectl get all -n argocd
NAME                                                    READY   STATUS    RESTARTS   AGE
pod/argocd-application-controller-0                     1/1     Running   0          2d22h
pod/argocd-applicationset-controller-8654ccc65c-622ch   1/1     Running   0          2d22h
pod/argocd-dex-server-78dbb98874-sw9dn                  1/1     Running   0          2d22h
pod/argocd-notifications-controller-6f8566bfb6-wttv2    1/1     Running   0          2d22h
pod/argocd-redis-74d77964b-t4ptf                        1/1     Running   0          2d22h
pod/argocd-repo-server-69869c95f9-pxh9g                 1/1     Running   0          2d22h
pod/argocd-server-85489fbf89-97xzf                      1/1     Running   0          2d22h

NAME                                              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/argocd-applicationset-controller          ClusterIP   10.233.61.209   <none>        7000/TCP,8080/TCP            2d22h
service/argocd-dex-server                         ClusterIP   10.233.30.157   <none>        5556/TCP,5557/TCP,5558/TCP   2d22h
service/argocd-metrics                            ClusterIP   10.233.12.89    <none>        8082/TCP                     2d22h
service/argocd-notifications-controller-metrics   ClusterIP   10.233.56.203   <none>        9001/TCP                     2d22h
service/argocd-redis                              ClusterIP   10.233.12.24    <none>        6379/TCP                     2d22h
service/argocd-repo-server                        ClusterIP   10.233.24.86    <none>        8081/TCP,8084/TCP            2d22h
+service/argocd-server                             NodePort    10.233.45.136   <none>        80:30421/TCP,443:31609/TCP   2d22h
service/argocd-server-metrics                     ClusterIP   10.233.45.199   <none>        8083/TCP                     2d22h

NAME                                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/argocd-applicationset-controller   1/1     1            1           2d22h
deployment.apps/argocd-dex-server                  1/1     1            1           2d22h
deployment.apps/argocd-notifications-controller    1/1     1            1           2d22h
deployment.apps/argocd-redis                       1/1     1            1           2d22h
deployment.apps/argocd-repo-server                 1/1     1            1           2d22h
deployment.apps/argocd-server                      1/1     1            1           2d22h

NAME                                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/argocd-applicationset-controller-8654ccc65c   1         1         1       2d22h
replicaset.apps/argocd-dex-server-78dbb98874                  1         1         1       2d22h
replicaset.apps/argocd-notifications-controller-6f8566bfb6    1         1         1       2d22h
replicaset.apps/argocd-redis-74d77964b                        1         1         1       2d22h
replicaset.apps/argocd-repo-server-69869c95f9                 1         1         1       2d22h
replicaset.apps/argocd-server-85489fbf89                      1         1         1       2d22h

NAME                                             READY   AGE
statefulset.apps/argocd-application-controller   1/1     2d22h
root@0c737c8c793e:/# 
```

**NOTE** In order to expose the service we modify ``service/argocd-server `` to NodePort. 


##### CLI

```
root@0c737c8c793e:/# curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
```

```
root@0c737c8c793e:/# install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
```

```
root@0c737c8c793e:/# argocd version
```
```
argocd: v2.7.10+469f257
  BuildDate: 2023-07-31T22:22:19Z
  GitCommit: 469f25753b2be7ef0905a11632a6382060bcae99
  GitTreeState: clean
  GoVersion: go1.19.11
  Compiler: gc
  Platform: linux/amd64
root@0c737c8c793e:/# 
```

```
root@0c737c8c793e:/# argocd version --client
argocd: v2.7.10+469f257
  BuildDate: 2023-07-31T22:22:19Z
  GitCommit: 469f25753b2be7ef0905a11632a6382060bcae99
  GitTreeState: clean
  GoVersion: go1.19.11
  Compiler: gc
  Platform: linux/amd64
root@0c737c8c793e:/# 
```

#### Access the GUI ### Default Username: Admin

* Get the initial password using: 

```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo 
```

### Setting up Git repo:- 

* Create a public repo [later we will check with private repo]
* Clone on your local machine and start adding the manifest. 
	* mkdir nginx-deployments-yaml &&  cd nginx-deployments-yaml 
		* nginx-deployment.yaml
		* nginx-service.yaml
	* git add . 
	* git commit -m "your comment"
	* git push
* Here your git repo is ready for CD with ArgoCD. 

* Copy the link **Clone the repo**

### On ArgoCD 

*  Go to settings
	* repository 
		* connect repo using HTTP's
			* Type: git
			* Repository URL : Paste the link here [github repo]
			* Username 
			* Password
			**NOTE** - leave this Username and Password as this is the public URL we no need of it. 
			* Connect 
			**NOTE** You should see ``Successful`` message. 

* Go to Application
	* New App 
		* Application Name: simple-nginx-appl
		* Project: default
		* SYNC Policy : Leave all to default
		* Source : 
			* Repo URL: Link 
			* Revision: Head
			* Path: Folder Name if there are any, where the yamls are residing. 
		* Destination: 
			* Cluster URL: Cluster URL 
			* Namespace: default 
		* Click on Create. 

		* You will see ``out of sync`` 
		* ArgoCD connect to Git, you can see in the image, however the State is out of sync. 

* App Diff							
	* You can check the deffrence before syncing the app. 
	* Current state on kubernetes 
* SYNC
	* This will SYNC argocd with git and deploy the app. 
	* Match the yaml from git and app will be ready. 

### On K8s cluster:- 

* We can check the same from k8s 

```
kubectl get pod
```

### Changing ArgoCD admin password

**NOTE** Here is my case I dont have any Ingress Controller to expose the services, So I am directly running commands on master node. 

```
root@kmaster1:~# argocd login 10.233.45.136:80
```

**NOTE** Accept the Certificate ## Proceed insecurely (y/n)? y ## 

```
root@kmaster1:~# argocd login 10.233.45.136:80
WARNING: server certificate had error: tls: failed to verify certificate: x509: cannot validate certificate for 10.233.45.136 because it doesn't contain any IP SANs. Proceed insecurely (y/n)? y
Username: admin
Password: 
'admin:login' logged in successfully
Context '10.233.45.136:80' updated
root@kmaster1:~# 
```

* Change or update the passord: 

```
root@kmaster1:~# argocd account update-password
*** Enter password of currently logged in user (admin): 
*** Enter new password for user admin: 
*** Confirm new password for user admin: 
Password updated
Context '10.233.45.136:80' updated
root@kmaster1:~# 
```

#### Syncing updated infradtructure manifest:- 

* CD into the directory where the application manifest is cloned in local system. 
	* cd nginx-deployments-yaml 
		* nginx-deployment.yaml
		* nginx-service.yaml
	* Update the ``nginx-deployment.yaml`` configuration ### change replicaset
	* git add . 
	* git commit -m "message"
	* git push 
* Now the git repo is updated, ### validate that by login into github

# On ArgoCD GUI:- 

* You will see the message ``OutOfSync``
* APP DIFF - Cross check the difference ``compact-diff`` 
* SYNC manually clicking it. 
* ArgoCD will SYNC the changes and applies to k8s cluster. 
* You will have ``Synced``	messages
* Check ``kubectl get replicaset``

#### Configure deployment using ``kubectl``


```
kubectl scale deploy nginx-deployment --replicas 3
```

* Per ArgoCD we will have 2 replicas, however creating from kubectl created a one more replicas, 
* ArgoCD is now ``OutOfSync`` which is true, live deployment is not matching 
* If we now do a manual SYNC, extra replica will be terminated, and the application state will go to desirsed state. 

#### Autmated SYNC, Automated Prune, and Self-Healing 

* Login to ArgoCD GUI:
	* Application
		* App Details
			* SYNC Policy
				* ENABLE AUTO-SYNC ### OK or Cancel 
			* AUTOMATED - <DISABLE AUTO SYNC>
			* PRUNE RESOURCES - ENABLE ## auto delete resource
			* SELF HEAL - ENABLE ## Manual healing if the app is deployed by kubectl which is managed by ArgoCD. ### Always matched what is in github. 

#### ArgoCD Using CLI:- 

* List project:

```
argocd proj list
```

* List repo

```
argocd repo list
```

* view app list 

```
argocd app list
```

* Create a new project 

```
argocd proj create <name of the project> -d https://<clusterip:port>,default -s https://<github repo>
```

* Verify it from GUI, if that is visible or not. 

#### Conect with Private registry: 

* Create a private repo on github 
* git clone 
* Add the manifest files
* git add . 
* git commit -m "comment"
* git push 

### GUI

* Setting 
	* repository 
		* connect repo using https
			* git 
				repo: github repo
				username: github username
				password: github password 
	Connect
	
### CLI:-

```
argocd proj add-source <project name> http://<github repo>
```

### GUI:- 

* Setting 
	* Project
		*  Proj name
			* You should see the newly added repo

* Application
	* New APP
		* App Name: any name
		* Project: select the proj
		* repo name: private repo
		* revision: head
		* path: if any sub directory 
		* destination: select the cluster
		* namespace: default
		* SYNC Policy
	* Create
	* SYNC

### K8s CLI

```
kubectl get all
```











