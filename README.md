#  Multi-tier application to K8s cluster with RBAC + Dashboard for cluster + NFS server for Persistent Volumes

K8s cluster setup - manual 

On Master Node:
  # sudo wget https://raw.githubusercontent.com/lerndevops/labs/master/kubernetes/0-install/daemon.json -P /etc/docker
  # sudo systemctl restart docker.service
Initialize kubernetes Master Node
  # sudo kubeadm init
wait for few seconds for the task to complete.
Execute these commands:
  # mkdir -p $HOME/.kube
  # sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  # sudo chown $(id -u):$(id -g) $HOME/.kube/config
  Wait for few seconds now and execute below command:
  # kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.1/manifests/calico.yaml
  # kubectl get nodes
  output will be like this:
    NAME               STATUS   ROLES                  AGE     VERSION
    ip-172-31-57-157   Ready    control-plane,master   2m40s   v1.23.4
  Execute below command on Master node get a token
  # sudo kubeadm token create --print-join-command
  Now copy the generated token in a note pad
  Example token will look like these:
  kubeadm join 172.31.57.157:6443 --token u1bax4.diyl7ae60dksps65 --discovery-token-ca-cert-hash sha256:4eb21a2496352ce324f2e9fe0bc7b07c73033146705c6dfb09862e8b8b5c247d 

===========================================================================================================
Execute below steps on Worker Node
===========================================================================================================
  # sudo su -
  # hostname WORKER1
  # sudo su -
  # sudo wget https://raw.githubusercontent.com/lerndevops/labs/master/kubernetes/0-install/daemon.json -P /etc/docker
  # sudo systemctl restart docker.service
Copy the token from moster node and paste on Worker node
  example
  # kubeadm join 172.31.57.157:6443 --token u1bax4.diyl7ae60dksps65 --discovery-token-ca-cert-hash sha256:4eb21a2496352ce324f2e9fe0bc7b07c73033146705c6dfb09862e8b8b5c247d 
you will copy your generated token
Now worker will join the master

==================================================================================

On master execute below command:

# kubectl get nodes

both master worker nodes will be there.

===========================================================================


Project execution:

 Step 1: Create a namespace cep-project1
      kubectl create namespace cep-project1 -l slearn=project1
Step 2: Create a service Account with name as Sandry and namespace = cep-project1
      kubectl create serviceaccount sandry --namespace cep-project1
Step 3: Create a cluster role binding to the service account Sandry and give access as cluster admin role 
      kubectl create clusterrolebinding sandry-access --serviceaccount=cep-project1:sandry --clusterrole=cluster-admin
Step 4: Deploy the dashboard
      kubectl create -f https://raw.githubusercontent.com/sonal0409ORG/educka/master/dashboard/dashboard-insecure-v2.4.0.yml     
Step5: On Master node set up NFS server:

====================================
Create an NFS server
  sudo mkdir -p /data
  cd /data
Install the NFS kernel server on the machine
  sudo apt install nfs-kernel-server
Change the owner user and group to nobody and nogroup
  sudo chown nobody:nogroup /data/
Set permissions to 777 to allow everyone to read, write, and execute files in this directory
  sudo chmod 777 /data/
Open the exports file in the /etc directory for permission to access the host server machine
  sudo vi /etc/exports
Add the following code to the file:
  /data *(rw,sync,no_root_squash)
Note: Exit the file and save the changes
Use the exportfs command to export all shared folders you registered in the /etc/exports file after making the appropriate changes
  sudo exportfs -rv
Restart the NFS kernel server to apply the configuration changes
  sudo systemctl restart nfs-kernel-server
Deploy NFS client on worker nodes. Go to worker Node and execute below command
  sudo apt install nfs-common
