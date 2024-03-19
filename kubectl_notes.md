kubectl edit deployment/a4p-spa-deploy -n epistola --> change the image name
kubectl rollout status deployment/a4p-spa-deploy -n epistola
kubectl rollout history deployment/a4p-spa-deploy -n epistola

copy secret from one namespace to testing namespace:
kubectl get secret a4p-spa-secret -n epistola -o yaml | sed 's/namespace: .*/namespace: testing/' | kubectl apply -f -

Renew Certificates:
- kubeadm certs renew all
- restart control plane pods:  to restart a static Pod you can temporarily remove its manifest file from /etc/kubernetes/manifests/ and wait for 20 seconds (see the fileCheckFrequency value in KubeletConfiguration struct. The kubelet will terminate the Pod if it's no longer in the manifest directory. You can then move the file back and after another fileCheckFrequency period, the kubelet will recreate the Pod and the certificate renewal for the component can complete.
- cp .kube/config .kube/config_old
- sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
- sudo chown $(id -u):$(id -g) $HOME/.kube/config

=============================================
container --> must be quick to start and quick to scale

[1]
Virtual machines can give the appearance of separation while sharing physical
hardware, but a virtual machine has the overhead of running its own operating system, services, and virtual devices, making it slower to start and less scalable.

mkdir /tmp/newroot
cp --parents /bin/bash /bin/ls
cp --parents /lib64/ld-linux-x86-64.so.2 $(ldd /bin/bash /bin/ls | grep '=>' | awk '{print $3}') /tmp/newroot
chroot /tmp/newroot /bin/bash


At the same time, processes in a container are still sharing the same CPU, memory, and network. A process that uses too many of those resources will prevent other processes from running properly. Namespaces can’t solve that problem, however. To prevent this issue, you need resource limiting (cgroups).


[2]
Process prioritization will not help a container orchestration environment like Kubernetes know what host to use when scheduling a new container, because even low-priority processes can get a lot of CPU time when the CPU is idle.

root@host01:/sys/fs/cgroup/cpu# cd system.slice/runc-${CUL_ID}.scope
cpu.shares --> Slice of the CPU relative to this cgroup’s peers
cpu.cfs_period_us --> Length of a period, in microseconds
cpu.cfs_quota_us --> CPU time during a period, in microseconds


[3]
- Container image: more than just the set of layers that make up the overlay
filesystem. It also includes important metadata, such as the initial command
for the container and any environment variables for that command.
- Containers: transform the way we package and deploy application components.
- Benefits of modern application architecture: scalability, reliability, and resiliency.
- To build a scalable and reliable application, we have always needed to take advantage of multiple servers to handle the application’s load and preclude a single point of failure.


[4]
A taint restricts what can be scheduled on a node.
# kubectl get node -o json | jq '.items[]|.metadata.name,.spec.taints[]'



===== Upgrade =====
apt update
apt-cache madison kubeadm

sudo apt-mark unhold kubeadm && sudo apt-get update && sudo apt-get install -y kubeadm=1.23.10-00 && sudo apt-mark hold kubeadm

kubeadm version
kubeadm upgrade plan
sudo kubeadm upgrade apply v1.23.10

sudo kubeadm upgrade node

kubectl drain kubem --ignore-daemonsets --delete-emptydir-data
kubectl drain chaos --ignore-daemonsets --delete-emptydir-data

sudo apt-mark unhold kubelet kubectl && sudo apt-get update && sudo apt-get install -y kubelet=1.23.10-00 kubectl=1.23.10-00 && sudo apt-mark hold kubelet kubectl

sudo systemctl daemon-reload
sudo systemctl restart kubelet

kubectl uncordon kubem
kubectl uncordon chaos
===== End of Upgrade =====


journalctl -p err -b
sudo journalctl -u kubelet



[5]
$ kubectl config set-context --current --namespace shield

Kubernetes clusters run an internal DNS service that is the centre of service discovery. Service names are automatically registered with the cluster DNS, and every Pod and container is pre-configured to use the cluster DNS. This means every Pod/container can resolve every Service name to a ClusterIP and connect to the Pods behind it.

Ingress fixes this by exposing multiple Services through a single cloud load-balancer. It creates a LoadBalancer Service, on port 80 or 443, and uses host-based and path-based routing to send traffic to the correct backend Service.

A cloud-native application is one that’s designed to meet cloud-like demands of auto-scaling, self-healing, rolling updates, rollbacks and more.

1. You post a new Service manifest to the API server
2. The request is authenticated, authorized, and subjected to admission policies
3. The Service is allocated a stable virtual IP address called a ClusterIP
4. An Endpoints object (or EndpointSlice) is created to hold a list of healthy Pods matching the Service’s
label selector
5. The Pod network is configured to handle traffic sent to the ClusterIP (more on this later)
6. The Service’s name and IP are registered with the cluster DNS

kube-proxy: is a Pod-based Kubernetes-native app that implements a controller watching the API server for new Services and Endpoints objects. When it sees them, it creates local IPVS rules telling the node to intercept traffic destined for the Service’s ClusterIP and forward it to individual Pod IPs.

$ kubectl run -it dnsutils --image gcr.io/kubernetes-e2e-test-images/dnsutils:1.3

A Kubernetes cluster is running on AWS and the AWS administrator has created a 25GB EBS volume called “ebs-vol”. The Kubernetes administrator creates a PV called “k8s-vol” that links back to the “ebs-vol” via the
ebs.csi.aws.com CSI plugin. While that might sound complicated, it’s not. The PV is simply a way of representing the external storage asset on the Kubernetes cluster. Finally, the Pod uses a PVC to claim access to the PV and
start using it.



[6]
We’re defining a stateful application as one that creates and saves valuable data.

Failed Pods managed by a StatefulSet will be replaced by new Pods with the exact same Pod name, the exact same DNS hostname, and the exact same volumes.


[7] RBAC
Which users can perform which actions against which resources.

A powerful pattern is to define Roles at the cluster level (ClusterRoles) and bind them to specific Namespaces via RoleBindings. This lets you define common roles once, and re-use them across multiple Namespaces. For
example, the following YAML defines the same “read-deployments” role, but this time at the cluster level. This can be re-used in selected Namespaces via RoleBindings.

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
 name: read-deployments
rules:
- apiGroups: ["apps"]
 resources: ["deployments"]
 verbs: ["get", "watch", "list"]



[8]
The API is RESTful. This is jargon for a modern web API that accepts CRUD-style requests via standard HTTP methods.

REST requests comprise a verb and a path to a resource. Verbs relate to actions, and are the standard HTTP methods you saw in the previous table. Paths are a URI path to the resource in the API.

$ kubectl proxy --port 9000 &
$ curl http://localhost:9000/api/v1/namespaces/shield/pods

$ curl -X POST -H "Content-Type: application/json" --data-binary @ns.json http://localhost:9000/api/v1/namespaces




[9]
In Kubernetes, Ingress is a set of routing rules that define how external traffic is routed to an application inside a Kubernetes cluster. An Ingress controller watches for changes to objects in the cluster and then wires together a data path for each request to be resolved.

Validate even further by doing a dry run install:
kubectl apply -f contour.yaml --dry-run=client

kubectl create namespace my-ingress-app -o yaml > my-ingress-app-namespace.yaml

cat << EOF > rate-limit.yaml
apiVersion: projectcontour.io/v1
kind: HTTPProxy
metadata:
 name: rate
 namespace: http-proxy
spec:
 virtualhost:
   fqdn: rate.$INGRESS_HOST
   rateLimitPolicy:
     local:
       requests: 2
       unit: minute
 routes:
   - conditions:
     - prefix: /
     services:
       - name: nginx
         port: 80
EOF



[10] Δημιουργία certificate για αναφορά στον HTTPProxy του Envoy:
openssl req -new -newkey rsa:2048 -nodes -keyout tls.key -out tls.csr
Αποθηκεύω το tls.key και το pem certificate tls.pem
kubectl create secret tls a4p-mgmt-tls -n epistola --key tls.key --cert tls.pem


[11] Copy files from the local system (agro_webgl) to a pod (wordpress-6fdd58fc4d-6ln7p) to path (/var/www/html)
kubectl cp -n epistola .\agro_webgl\ wordpress-6fdd58fc4d-6ln7p:/var/www/html


==========


docker tag e27a4cc50d0a epistola/flask-api  ==> tag existing image as epistola/fask-api
docker push epistola/flask-api
kubectl apply -f spa-flaskapi-deploy.yaml
kubectl delete -f spa-flaskapi-deploy.yaml
kubectl get all -n epistola
kubectl describe pods <pod-name> -n epistola
kubectl port-forward -n epistola spa-nginx-d-5b5d476885-kq79k 8087:80
kubectl exec -n epistola <pod-name> -it – /bin/bash

You can run the following commands inside the container:
ls /
cat /proc/mounts
cat /proc/1/maps
apt-get update
apt-get install -y tcpdump
tcpdump
apt-get install -y lsof
lsof
apt-get install -y procps
ps aux
ps aux | grep nginx

kubectl run curl --image=radial/busyboxplus:curl -i --tty
nslookup php-service.epistola

service nginx reload
195.251.3.1
====================================================================
Build docker image:
docker login
docker build -t epistola/php .   // must have a Dockerfile in the directory
docker push epistola/php
Create Commands:
Create a persistent volume claim : kubectl apply -f pv-www-claim.yml --namespace=epistola
Create a config map for the configuration file of nginx : kubectl apply -f configmap.yaml --namespace=epistola
Create a deployment for the nginx : kubectl apply -f nginx-deploy.yaml --namespace=epistola
Create a deployment for the php : kubectl apply -f php-deploy.yaml --namespace=epistola
Create a deployment for the mysql : kubectl apply -f mysql-deploy.yaml --namespace=epistola      // contains persistent volume claim, service and deployment
Secrets:
echo -n 'qw#rty!(*#' > ./root_pass.txt
echo -n 'user' > ./user_name.txt      
echo -n 'user' > ./user_pass.txt
kubectl create secret generic mysql-pass --from-file=root-pass=./root_pass.txt --from-file=user-name=./user_name.txt --from-file=user-pass=./user_pass.txt --from-file=database=./database.txt --namespace epistola
kubectl get secrets
kubectl describe secrets/mysql-pass
kubectl get secret mysql-pass -o jsonpath='{.data}' --namespace=epistola  => echo <pass> | base64 --decode
View Commands:
kubectl get deploy --namespace=epistola
kubectl describe deploy --namespace=epistola
kubectl get pvc --namespace=epistola
kubectl get pods --namespace=epistola
kubectl describe pods --namespace=epistola
kubectl get svc --namespace=epistola

Delete Commands:
kubectl delete secret mysql-pass --namespace=epistola
kubectl delete svc mysql-service --namespace=epistola
kubectl delete pvc mysql-pv-claim --namespace=epistola
kubectl delete deploy mysql-d --namespace=epistola


Your mysql service can be accessed through the following DNS name from within your cluster:
mysql-service.epistola.svc.cluster.local (port 80)
Get the Service IP  :
export SERVICE_IP=$(kubectl get svc --namespace a4p a4p-invs-wordpress --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
http://$SERVICE_IP:port/
Get the environment variables for the mysql pod :
kubectl exec mysql-d-6dfd95db4-xv7pp --namespace=epistola -- env
How to get the node the pod is running on :
kubectl get pods --namespace epistola -o wide
Connect to mysql service :
host : kubem.ceti.gr
port : 32119
type : NodePort
View logs :
kubectl logs nginx-d-86474464b9-nzkhd --namespace epistola
How to List all Resources in a Kubernetes Namespace :
kubectl get all -n epistola
How to run a shell in a pod:
kubectl exec php-d-7679c87677-hh2qw -n epistola -it /bin/bash



configmap.yaml for nginx:

kind: ConfigMap
apiVersion: v1
metadata:
name: nginx-config
namespace: epistola
data:
nginx.conf: |
  server {
    listen 80;
    index index.php index.html;
    server_name localhost;
    error_log /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
    root /code;
    
    location ~ \.php$ {
      try_files $uri = 404;
      fastcgi_split_path_info ^(.+\.php)(/.+)$;
      fastcgi_pass php-service:9000;
      fastcgi_index index.php;
      include fastcgi_params;
      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
      fastcgi_param PATH_INFO $fastcgi_path_info;
    }
  }






https://kubernetes.io/docs/concepts/services-networking/ingress/
Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.
You must have an Ingress controller to satisfy an Ingress. Only creating an Ingress resource has no effect. You may need to deploy an Ingress controller such as ingress-nginx. You can choose from a number of Ingress controllers.

Inspect the environment of your running nginx Pods : kubectl exec nginx-d-86474464b9-l7dqt -- printenv | findstr SERVICE

Kubernetes offers a DNS cluster addon Service that automatically assigns dns names to other Services => kubectl get services kube-dns --namespace=kube-system

kubectl run curl --image=radial/busyboxplus:curl -i --tty
nslookup php-service.epistola

A Service is a set of Pods, which can be reached by a single, fixed DNS name or IP address.
ClusterIP provides a single IP address for the set of Pods the Service is pointing to. This IP address is accessible only within the cluster.

kubectl get service hostnames -o json
iptables-save | grep hostnames

For each port of each Service, there should be 1 rule in KUBE-SERVICES and one KUBE-SVC-<hash> chain. For each Pod endpoint, there should be a small number of rules in that KUBE-SVC-<hash> and one KUBE-SEP-<hash> chain with a small number of rules in it. The exact rules will vary based on your exact config (including node-ports and load-balancers).

Is kube-proxy running? ⇒ ps auxw | grep kube-proxy

Finally, at the php-deploy.yaml file, the following lines must be commented out:
# name: http
# protocol: TCP
because the php-fpm doesn't use the http protocol, it uses fastcgi. The only way to connect with it was by setting up Nginx to use fastcgi.

GRANT ALL ON farm_management.* TO 'user'@'%' identified by 'user';  ----> grant access to user from any machine
'user' did not have access from php pod to the mysql pod ----> the host name is the hostname that the server believes you are coming from, not the hostname you are really coming from.



==========


The files referenced on the commands can be found at https://github.com/vpistola/kubectl-manifests.git
[docker hub: epistola - !$!Ovangel!s198#]

Build docker image:
docker login
docker build -t epistola/php .   // must have a Dockerfile in the directory
docker push epistola/php

Create Commands:
Create a persistent volume claim : kubectl apply -f pv-www-claim.yml --namespace=epistola
Create a config map for the configuration file of nginx : kubectl apply -f configmap.yaml --namespace=epistola
Create a deployment for the nginx : kubectl apply -f nginx-deploy.yaml --namespace=epistola
Create a deployment for the php : kubectl apply -f php-deploy.yaml --namespace=epistola
Create a deployment for the mysql : kubectl apply -f mysql-deploy.yaml --namespace=epistola      // contains persistent volume claim, service and deployment

Secrets:
echo -n 'qw#rty!(*#' > ./root_pass.txt
echo -n 'user' > ./user_name.txt      
echo -n 'user' > ./user_pass.txt
kubectl create secret generic mysql-pass --from-file=root-pass=./root_pass.txt --from-file=user-name=./user_name.txt --from-file=user-pass=./user_pass.txt --from-file=database=./database.txt --namespace epistola
kubectl get secrets
kubectl describe secrets/mysql-pass
kubectl get secret mysql-pass -o jsonpath='{.data}' --namespace=epistola  => echo <pass> | base64 --decode

View Commands:
kubectl get deploy --namespace=epistola
kubectl describe deploy --namespace=epistola
kubectl get pvc --namespace=epistola
kubectl get pods --namespace=epistola
kubectl describe pods --namespace=epistola
kubectl get svc --namespace=epistola

Delete Commands:
kubectl delete secret mysql-pass --namespace=epistola
kubectl delete svc mysql-service --namespace=epistola
kubectl delete pvc mysql-pv-claim --namespace=epistola
kubectl delete deploy mysql-d --namespace=epistola

Your mysql service can be accessed through the following DNS name from within your cluster:
mysql-service.epistola.svc.cluster.local (port 80)

Get the Service IP  :
export SERVICE_IP=$(kubectl get svc --namespace a4p a4p-invs-wordpress --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
http://$SERVICE_IP:port/

Get the environment variables for the mysql pod :
kubectl exec mysql-d-6dfd95db4-xv7pp --namespace=epistola -- env

How to get the node the pod is running on :
kubectl get pods --namespace epistola -o wide

Connect to mysql service :
host : kubem.ceti.gr
port : 32119
type : NodePort

View logs :
kubectl logs nginx-d-86474464b9-nzkhd --namespace epistola
How to List all Resources in a Kubernetes Namespace :
kubectl get all -n epistola

How to run a shell in a pod:
kubectl exec php-d-7679c87677-hh2qw -n epistola -it /bin/bash

configmap.yaml for nginx:
kind: ConfigMap
apiVersion: v1
metadata:
name: nginx-config
namespace: epistola
data:
nginx.conf: |
  server {
  listen 80;
  index index.php index.html;
  server_name localhost;
  error_log /var/log/nginx/error.log;
  access_log /var/log/nginx/access.log;
  root /code;
    location ~ \.php$ {
      try_files $uri = 404;
      fastcgi_split_path_info ^(.+\.php)(/.+)$;
      fastcgi_pass php-service:9000;
      fastcgi_index index.php;
      include fastcgi_params;
      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
      fastcgi_param PATH_INFO $fastcgi_path_info;
    }
  }

https://kubernetes.io/docs/concepts/services-networking/ingress/
Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.
You must have an Ingress controller to satisfy an Ingress. Only creating an Ingress resource has no effect. You may need to deploy an Ingress controller such as ingress-nginx. You can choose from a number of Ingress controllers.
Inspect the environment of your running nginx Pods : kubectl exec nginx-d-86474464b9-l7dqt -- printenv | findstr SERVICE
Kubernetes offers a DNS cluster addon Service that automatically assigns dns names to other Services => kubectl get services kube-dns --namespace=kube-system
kubectl run curl --image=radial/busyboxplus:curl -i --tty
nslookup php-service.epistola
A Service is a set of Pods, which can be reached by a single, fixed DNS name or IP address.

ClusterIP provides a single IP address for the set of Pods the Service is pointing to. This IP address is accessible only within the cluster. 
(https://kubernetes.io/docs/tutorials/stateless-application/guestbook/)

kubectl get service hostnames -o json
iptables-save | grep hostnames

For each port of each Service, there should be 1 rule in KUBE-SERVICES and one KUBE-SVC-<hash> chain. For each Pod endpoint, there should be a small number of rules in that KUBE-SVC-<hash> and one KUBE-SEP-<hash> chain with a small number of rules in it. The exact rules will vary based on your exact config (including node-ports and load-balancers).

Is kube-proxy running? :
ps auxw | grep kube-proxy
root@nginx-d-86474464b9-zxqvq:/# curl php-service.epistola:9000
curl: (56) Recv failure: Connection reset by peer

$host = 'mysql-service.epistola:9002';    // functions.php : the $host is the name of the mysql service dns name and the port that is listening on
Nginx conf file must include the following line
fastcgi_pass php-service.epistola:9000;
Finally, at the php-deploy.yaml file, the following lines must be commented out:
# name: http
# protocol: TCP
because the php-fpm doesn't use the http protocol, it uses fastcgi. The only way to connect with it was by setting up Nginx to use fastcgi.

GRANT ALL ON farm_management.* TO 'user'@'%' identified by 'user';  ----> grant access to user from any machine
'user' did not have access from php pod to the mysql pod ----> the host name is the hostname that the server believes you are coming from, not the hostname you are really coming from.



==========


[1] COPY mgmt-registry-credentials SECRET FROM EPISTOLA NS TO TESTING
kubectl get secret mgmt-registry-credentials --namespace=epistola -o yaml | sed 's/namespace: .*/namespace: testing/' | kubectl apply -f -


[2] COPY a4p-mgmt-secret SECRET FROM EPISTOLA NS TO TESTING
kubectl get secret a4p-mgmt-secret --namespace=epistola -o yaml | sed 's/namespace: .*/namespace: testing/' | kubectl apply -f -


[3] HOW TO READ KUBERNETES SECRET
kubectl get secret mgmt-registry-credentials -n testing -o yaml
echo "<the data section from the output of the previous command>" | base64 --decode
kubectl get secret mgmt-registry-credentials -n testing -o jsonpath="{.data.dockerconfigjson}" | base64 --decode


[4] CREATE NEW TLS SECRET FROM CERTIFICATE FILE
kubectl create secret tls test-tls --key="tls.key" --cert="tls.crt"


[5] RENEW TLS SECRET WITH THE NEW CRT FILE
Get the new crt file and copy the contents in the tls.crt section of the secret file (see below). The tls.key (the private key) is the same


// tls secret file
apiVersion: v1
data:
tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURKakNDQWc2Z0F3SUJBZ0lKQUw2Y3R2bk9zMzlUTUEwR0NTcUdTSWIzRFFFQkJRVUFNQll4RkRBU0JnTlYKQkFNVEMyWnZieTVpWVhJdVkyOXRNQjRYRFRFNE1USXhOREUxTWpJeU1Gb1hEVEU1TVRJeE5ERTFNakl5TUZvdwpGakVVTUJJR0ExVUVBeE1MWm05dkxtSmhjaTVqYjIwd2dnRWlNQTBHQ1NxR1NJYjNEUUVCQVFVQUE0SUJEd0F3CmdnRUtBb0lCQVFDbWVsQTNqVy9NZ2REejJNazMwbXZ4K2VOSHJkQlIwMEJ4ZUR1VjBjYWVFUGNFa2RmSnk5V28KaTFpSXV1V04vZGV6UEhyTWMxenBPNGtzbWU5NThRZVFCWjNmVThWeGpRYktmb1JzNnhQUlNKZVVSckVCcWE4SQpUSXpEVVdaUTAwQ2xsa1dOejE4dDYvVjJycWxJd1VvaTVZWHloOVJsaWR4MjZRaXJBcFFFaXZDY2QzdUExc3AwCkUxRXdIVGxVdzFqSE9Eb3BLZGxaRndmcWhFSHNmYjZvLzJFb1A1MXMwY2JuTld6MHNsUjhhejdzOExVYnhBWnkKQkNQdDY1Z2VhT3hYWWUxaWhLYzN4SE4wYSsxMXpBYUdDMnpTemdOcEVWeFFJQ3lZdVZld3dNb0FrcHNkdGEybwpnMnFTaDZQZzRHeFFabzRwejIwN0c2SkFUaFIyNENiTEFnTUJBQUdqZHpCMU1CMEdBMVVkRGdRV0JCU3NBcUZoCkpPS0xZaXNHTkNVRGU4N1VWRkp0UERCR0JnTlZIU01FUHpBOWdCU3NBcUZoSk9LTFlpc0dOQ1VEZTg3VVZGSnQKUEtFYXBCZ3dGakVVTUJJR0ExVUVBeE1MWm05dkxtSmhjaTVqYjIyQ0NRQytuTGI1enJOL1V6QU1CZ05WSFJNRQpCVEFEQVFIL01BMEdDU3FHU0liM0RRRUJCUVVBQTRJQkFRQU1wcDRLSEtPM2k1NzR3dzZ3eU1pTExHanpKYXI4Cm8xbHBBa3BJR3FMOHVnQWg5d2ZNQWhsYnhJcWZJRHlqNWQ3QlZIQlc1UHZweHpKV3pWbmhPOXMrdzdWRTlNVHUKWlJHSXVRMjdEeExueS9DVjVQdmJUSTBrcjcwYU9FcGlvTWYyUVUvaTBiN1B2ajJoeEJEMVZTVkd0bHFTSVpqUAo0VXZQYk1yTWZUWmJka1pIbG1SUjJmbW4zK3NTVndrZTRhWXlENVVHNnpBVitjd3BBbkZWS25VR0d3TkpVMjA4CmQrd3J2UUZ5bi9kcVBKTEdlNTkvODY4WjFCcFIxRmJYMitUVW4yWTExZ0dkL0J4VmlzeGJ0b29GQkhlVDFLbnIKTTZCVUhEeFNvWVF0VnJWSDRJMWh5UGRkdmhPczgwQkQ2K01Dd203OXE2UExaclVKOURGbFl2VTAKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
tls.key: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb3dJQkFBS0NBUUVBcG5wUU40MXZ6SUhRODlqSk45SnI4Zm5qUjYzUVVkTkFjWGc3bGRIR25oRDNCSkhYCnljdlZxSXRZaUxybGpmM1hzeng2ekhOYzZUdUpMSm52ZWZFSGtBV2QzMVBGY1kwR3luNkViT3NUMFVpWGxFYXgKQWFtdkNFeU13MUZtVU5OQXBaWkZqYzlmTGV2MWRxNnBTTUZLSXVXRjhvZlVaWW5jZHVrSXF3S1VCSXJ3bkhkNwpnTmJLZEJOUk1CMDVWTU5ZeHpnNktTblpXUmNINm9SQjdIMitxUDloS0QrZGJOSEc1elZzOUxKVWZHcys3UEMxCkc4UUdjZ1FqN2V1WUhtanNWMkh0WW9Tbk44UnpkR3Z0ZGN3R2hndHMwczREYVJGY1VDQXNtTGxYc01ES0FKS2IKSGJXdHFJTnFrb2VqNE9Cc1VHYU9LYzl0T3h1aVFFNFVkdUFteXdJREFRQUJBb0lCQUMvSitzOEhwZWxCOXJhWgpLNkgvb0ljVTRiNkkwYjA3ZEV0ZVpWUnJwS1ZwWDArTGdqTm1kUTN0K2xzOXMzbmdQWlF4TDFzVFhyK0JISzZWCi9kMjJhQ0pheW1mNmh6cENib21nYWVsT1RpRU13cDZJOEhUMnZjMFhGRzFaSjVMYUlidW0rSTV0MGZlL3ZYWDEKUzVrY0Mya2JGQ2w3L21lcmZJTVNBQy8vREhpRTUyV1QydEIrQk01U2FMV3p4cDhFa3NwNkxWN3ZwYmR4dGtrTwpkZ1A4QjkwWlByck5SdUN5ekRwRUkvMnhBY24yVzNidlBqRGpoTjBXdlhTbTErVk9DcXNqOEkrRkxoUzZJemVuCm1MUkFZNnpWVGpZV05TU2J3dTRkbnNmNElIOEdiQkZJajcrdlN5YVNVTEZiVGJzY3ZzQ3I1MUszbWt2bEVMVjgKaWsvMlJoa0NnWUVBMFpmV2xUTjR2alh2T0FjU1RUU3MwMFhIRWh6QXFjOFpUTEw2S1d4YkxQVFJNaXBEYklEbQp6b3BiMGNTemxlTCtNMVJCY3dqMk5HcUNodXcyczBaNTQyQVhSZXdteG1EcWJaWkFQY0UzbERQNW5wNGRpTFRCClZaMFY4UExSYjMrd2tUdE83VThJZlY1alNNdmRDTWtnekI4dU1yQ1VMYnhxMXlVUGtLdGpJdThDZ1lFQXkxYWMKWjEyZC9HWWFpQjJDcWpuN0NXZE5YdGhFS2dOYUFob21nNlFMZmlKakVLajk3SExKalFabFZ0b3kra1RrdTJjZAp0Wm1zUi9IU042YmZLbEpxckpUWWkzY2E1TGY4a3NxR0Z5Y0x1MXo3cmN6K1lUaEVWSFIyOVkrVHVWYXRDTnkzCklCOGNUQW1ORWlVMlVHR2VKeUllME44Z1VZRXRCYzFaMEg2QWllVUNnWUFETDIrUGJPelUxelQvZ1B3Q09GNjQKQjBOelB3U2VrQXN1WXpueUR6ZURnMlQ2Z2pIc0lEbGh3akNMQzVZL0hPZ0lGNnUyOTlmbURBaFh6SmM0T2tYMwo4cW5uNGlMa3VPeFhKZ1ZyNnRmUlpNalNaRXpHbXhpbEdISVE2MS9MZGdGVTg3WExYWHdmaTZPdW80cUVhNm9YCjhCRmZxOWRVcXB4bEVLY2Y1N3JsK1FLQmdGbjVSaFc2NS9oU0diVlhFWVZQU0pSOW9FK3lkRjcrd3FvaGRoOVQKekQ0UTZ6THBCQXJITkFYeDZZK0gxM3pFVlUzVEwrTTJUM1E2UGFHZ2Rpa2M5TlRPdkE3aU1nVTRvRXMzMENPWQpoR2x3bUhEc1B6YzNsWXlsU0NvYVVPeDJ2UFFwN2VJSndoU25PVVBwTVdKWi80Z2pZZTFjZmNseTFrQTJBR0x3ClJ1STlBb0dCQU14aGFJSUdwTGdmcHk0K24rai9BSWhJUUhLZFRCNVBqaGx0WWhqZittK011UURwK21OeTVMbzEKT0FRc0Q0enZ1b3VxeHlmQlFQZlllYThvcm4vTDE3WlJyc3lSNHlhS1M3cDVQYmJKQlNlcTc5Z0g5ZUNIQkxMbQo0aThCUFh0K0NmWktMQzg3NTNHSHVpOG91V25scUZ0NGxMQUlWaGJZQmtUbURZSWo4Q0NaCi0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==
kind: Secret
metadata:
name: test-tls
namespace: default
type: kubernetes.io/tls

