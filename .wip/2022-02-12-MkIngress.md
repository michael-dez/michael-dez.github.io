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
## Create an Ingress Resource
Here's my example yaml, `jenkins-ingress.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: jenkins-ingress
spec:
  rules:
    - host: jenkins.bigkea.com
      http:
    paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: jenkins
            port:
              number: 8080
```
Create the resource.
```bash
k apply -f <file> # e.g. jenkins-ingress.yaml
```
