# Kubernetes Engine: Qwik Start

## 1. Set a default compute zone
```
gcloud config set compute/region us-west4
```

<br>

## 2. Create a GKE cluster
```
gcloud container clusters create --machine-type=e2-medium --zone=us-west4-a lab-cluster 
```

<br>

## 3. Get authentication credentials for the cluster
```
gcloud container clusters get-credentials lab-cluster 
```

<br>

## 4. Deploy an application to the cluster
1. Create a new Deployment hello-server from the hello-app container image.
```
kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
```

2. Create a Kubernetes Service which is a Kubernetes resource that lets you expose your application to external traffic.
```
kubectl expose deployment hello-server --type=LoadBalancer --port 8080
```

3. Inspect the hello-server Service.
```
kubectl get service
```

4. Finally, view the application from your web browser!
```
http://[EXTERNAL-IP]:8080
```

<br>

## 5. Deleting the cluster
```
gcloud container clusters delete lab-cluster 
```
