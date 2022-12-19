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
