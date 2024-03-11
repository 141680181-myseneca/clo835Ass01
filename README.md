# clo835Assignment 2

Assignment2: Create K8s cluster, deploy containerized stateless applications using K8s manifests, expose the applications as NodePort services, roll out an updated version of the application

1.	Verify K8s is running by using following commands:
scp -i week5 *.* 35.153.156.152:/tmp
Setup user data for creating K8s in the ec2, which has 30 GB and is of t3.medium type.
 ```bash
 #!/bin/bash
set -ex
sudo yum update -y
sudo yum install docker -y
sudo systemctl start docker
sudo usermod -a -G docker ec2-user
curl -sLo kind https://kind.sigs.k8s.io/dl/v0.11.0/kind-linux-amd64
sudo install -o root -g root -m 0755 kind /usr/local/bin/kind
rm -f ./kind
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
rm -f ./kubectl
kind create cluster --config kind.yaml
yum install mysql

k get nodes
k get all -A

 ```
default       service/kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP                  28d
1.a.   What is the IP of the K8s API server in your cluster?
```bash
kubectl cluster-info
Kubernetes control plane is running at https://127.0.0.1:40277
KubeDNS is running at https://127.0.0.1:40277/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
k get nodes -owide

```
NAME                 STATUS   ROLES    AGE   VERSION    INTERNAL-IP   EXTERNAL-IP   OS-IMAGE       KERNEL-VERSION                  CONTAINER-RUNTIME
kind-control-plane   Ready    master   28d   v1.19.11   172.18.0.2    <none>        Ubuntu 21.04   4.14.336-257.562.amzn2.x86_64   containerd://1.5.2


2.	Deploy MySQL and the first version of web application.
```bash
k create namespace mydb 
k create namespace myapp
export ECR=544378344870.dkr.ecr.us-east-1.amazonaws.com/clo835-week4
aws ecr get-login-password --region us-east-1 | docker login -u AWS ${ECR} --password-stdin
docker pull  544378344870.dkr.ecr.us-east-1.amazonaws.com/clo835-week4:app
docker pull  544378344870.dkr.ecr.us-east-1.amazonaws.com/clo835-week4:mysql
k apply -f mysecret.yaml -n mydb
k apply -f mysecret.yaml -n myapp
k apply -f mysql_pod_manif.yaml -n mydb

```



then we need to check it for future usage by running:
```bash
k describe pod mysqlpod -n mydb
```
I got the mysql’s ip : 10.244.0.19, and then modify webapp_pod_manif.yaml accordingly.
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  6s    default-scheduler  Successfully assigned mydb/mysqlpod to kind-control-plane
  Normal  Pulled     5s    kubelet            Container image "544378344870.dkr.ecr.us-east-1.amazonaws.com/clo835-week4:mysql" already present on machine
  Normal  Created    5s    kubelet            Created container mysql-container
  Normal  Started    5s    kubelet            Started container mysql-container
To check the database working:
```bash
k exec -it mysqlpod -n mydb -c mysql-container -- /bin/bash
mysql -u root -p 
use employees
```
apply app pod yaml:
```bash
k apply -f webapp_pod_manif.yaml -n myapp
```