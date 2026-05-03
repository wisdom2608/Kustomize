To interact with Kubernetes, we use YML files. There are multiple ways of dealing with YML files and complicated ways such as HELM chart. In this lesson, we’ll be taking a look at a much simpler approach for bundling simple kubernetes with a tool called *kustomize*.

Kubernetes allows to describe our infrastructures, and our applications as YML files.

```bash
K8s/
└── manifests/
    ├── nginx-namespace.yml
    ├── nginx-configmap.yml
    ├── nginx-deployment.yml
    └── nginx-service.yml
```
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

We can apply the entire folder

Sometimes this is good enough to say let us just apply the entire folder and that folder becomes our package, or chart, etc.

For some people, this is good enough because you don't need any kind of complicate templating, and parameter injection, you can just take a whole folder and deploy your infrastructure.

```bash
kubectl apply -f K8s/manifests
```
We we. Want to deploy things like Jenkins, ArgoCD, etc, you can basically use yaml folder by saying 
`kubectl apply -f`. This is because it’s easy to digest, easy to read, and easy to follow and understand.  

With kubernetes, you don’t need complex abstractions, complex template rendering, complicated nested templates. All of these complexities may make it much harder for people to read, digest, follow and understand. It also makes it harder to upgrade and maintain your systems.

We recommend you stick to the very basic if possible. If you need something more complicated, use something like `kustomize`. Also try to keep your folders and yml files structure flat and simple.

**Kubernetes native configuration management**

Kustomize introduces a *template-free* way to customize application configutation that simplifies the use of off-the-shelf applications. Now, built into `kubectl` as `apply -k`
We can take out our `deployment.yml` file that and we can keep it untouched. What we we can do is that we can introduce patches such as patches for dev, patches for stage and patches for prod. And then apply that with the tool called *kustomize* and deploy them to our kubernetes environment.

Let add `kustomize.yml` to have the subsequent new folder structure.

```bash
K8s/
└── manifests/
    ├── nginx-namespace.yml
    ├── nginx-configmap.yml
    ├── nginx-deployment.yml
    ├── nginx-service.yml
    └── kustomization.yml
```

We can keep our previous  `nginx-namespace.yml`, `nginx-configmap.yml`, `nginx-deployment.yml`, and `nginx-service.yml` intact, and unchanged and just apply the necessary changes that we need to our *dev*, *staging*, and *prod* environment.

To start we kustomize, we need a `kustomization.yml` that describes what we want our bundle to look like. So we in the *kustomization.yml* file, we introduce resources and list all the resources we want in our bundle.
So, we’ll include all the yml files- `nginx-namespace.yml`, `nginx-configmap.yml`, `nginx-deployment.yml`, and `nginx-service.yml` that we have in the K8s/manifests folder.

`kustomization.yml`

```yml
#kustomization.yml

apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
    - nginx-namespace.yml
    - nginx-configmap.yml
    - nginx-deployment.yml
    - nginx-service.yml
```
With the latest version of kubernetes, a Kustomize binary have been included to kubectl, which makes it really easy to use. 

We can easily apply our yml objects to kubernetes environment using kustomize.

```bash
kubectl Kustomize ./K8s/manifests/ | kubectl apply -f 
```
Or

```bash
kubectl apply -k ./K8s/manifests/
```

```
```

```
```

```
```
