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
