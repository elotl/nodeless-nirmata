# Manage Nodeless Kubernetes clusters with Nirmata

### Step 1: Create 1-worker Nodeless Kubernetes cluster

[Follow instructions in this repo](https://github.com/elotl/kubeadm-aws) to create a {1 master, 1 worker} Nodeless Kubernetes cluster.

Log on to Kubernetes master, verify cluster is up.

```
$ ssh -i "myechuri-key2.pem" ubuntu@100.26.23.126

$ kubectl cluster-info
Kubernetes master is running at https://10.0.23.147:6443
KubeDNS is running at https://10.0.23.147:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

$ kubectl get nodes -o wide
NAME                          STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP     OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
ip-10-0-20-188.ec2.internal   Ready    worker   28d   v1.16.1   10.0.20.188   54.227.45.93    Ubuntu 16.04.6 LTS   4.4.0-1092-aws   containerd://1.2.6
ip-10-0-23-147.ec2.internal   Ready    master   28d   v1.16.1   10.0.23.147   100.26.23.126   Ubuntu 16.04.6 LTS   4.4.0-1092-aws   docker://18.9.7
```

### Step 2: Import Nodeless cluster into Nirmata

Go to Nirmata Dashboard and initiate `Create Cluster` workflow.

![alt text](https://github.com/elotl/nodeless-nirmata/blob/master/nirmata-create-cluster-1.png "Create Cluster 1")

Select `Managing an existing Kubernetes Cluster` option.

![alt text](https://github.com/elotl/nodeless-nirmata/blob/master/nirmata-create-cluster-2.png "Create Cluster 2")

Fill in cluster `Name` and set `Cloud Provider` to `Amazon Web Services`.

![alt text](https://github.com/elotl/nodeless-nirmata/blob/master/nirmata-create-cluster-3.png "Create Cluster 3")

Download `nirmata-kube-controller.yaml`.

![alt text](https://github.com/elotl/nodeless-nirmata/blob/master/nirmata-create-cluster-4.png "Create Cluster 4")

Scp `nirmata-kube-controller.yaml` to a node with `kubeconfig` access to Nodeless cluster.

```
$ scp -i myechuri-key2.pem /Users/myechuri/Downloads/nirmata-kube-controller.yaml ubuntu@100.26.23.126:/home/ubuntu/nirmata/
```

Apply `nirmata-kube-controller.yaml`.

```
$ kubectl create -f nirmata-kube-controller.yaml 
namespace/nirmata created
serviceaccount/nirmata created
clusterrole.rbac.authorization.k8s.io/nirmata:cluster-admin created
clusterrolebinding.rbac.authorization.k8s.io/nirmata-cluster-admin-binding created
deployment.apps/nirmata-kube-controller created
```

Verify that Nirmata namespace components are up and running.

```
$ kubectl get all -n nirmata
NAME                                           READY   STATUS    RESTARTS   AGE
pod/nirmata-kube-controller-859c54dd97-wdn48   1/1     Running   0          3m

NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nirmata-kube-controller   1/1     1            1           3m

NAME                                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/nirmata-kube-controller-859c54dd97   1         1         1       3m
```

On Nirmata dashboard, click `I have installed the Nirmata Kubernetes Cluster`.

![alt text](https://github.com/elotl/nodeless-nirmata/blob/master/nirmata-create-cluster-5.png "Create Cluster 5")

Your Nodeless k8s cluster is now fully managed by Nirmata!

![alt text](https://github.com/elotl/nodeless-nirmata/blob/master/nirmata-create-cluster-complete.png "Create Cluster Complete")

### Step 3: Deploy Nginx workload, manage via Nirmata dasboard

Deploy Nginx deployment with 3 replicas.

```
$ wget https://raw.githubusercontent.com/elotl/nodeless-nirmata/master/nginx.yaml

$ kubectl create -f nginx.yaml
```

All 3 replicas of Nginx will be deployed in a Nodeless fashion - just-in-time, right-sized, cost-effective compute will be started for each of the replicas, and the pods will be dispatched to the compute cells.

Nginx replicas can be visualized through Nirmata dashboard.

![alt text](https://github.com/elotl/nodeless-nirmata/blob/master/nirmata-nginx-overview.png "Nginx Deployment overview")


### Teardown

Follow [teardown instructions from kubeadm repo](https://github.com/elotl/kubeadm-aws#teardown).
