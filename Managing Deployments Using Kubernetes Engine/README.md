## Managing Deployments Using Kubernetes Engine

### Set the zone
```
gcloud config set compute/zone us-east5-b
```

<br>

### Get sample code for this lab

1. Get the sample code
```
gsutil -m cp -r gs://spls/gsp053/orchestrate-with-kubernetes .
cd orchestrate-with-kubernetes/kubernetes
```

<br>

2. Create a cluster with 3 nodes.
```
gcloud container clusters create bootcamp \
  --machine-type e2-small \
  --num-nodes 3 \
  --scopes "https://www.googleapis.com/auth/projecthosting,storage-rw"
```


<br>

### Learn about the deployment object
1. `kubectl explain`: show the information about the deployment object.
```
# --recursive option: show all of the fields
kubectl explain deployment
kubectl explain deployment.metadata.name
```


<br>

<br>

### Create a deployment
1. Update the `deployments/auth.yaml` configuration file.
```
vi deployments/auth.yaml

# change the image in the containers section of the deployment
...
containers:
- name: auth
  image: "kelseyhightower/auth:1.0.0"
...
```


<br>

2. `kubectl create` : create the deployment object.
```
kubectl create -f deployments/auth.yaml
```

<br>

3. `kubectl get deployments`: verify that the deployment was created.
```
kubectl get deployments
```

<br>


4. `kubectl get replicasets`: verify that a ReplicaSet was created for out deployment.
```
kubectl get replicasets
```

<br>


5. `kubectl get pods`: view the Pods that were created as part of our deployment.
```
kubectl get pods
```

<br>

6. Then, create the auth service.
```
kubectl create -f services/auth.yaml
```

<br>

7. Do the same thing to create and expose the hello, frontend deployemtn.
```
kubectl create -f deployments/hello.yaml
kubectl create -f services/hello.yaml

kubectl create secret generic tls-certs --from-file tls/
kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
kubectl create -f deployments/frontend.yaml
kubectl create -f services/frontend.yaml
```

<br>

8. Get the frontend's external IP and then curl it.
```
kubectl get services frontend
curl -ks https://<EXTERNAL-IP>

<br>

# or
curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`
```

<br>

<br>

### Scale a deployment
1. `kubectl scale`: update the replicas field.
```
kubectl scale deployment hello --replicas=5
```

<br>

2. Verify that there are now 5 `hello` Pods running.
```
kubectl get pods | grep hello- | wc -1
```

<br>

<br>

### Rolling update
1. To trigger a rolling update, change the image in the containers section of the deployment.
```
kubectl edit deployment hello

...
containers:
  image: kelseyhightower/hello:2.0.0
...
```

<br>

2. See the new ReplicaSet and the rollout history.
```
kubectl get replicaset
kubectl rollout history deployment/hello
```

<br>

3. `kubectl rollout pause`: pause a running rollout to stop the update.
```
kubectl rollout pause deployment/hello
```

<br>

4. `kubectl rollout resume`: resume the paused rollout.
```
kubectl rollout resume deployment/hello
```

<br>

5. `kubectl rollout undo`: roll back to the previous version.
```
kubectl rollout undo deployment/hello
```

<br>

6. verify the result by using following commands
```
kubectl rollout status deployment/hello
kubectl rollout history deployment/hello
kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
```

<br>

<br>

### Canary deployments
1. Create the canary deployment.
```
kubectl create -f deployments/hello-canary.yaml
```

<br>

2. Run this command serveral times and verify that a small subset are served by hello 2.0.0.
```
curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
```

<br>

<br>

### Blue-green deployments
1. Update the service from hello.yaml to hello-blue.yaml.
```
kubectl apply -f services/hello-blue.yaml
```

<br>

2. Create the green deployment. (Current version of 1.0.0 is still being used.)
```
kubectl create -f deployments/hello-green.yaml
```

<br>

3. Update the service to point to the new version.
```
kubectl apply -f services/hello-green.yaml
```

<br>

4. Then, verify that the new version is always being used.
```
curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
```

<br>

5. While the "blue" deployment is still running, just update the service back to the old version.
```
kubectl apply -f services/hello-blue.yaml

curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
```
