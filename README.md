### k8s

### CKA Simulator Kubernetes 1.25

Una vez que haya obtenido acceso a su terminal, sería conveniente dedicar aproximadamente 1 minuto a configurar su entorno. Podrías configurar estos:

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
