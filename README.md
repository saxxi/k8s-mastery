# Setup local

## Dockr for mac

1) Open `Docker for Mac` -> `Preferences` -> `Enable K8s` (if you get an error go to `Preferences` -> `reset` -> `reset cluster`)
2) Install `kubernetes-cli` & `Kubernetes dashboard`

```
brew install kubernetes-cli
kubectl config use-context docker-for-desktop
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended/kubernetes-dashboard.yaml
kubectl get pod --namespace=kube-system | grep dashboard | awk '{print $1}'
kubectl port-forward $(kubectl get pod --namespace=kube-system | grep dashboard | awk '{print $1}') 8443:8443 --namespace=kube-system
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret |grep replicaset | awk '{print $1}')
```

Then copy the JTW output and use as login in this page:

```
https://localhost:8443
```

Launch service

```
kubectl apply -f sa-frontend-pod.yaml
kubectl port-forward sa-frontend 8009:80
```

Service should be visible on http://127.0.0.1:8009

## Attempt to scale manually (wrong way)

```
kubectl apply -f sa-frontend-pod.yaml
kubectl apply -f sa-frontend-pod2.yaml
kubectl get pod -l app=sa-frontend # --show-labels
```

```
kubectl apply -f service-sa-frontend-lb.yaml
kubectl describe services sa-frontend-lb
```

## Scale deploy (correct way)

```
kubectl apply -f sa-frontend-deployment.yaml
```


## Test Docker

### Frontened

```
docker build -f Dockerfile -t tmp-sentiment-analysis-frontend .
docker run -d -p 8000:80 tmp-sentiment-analysis-frontend
```

### Webapp (JAVA)

```
mvn install
docker build -f Dockerfile -t tmp-sentiment-analysis-web-app .
docker run -d -p 8080:8080 -e SA_LOGIC_API_URL='http://localhost:5000' tmp-sentiment-analysis-web-app
```

### Python

```
docker build -f Dockerfile -t tmp-sentiment-analysis-logic .
docker run -d -p 5050:5000 tmp-sentiment-analysis-logic
```
