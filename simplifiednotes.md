Folder structure

```code
K8s/
└── manifests/
    ├── nginx-namespace.yml
    ├── nginx-configmap.yml
    ├── nginx-deployment.yml
    └── nginx-service.yml
```
File contents

K8s/manifests/nginx-namespace.yml

```yml
apiVersion: v1
kind: Namespace
metadata:
  name: nginx
```
K8s/manifests/nginx-configmap.yml

```yml
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
K8s/manifests/nginx-deployment.yml

```yml
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
K8s/manifests/nginx-service.yml

```yml
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
we can run these files individualy to apply the yml objects
to kunernetes as shown below:

````bash
kubectl apply -f nginx-namespace.yml
kubectl apply -f nginx-configmap.yml
kubectl apply -f nginx-deployment.yml
kubectl apply -f nginx-service.yml
````
