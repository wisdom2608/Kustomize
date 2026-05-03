Folder structure

```bash
K8s/
└── manifests/
    ├── nginx-namespace.yml
    ├── nginx-configmap.yml
    ├── nginx-deployment.yml
    └── nginx-service.yml
```
File contents

a). K8s/manifests/nginx-namespace.yml

```yml
#nginx-namespace.yml

apiVersion: v1
kind: Namespace
metadata:
  name: nginx
```
b). K8s/manifests/nginx-configmap.yml

```yml
#nginx-configmap.yml

apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
  namespace: nginx
data:
  index.html: |
    <html>
      <body>
        <h1>Welcome to Nginx on Kubernetes</h1>
        <p>Deployed via Kustomize</p>
      </body>
    </html>
```

c). K8s/manifests/nginx-deployment.yml

```yml
#nginx-deployment.yml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: nginx
  labels:
    app: nginx
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
        image: nginx:1.27
        ports:
                - containerPort: 80
        volumeMounts:
                - name: html-content
          mountPath: /usr/share/nginx/html
      volumes:
            - name: html-content
        configMap:
          name: nginx-config
```
d). K8s/manifests/nginx-service.yml

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
    - port: 80
    targetPort: 80
    nodePort: 30080
    protocol: TCP
```

The ConfigMap mounts a simple `index. html` into nginx so you have something to test when you hit `http: // <NODE-IP>: 30080`.


We can run these files individualy and apply the yml objects
to kunernetes as shown below:

````bash
kubectl apply -f nginx-namespace.yml
kubectl apply -f nginx-configmap.yml
kubectl apply -f nginx-deployment.yml
kubectl apply -f nginx-service.yml
````

OR

```bash
kubectl apply -f K8s/manifests/nginx-namespace.yml
kubectl apply -f K8s/manifests/nginx-configmap.yml
kubectl apply -f K8s/manifests/nginx-deployment.yml
kubectl apply -f K8s/manifests/nginx-service.yml
```
*We can apply the entire folder*

Sometimes this is good enough to say let us just apply the entire folder and that folder becomes our package, or chart, etc.

For some people, this is good enough because you don't need any kind of complicate templating, and parameter injection, you can just take a whole folder and deploy your infrastructure.


```bash
kubectl apply -f K8s/manifests
```

We can bundle everything (yml objects) into a single K8s configuration management tool called `kustomizeation.yml`

The folder structure will look like this.

```bash
K8s/
└── manifests/
    ├── nginx-namespace.yml
    ├── nginx-configmap.yml
    ├── nginx-deployment.yml
    ├── nginx-service.yml
    └── kustomization.yml
```
Create a `kustomization.yml` file to bundle  the `namespace`, `configmap`, `deployment`, and `service` .*yml* objects

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

Run it

From the K8s directory:

```bash
kubectl apply -k K8s/manifests/
```

**Cleanup**

To remove everything 

```bash
kubectl delete -k K8s/manifests/
```





