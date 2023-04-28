# Introduction to Docker
## Activate Cloud Shell
1. You can list the activate account name with this command.
```
gcloud auth list
```

2. You can list the project ID with this command.
```
gcloud config list project.
``` 

<br>

## Task 1. Hello world
1. Run a hello world container.
```
docker run hello-world
```


2. Take a look at the container image it pulled from Docker Hub.
```
docker images
```

<br>

If you run this image one more time, the Docker daemon finds this image in your local registry and runs the container from that. 
<br>
It doesn't have to pull the image from Docker Hub.

<br>

3. Look at the running containers.
```
# -a option: see all containers including ones that have finishied executing
docker ps
```

<br>

The container names can be specified with `docker run --name [container-name] [docker-image]`

<br>

## Task 2. Build
1. Create a `Dockerfile`.
```
cat > Dockerfile <<EOF
FROM node:lts
WORKDIR /app
ADD . /app
EXPOSE 80
CMD ["node", "app.js"]
EOF
```

2. Create the node application.
```
cat > app.js <<EOF
const http = require('http');
const hostname = '0.0.0.0';
const port = 80;
const server = http.createServer((req, res) => {
    res.statusCode = 200;
    res.setHeader('Content-Type', 'text/plain');
    res.end('Hello World\n');
});
server.listen(port, hostname, () => {
    console.log('Server running at http://%s:%s/', hostname, port);
});
process.on('SIGINT', function() {
    console.log('Caught interrupt signal and will exit');
    process.exit();
});
EOF
```

3. Build the image.
```
docker build -t node-app:0.1 .
```

4. You can find this image when running the following command.
```
docker images
```

<br>

## Run
1. Run containers based on the image you built.
```
# -d option: start the container in the background.
docker run -p 4000:80 --name my-app node-app:0.1
```

2. Then, test the server.
```
curl http://localhost:4000
```

3. Look at the logs.
```
docker logs [container_id]
```

4. Stop and remove the container.
```
docker stop my-app && docker rm my-app
```

<br>

## Debug
1. Start an interactive Bash session inside the running container.
```
docker exec -it [container_id] bash
```

2. Exit the Bash session.
```
exit
```

3. Examine a container's metadata in Docker by using Docker inspect.
```
# --format: inspect specific fields
docker inspect [container_id]
```

<br>

## Publish
1. Create the target Docker repository by using Google Artifact Registry.
2. Configure authentication to Artifact Registry.
```
gcloud auth configure-docker us-central1-docker.pkg.dev
```
3. Set your project ID.
```
export PROJECT_ID=$(gcloud config get-value project)
```
4. Tag `node-app:0.2`.
```
docker build -t us-central1-docker.pkg.dev/$PROJECT_ID/my-repository/node-app:0.2 .
```
5. Push this image to Artifact Registry.
```
docker push us-central1-docker.pkg.dev/$PROJECT_ID/my-repository/node-app:0.2
```
6. Remove all installed images. 
```
docker rmi -f $(docker images -aq)
```
7. Pull the image and run it.
```
docker pull us-central1-docker.pkg.dev/$PROJECT_ID/my-repository/node-app:0.2
docker run -p 4000:80 -d us-central1-docker.pkg.dev/$PROJECT_ID/my-repository/node-app:0.2
curl http://localhost:4000
```
