### k8s

### CKA Simulator Kubernetes 1.25

Once you've gained access to your terminal it might be wise to spend ~1 minute to setup your environment. You could set these:

```bash
alias k=kubectl                         # will already be pre-configured
export do="--dry-run=client -o yaml"    # k create deploy nginx --image=nginx $do
export now="--force --grace-period 0"   # k delete pod x $now
```
#
### Question 1 | Contexts

You have access to multiple clusters from your main terminal through `kubectl` contexts. Write all those context names into `/opt/course/1/contexts`.

Next write a command to display the current context into `/opt/course/1/context_default_kubectl.sh`, the command should use `kubectl`.

Finally write a second command doing the same thing into `/opt/course/1/context_default_no_kubectl.sh`, but without the use of `kubectl`.


**Answer**:

Maybe the fastest way is just to run:

```bash
k config get-contexts # copy manually
k config get-contexts -o name > /opt/course/1/contexts
```
Or using jsonpath:

```bash
k config view -o yaml # overview
k config view -o jsonpath="{.contexts[*].name}"
k config view -o jsonpath="{.contexts[*].name}" | tr " " "\n" # new lines
k config view -o jsonpath="{.contexts[*].name}" | tr " " "\n" > /opt/course/1/contexts 
```

The content should then look like:
```bash
~# cat /opt/course/1/contexts
k8s-c1-H
k8s-c2-AC
k8s-c3-CCC
```

Next create the first command:
```bash
~# cat /opt/course/1/context_default_kubectl.sh
kubectl config current-context
```

```bash
~# sh /opt/course/1/context_default_kubectl.sh
k8s-c1-H
```

And the second one:
```bash
~# cat /opt/course/1/context_default_no_kubectl.sh
cat ~/.kube/config | grep current
```

```bash
~# sh /opt/course/1/context_default_no_kubectl.sh
current-context: k8s-c1-H
```

In the real exam you might need to filter and find information from bigger lists of resources, hence knowing a little jsonpath and simple bash filtering will be helpful.

The second command could also be improved to:
```bash
~# cat /opt/course/1/context_default_no_kubectl.sh
cat ~/.kube/config | grep current | sed -e "s/current-context: //"
```

#
### Question 2 | Schedule Pod on Master Node
Use context: `kubectl config use-context k8s-c1-H`

Create a single Pod of _image_ `httpd:2.4.41-alpine` in _Namespace_ `default`. The Pod should be named `pod1` and the container should be named `pod1-container`. This Pod should **only** be scheduled on a master **node**, do not add new labels any nodes.

**Answer**:

First we find the master node(s) and their taints:
```bash
k get node # find master node
k describe node cluster1-master1 | grep Taint -A1 # get master node taints
k get node cluster1-master1 --show-labels # get master node labels
```

Next we create the Pod template:
```bash
# check the export on the very top of this document so we can use $do
k run pod1 --image=httpd:2.4.41-alpine $do > 2.yaml
vim 2.yaml
```
Perform the necessary changes manually. Use the Kubernetes docs and search for example for tolerations and nodeSelector to find examples:

```yaml
# 2.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod1
  name: pod1
spec:
  containers:
  - image: httpd:2.4.41-alpine
    name: pod1-container                       # change
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  tolerations:                                 # add
  - effect: NoSchedule                         # add
    key: node-role.kubernetes.io/control-plane # add
  nodeSelector:                                # add
    node-role.kubernetes.io/control-plane: ""  # add
status: {}
```
Important here to add the toleration for running on master nodes, but also the nodeSelector to make sure it only runs on master nodes. If we only specify a toleration the Pod can be scheduled on master or worker nodes.


Now we create it:
`k -f 2.yaml create`

Let's check if the pod is scheduled:
```bash
➜ k get pod pod1 -o wide
NAME   READY   STATUS    RESTARTS   ...    NODE               NOMINATED NODE
pod1   1/1     Running   0          ...    cluster1-master1   <none>  
```


#
### Question 3 | Scale down StatefulSet

Use context: `kubectl config use-context k8s-c1-H`

There are two _Pods_ named `o3db-*` in Namespace `project-c13`. C13 management asked you to scale the _Pods_ down to one replica to save resources.
 
**Answer**:

If we check the Pods we see two replicas:
```yaml
➜ k -n project-c13 get pod | grep o3db
o3db-0                                  1/1     Running   0          52s
o3db-1                                  1/1     Running   0          42s
```
From their name it looks like these are managed by a _StatefulSet_. But if we're not sure we could also check for the most common resources which manage Pods:

```yaml
➜ k -n project-c13 get deploy,ds,sts | grep o3db
statefulset.apps/o3db   2/2     2m56s
```

Confirmed, we have to work with a _StatefulSet_. To find this out we could also look at the Pod labels:

```yaml
➜ k -n project-c13 get pod --show-labels | grep o3db
o3db-0        1/1     Running   0   3m29s   app=nginx,controller-revision-hash=o3db-5fbd4bb9cc,statefulset.kubernetes.io/pod-name=o3db-0
o3db-1        1/1     Running   0   3m19s   app=nginx,controller-revision-hash=o3db-5fbd4bb9cc,statefulset.kubernetes.io/pod-name=o3db-1
```
To fulfil the task we simply run:

```yaml
➜ k -n project-c13 scale sts o3db --replicas 1
statefulset.apps/o3db scaled
```

```yaml
➜ k -n project-c13 get sts o3db
NAME   READY   AGE
o3db   1/1     4m39s
```
C13 Mangement is happy again.

#
### Question 4 | Pod Ready if Service is reachable
```bash
```
