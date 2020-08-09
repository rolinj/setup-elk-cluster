ELK Cluster App
===============

This repo is for the testing ELK cluster

#### Step 1: Setting Up the Vagrant VM
##### Step 1.0
You may clone this repo to utilize the included Vagrantfile to provision a single VM. Just hit up `vagrant up` to start the provisioning. It will also run some scripts during setup that will..
- Install docker runtime
- Configure DNS to use Google's DNS server.
- Install Kubeadm
- Install ubuntu-desktop to enable the GUI.

##### Step 1.1
After the VM provisioning, we should see now the GUI enabled on VM. The default user is `vagrant` but this user is configured to have root access without specifying password. To be able to login to the GUI, we have to set a password for it.

_SSH on the Vagrant VM_
```
vagrant ssh master-1
```

_Set a password for vagrant user_
```
sudo passwd vagrant
```

Once the password is set, you can now login to the GUI.

#### Step 2: Configuring Kubernetes
##### Step 2.0 -- Kubectl Autocomplete
This will enable the auto complete via tab button of kubectl commands.
```
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
```

##### Step 2.1 -- Initialize the Kubernetes cluster using kubeadm utility
This will bootstrap the controlplane components. 
```
sudo kubeadm init
```

##### Step 2.2 -- Enable vagrant user to run kubectl commands
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

##### Step 2.3 -- Remove taint from master node
This will allow us to schedule pods on the master node. _(since we only provisioned 1 VM)_
```
kubectl taint nodes master-1 node-role.kubernetes.io/master:NoSchedule-
```

#### Step 3: Setting up the ELK Cluster
##### Step 3.0 -- Install custom resource definitions and the operator
```
kubectl apply -f https://download.elastic.co/downloads/eck/1.1.2/all-in-one.yaml
```

##### Step 3.1 -- Create a Persistent Volume
```
kubectl apply -f pv-elk.yaml
```

##### Step 3.2 -- Deploy the elasticsearch cluster
This yaml file is included on this repo as well. You can either clone this repo from within the VM or you can just create a file and copy the contentsm then proceed to apply.
**NOTE:** We have disabled the SSL for this as we do not have certificate.
```
kubectl apply -f elasticsearch-elk.yaml
```

##### Step 3.3 -- Get credentials of elastic user
A default user named elastic is automatically created with the password stored in a Kubernetes secret. Save the password as we will use it to configure the filebeats and access the Kibana.
```
kubectl get secret quickstart-es-elastic-user -o=jsonpath='{.data.elastic}' | base64 --decode; echo
```

##### Step 3.4 -- Deploy kibana instance
This yaml file is included on this repo as well. You can either clone this repo from within the VM or you can just create a file and copy the contentsm then proceed to apply.
```
kubectl apply -f kibana-elk.yaml
```

##### Step 3.5 -- Deploy the filebeats instance
This yaml file is included on this repo as well. You can either clone this repo from within the VM or you can just create a file and copy the contentsm then proceed to apply.
**NOTE:** Before you proceed to apply, please check the file contents, there's an attribute wherein you need to supply the password that we have retrieved from Step 3.3.
```
kubectl apply -f filebeats-elk.yaml
```

##### Step 3.6 -- Test Running Kibana
```
kubectl port-forward service/kibana-name-kb-http 5601
```
After running the port-forward command, open a browser and you can now access Kibana at https://localhost:5601. Just keep on the SSL warning as we provided no certificate for this. 

- **Username:** elastic
- **Password:** _retrieved from Step 3.3_


