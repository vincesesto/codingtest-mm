## Running httpbin in Kubernetes technical test

App running on kubernetes namespace httpbin-dev
Routing set up via nginx proxy cause I couldn't get ingress-controller working

```
kubectl get namespace | grep httpbin-dev
httpbin-dev       Active   106m

kubectl get pods -n httpbin-dev
NAME                           READY   STATUS    RESTARTS   AGE
httpbin-7556469ddd-b4ms9       1/1     Running   0          101m
nginx-proxy-5f78f9c566-mdgjq   1/1     Running   0          8m

kubectl -n httpbin-dev get deployments
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
httpbin       1/1     1            1           101m
nginx-proxy   1/1     1            1           8m5s
```

# How I set everything up

1. Create a namespace
```
kubectl create namespace httpbin-dev
```
3. Create the httpbin deployment
```
kubectl -n httpbin-dev create deployment httpbin --image=kennethreitz/httpbin
```
4. Testing on the pod to make sure it is all working
```
curl -v http://localhost/status/200 
*   Trying ::1...
...> 
< HTTP/1.1 200 OK
```
4. Expose the deployment via a service...Create the service yaml with
```
kubectl -n httpbin-dev expose deployment httpbin --name=httpbin-service --port=80 --target-port=80 --dry-run=client -o yaml > service.yaml

# then run apply 
kubectl -n httpbin-dev apply -f service.yaml 
```
5. Installing Helm
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```
6. Installing ingress controllers
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace
```

NOTE: After a lot of messing around, it looks like I could not get this working...Possibly an issue with the ports available to the ingress controller

7. Set up NGINX to instead run as a proxy

```
# Create a config map with routing for the httpbin service and deploy
kubectl -n httpbin-dev apply -f configmap.yaml

# Create deployment for nginx to use the configmap and deploy
kubectl -n httpbin-dev apply -f nginx-deploy.yaml

# Expose the nginx service and create it as a LoadBalancer to access externally
kubectl -n httpbin-dev apply -f nginx-service.yaml
```



 
