To interact with Kubernetes, we use YML files. There are multiple ways of dealing with YML files and complicated ways such as HELM chart. In this lesson, we’ll be taking a look at a much simpler approach for bundling simple kubernetes with a tool called *kustomize*.

Kubernetes allows to describe our infrastructures, and our applications as YML files.

Let’s take a look at the traditional way of applying YML objects to a kubernetes and then we are going to take a look at what kustomize does to help us improve the process. 

In the basic way, we normally use the popular `kubectl get apply -f` to deal with YML files. For example, let’s consider the following kubernetes YML files.

1) `nginx-namespaces.yml`
```yml
#nginx-namespace.yml
apiVersion: v1
kind: Namespace
metadata:
  name: nginx
```
Namespace allows us to group objects together. It’s a great way for a company to split out different applications between different departments as well as separating infrastructures such as monitoring components, login components, ingress control etc.

 2) An application may need some configurations to function. This requires us to create a configMap.
`nginx-configmap.yml`


```yml
#configmap.yml

apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
  namespace: nginx
data:
  default.conf: |
    server {
        listen 80;
        server_name localhost;

        location / {
            return 200 'Welcome to NGINX running in Kubernetes!';
            add_header Content-Type text/plain;
        }
    }
```
This replaces the default NGINX config with a simple custom page.

3) `deployment.yml`
Deployment is a way for us to describe an application to kubernetes. Kubernetes knows to how deploy an application and keep it up and running.

```yml
#nginx-deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-config-volume
          mountPath: /etc/nginx/conf.d/default.conf
          subPath: default.conf
      volumes:
      - name: nginx-config-volume
        configMap:
          name: nginx-config
```

In the development.yml file above, the kind of the application is a deployment. We are creating it in the `nginx` namespace, we want to run 2 replicas (nginx pods), we are running an nginx image, we are opening port 80. And finally we are mounting the ConfigMap into NGINX so it overrides the default config. 



*Let’s explain in detail what the ConfigMap here is doing.*

In Kubernetes, a ConfigMap is just a way to store configuration (like text files) outside your container image.

Here, we created one named:
 `nginx-config`

Inside of it, you stored a file:
`default.conf: |`
That file is a real NGINX configuration file.

*What this specific config does*

```nginx
server {
    listen 80;
    server_name localhost;

    location / {
        return 200 'Welcome to NGINX running in Kubernetes!';
        add_header Content-Type text/plain;
    }
}
```
a. *server { ... }*

This defines a virtual server block in NGINX.

b. *listen 80;*

NGINX listens for HTTP traffic on port 80 inside the container.

c. *server_name localhost;*

It responds to requests targeting localhost (in practice, it will still respond to your Node IP).

d. *location / { ... }*

This means:

“For any request coming to / (the root URL), do the following…”

e. return 200 'Welcome to NGINX running in Kubernetes!';

Instead of serving a website, it:
	•	Immediately returns HTTP status 200 (OK)
	•	Sends back this plain text message

So NGINX becomes a simple API-like responder, not a web server serving files.

f. *add_header Content-Type text/plain;*

This tells the browser:

“Treat this response as plain text”

Without this, the browser might guess incorrectly.

*How Kubernetes uses it*
In your Deployment, this part is key:

```yml
volumeMounts:
- name: nginx-config-volume
  mountPath: /etc/nginx/conf.d/default.conf
  subPath: default.conf
```
This means:
	•	Take the default.conf from the ConfigMap
	•	Mount it as a file
	•	Replace the default NGINX config at:

```code
/etc/nginx/conf.d/default.conf
```
So when NGINX starts:
➡️ It uses your config instead of its default one


4) `nginx-service.yml`

A service is a way to expose applications in your cluster, either privately or via a load balancer. 

```yml
#nginx-service.yml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: nginx
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80        # Service port (inside cluster)
      targetPort: 80  # Container port
      nodePort: 30007 # External port (must be between 30000–32767)
```
In the file above, we are creating a service named `nginx-service` in the `nginx-namespace`. We are creating a service type of a load balancer. That means, because kubernetes is a cloud-native, it knows how to provision load balancers wherever it’s running. So, if you are running docker for desktop,  it will create a virtual load balancer so you can access your application over localhost.


To apply the above yml objects to kubernetes environment, we will use the popular `kubectl` command in the following order:

```bash
kubectl apply -f nginx-namespace.yml
kubectl apply -f nginx-configmap.yml
kubectl apply -f nginx-deployment.yml
kubectl apply -f nginx-service.yml
```



```
```

```
```

```
```
