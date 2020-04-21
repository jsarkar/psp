## What is PodSecurityPolicy(PSP) ?
A `PodSecurityPolicy` (PSP) is cluster level resource that controls security sensitive aspects of the Pod specefications via Kubernetes Admission Control.

## How to enable PodSecurityPolicy (PSP) into the Cluster?
* Set of policies (At least 2. A permisive policy for Kube-System namespace and restrictive policy for rest of the workload running in the cluster)
* RBAC for PSP Policies. (The policies are notthing but a Kubernetes object. In order to something in Kubernetes to use these policies, we have to create associated Role-base access control (RBAC))
* Last but not least, Activate the `PodSecurityPolicy` via admission control flag in Kube-Api Server (`--enable-admission-plugins=PodSecurityPolicy`). I purposely mentiond this last to stress the point, only enabling the PSP in Kube-API server and not having policy and assocated RBAC will make cluster non-operational.

## What are the steps to bring up the cluster along with PSP?
* Start the KIND Cluster. Check the `kind-config.yaml`. The PSP flag is enabled in the Kube-API Server
  ```
  kind create cluster --config=kind-config.yaml
  ```
* Apply the restricted policy along with RBAC
  ```
  kubectl apply -f restricted-psp.yaml
  kubectl apply -f restricted-rbac.yaml
  ```
* Apply the priviledge policy along with RBAC
  ```
  kubectl apply -f privileged-psp.yaml
  kubectl apply -f privileged-rbac.yaml
  ```
* make sure all critical Pods are up and running
  ```
  kubectl get pods -A
  kubectl get nodes
  ```
## We have PSP configured. Now lets discuss about the verious Control Aspects (#16) for pod specefications

### seccomp a.k.a Secure Compute Mode Profile
Secure compute (seccomp) model is feature, which focuses on limiting what system calls our containers will be able to execute

I have created a custome seccomp profile, which restrict to run `ls` command inside a continer. I have tested the same by running :

```
docker container run --rm -it --security-opt seccomp=default-no-ls.json alpine sh
/ # ls
sh: lseek: Operation not permitted
```

Now, I am trying activate the same profile in Kubernetes. 

I have created configMap for the seccomp profile.

I placed th profile inside the node /var/lib/kubelet/seccomp(using a daemonset)

and then created a deployment with following annotations:
container.security.alpha.kubernetes.io/alpine: "localhost/my-profile.json"



