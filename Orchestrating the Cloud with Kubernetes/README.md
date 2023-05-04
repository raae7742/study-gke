## Orchestrating the Cloud with Kubernetes

### Google Kubernetes Engine
1. Set your compute/zone.
```
gcloud config set compute/zone us-central1-b
```

2. Start up a cluster for use.
```
gcloud container clusters create io
```

<br>

### Task 1. Get the sample code
1. Copy the source code and list the files in this directory.
```
gsutil cp -r gs://spls/gsp021/* .
cd orchestrate-with-kubernetes/kubernetes
ls
```

<br>

### Task 2. Quick Kubernetes Demo
1. Launch a single instance of the nginx container.
```
kubectl create deployment nginx --image=nginx:1.10.0
```

2. If you want to view the running nginx container, use the `kubectl get pods` command.
```
kubectl get pods
```

3. Once the nginx container has a Running status, you can expose it outside of Kubernetes.
```
kubectl expose deployment nginx --port 80 --type LoadBalancer
```

4. List our services now.
```
kubectl get services
```

5. Finally, hit the Nginx container remotely.
```
curl http://<External IP>:80
```

<br>

### Task 3. Pods
- logical application at the core of Kubernetes
- one or more containers and volumes
- shared namespaces -> share the attached **volumes** (= data disks).
- one IP per pod.

<br>

### Task 4. Creating pods
1. Create the monolith pod and Examine it.
```
cat pods/monolith.yaml
kubectl create -f pods/monolith.yaml
kubectl get pods
```

2. Once the pod is running, get more information about it.
```
kubectl describe pods monolith
```

<br>

### Task 5. Interacting with pods
1. Set up port-forwarding.
```
kubectl port-forward monolith 10080:80
```

2. Then, in the another terminal, start talking to your pod using `curl`.
```
curl http://127.0.0.1:10080
```

4. Create an environment variable for the jwt token, and copy it.
```
TOKEN=$(curl http://127.0.0.1:10080/login -u user|jq - r '.token')
```

5. Use the token to hit the secure endpoint.
```
curl -H "Authorization: Bearer $TOKEN" http://127.0.0.1:10080/secure
```

6. View the logs. (If you see these in real-time, use -f option.)
```
kubectl logs monolith
```

7. Run an interactive shell inside the pod and test external connectivity using the `ping`.
```
kubectl exec monolith --stdin --tty -c monolith -- /bin/sh
ping -c 3 google.com
# exit
```

<br>

### Task 6. Services
- stable/persistent enpoint for set of pods
- use labels to select pods
- internal or external IPs

**Service's Type**
- `ClusterIP` : default type, only visible inside of the cluster.
- `NodePort` : each node is an externally accessible IP
- `LoadBalancer` : forwards traffic from the services to Nodes within it.

<br>

### Task 7. Creating a service
1. Create a secure pod that can handle https traffic.
```
cat pods/secure-monolith.yaml
kubectl create secret generic tls-certs --from-file tls/
kubectl create configmap nginx-proxy-conf --from-file nginx/proxy.conf
kubectl create -f pods/secure-monolith.yaml
```

2. To expose the secure-monolith Pod externally, create a Kubernetes service.
```
cat services/monolith.yaml
kubectl create -f services/monolith.yaml
```

3. Allow traffic to the service on the exposed nodeport.
```
gcloud compute firewall-rules create allow-monolith-nodeport \
  --allow=tcp:31000
```

<br>

### Task 8. Adding labels to pods

1. You can search running pods with the labels.
```
kubectl get pods -l "app=monolith,secure=enabled"
```

2. Add the missing label to the secure-monolith Pod.
```
kubectl label pods secure-monolith 'secure=enabled'
kubectl get pods secure-monolith --show-labels
```

3. View the list of endpoints on the monolith service.
```
kubectl describe services monolith | grep Endpoints
```

4. Test this by hitting again.
```
gcloud compute instances list
curl -k https://<EXTERNAL_IP>:31000
```

<br>

### Task 9. Deploying applications with Kubernetes
- Drive current state towards desired state
- Replica Sets: manage stating and stopping the Pods.

<br>

### Task 10. Creating deployments
We are going to break the monolith app into 3 separate pieces. (auth / hello / frontend)

1. Create the auth, hello and front deployments and services.
```
kubectl create -f deployments/auth.yaml
kubectl create -f services/auth.yaml

kubectl create -f deployments/hello.yaml
kubectl create -f services/hello.yaml

kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
kubectl create -f deployments/frontend.yaml
kubectl create -f services/frontend.yaml
```

2. Interact with the frontend.
```
kubectl get services frontend
curl -k https://<EXTERNAL-IP>
```
