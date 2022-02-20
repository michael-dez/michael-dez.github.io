# Create an Ingress in Minikube

**NOTE** `k` is an alias for `minikube kubectl --`
## Enable Ingress
Check to see if the ingress addon is enabled.
```bash
minikube addons list
```
If ingress shows disabled:
```bash
minikube addons enable ingress
```
Verify the ingress-nginx-controller is running.
```bash
k get pods -n ingress-nginx
# output should show a pod with name of ingress-nginx-controller
```
## 
