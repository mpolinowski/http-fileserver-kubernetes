# Kubernetes HTTP Fileserver

## Serving static files from a Docker Container

1. Create a Dockerfile to work with the [official NGINX Docker Container](https://hub.docker.com/_/nginx) that will copy all the content from the __download folder__ to the __public directory__ of the NGINX server:


```yaml
FROM nginx
COPY downloads/ /usr/share/nginx/html
```


2. Build the image and tag it for your Docker repository:


```bash
docker build -t my-docker-hub-account/http-fileserver-kubernetes .
```


3. (Optional) Test run your container:


```bash
docker run -d -p 8080:80 my-docker-hub-account/http-fileserver-kubernetes
```

You should now be able to access the files you stored inside the __downloads folder__ via `http://localhost:8080/dl/test.txt` (assuming that you used the __downloads__ folder from the repository, that contains a sub directory __dl__ that contains a text file __test.txt__):


![Docker](./image_01.png)



4. Push the Docker Image to Docker hub


```bash
docker login
docker push my-docker-hub-account/http-fileserver-kubernetes
```


Alternatively, do your build on the host system where you want to run the image - so the image is in local storage. This is a __hacky solution__ that is not fully supported - if you don't want to push your image to the Docker Hub, you are supposed to use a local/personal registry for your Docker images. That feels a little bit much for my problem here - so I will proceed with this way.

__Note__: if you want to use the local image, you have to build the image on every Node server that you are using in Kubernetes and that might be used to spawn the pod. You also have to set the `imagePullPolicy= Never` (see YAML file in the next step)! This will make sure that Kubernetes will always use the image from local storage instead of trying to pull it from Docker Hub.



5. Creating the Kubernetes Deployment


```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    service: http-fileserver
  name: http-fileserver
spec:
  replicas: 1
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        service: http-fileserver
    spec:
      containers:
      - image: my-docker-hub-account/http-fileserver-kubernetes:latest
        imagePullPolicy: Always
        name: http-fileserver
        resources: {}
      restartPolicy: Always
status: {}

---

apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    service: http-fileserver
  name: http-fileserver
spec:
  ports:
    - name: http
      port: 80
  selector:
    service: http-fileserver
status:
  loadBalancer: {}
```


Alternatively use a simple pod, instead of a deployment:



```yaml
kind: Pod
apiVersion: v1
metadata:
  name: http-fileserver
  labels:
    app: fileserver
spec:
  containers:
    - name: http-fileserver
      image: my-docker-hub-account/http-fileserver-kubernetes:latest
      imagePullPolicy: Always
      ports:
        - containerPort: 80

---

kind: Service
apiVersion: v1
metadata:
  name: http-fileserver
spec:
  selector:
    app: fileserver
  ports:
    - port: 80
```



6. You can run the deployment (or pod) by the Kubernetes command - see [Kubernetes Ingress](https://mpolinowski.github.io/kubernetes-nginx-ingress/):



```bash
kubectl create -f http-fileserver.yaml
```


![Docker](./image_02.png)




