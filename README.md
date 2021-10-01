# Kubernetes Nginx PHP-FPM App

In this tutorial, we will deploy a PHP application on a Kubernetes cluster with Nginx and PHP-FPM running in separate containers.

We want the web-server nginx and phpfpm to be co-located in separate pods.
![alt text](https://i.ibb.co/R4P9YB0/kubernetes-nginx-phpfpm1-drawio.png)

### PHP-FPM
PHP-FPM is an implementation of Fast-CGI for PHP with improved capabilities around process management, logging, and high traffic situations.

### Nginx
Nginx is a web server and reverse proxy that’s widely used for high traffic applications. When run in combination with PHP-FPM, Nginx is configured to send requests for .php routes to PHP-FPM to serve the page.

We should upload the application files first, to all worker nodes to the directory /var/website as /var/website is volume path to containers.

#### nginx-deployment.yml

~~~
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
    layer: frontend
spec:
    
  replicas: 6
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
          image: nginx:alpine
          ports:
            - containerPort: 80
          volumeMounts:
            
            - mountPath: /var/www/html/
              name: contents
                
            - name: nginx-config
              mountPath: /etc/nginx/conf.d/default.conf
              subPath: nginx.conf
                
      volumes:
        - name: contents
          hostPath:
            path: /var/website
            type: Directory
                
        - name: nginx-config
          configMap:
            name: nginx
~~~

#### nginx-nodeport.yml

~~~
---
kind: Service 
apiVersion: v1 
metadata:
  name: nginx
  labels:
    app: nginx
    layer: frontend
        
spec:

  type: NodePort
  selector:
    
    app: nginx

  ports:
   
    - nodePort: 30000
      port: 80
      targetPort: 80
~~~

#### nginx.conf

~~~
server {
  listen 80;
  listen [::]:80;
  access_log off;
  root /var/www/html;
  index index.php;
  server_name example.com;
  server_tokens off;
  location / {
    # First attempt to serve request as file, then
    # as directory, then fall back to displaying a 404.
    try_files $uri $uri/ /index.php?$args;
  }
  # pass the PHP scripts to FastCGI server listening on wordpress:9000
  location ~ \.php$ {
    fastcgi_split_path_info ^(.+\.php)(/.+)$;
      
    # Change The Service Name
    fastcgi_pass phpfpm:9000;
    fastcgi_index index.php;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_param SCRIPT_NAME $fastcgi_script_name;
  }
}
~~~

#### phpfpm-deployment.yml

~~~
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: phpfpm
  labels:
    app: phpfpm
    layer: backend
spec:
    
  replicas: 6
  selector:
    matchLabels:
      app: phpfpm
  template:
    
    
    metadata:
      labels:
        app: phpfpm
            
    spec:
    
      containers:
      
        - name: phpfpm
          image: php:fpm-alpine
          ports:
            - containerPort: 9000
          volumeMounts:
            - mountPath: /var/www/html/
              name: contents
        
      volumes:
        - name: contents
          hostPath:
            path: /var/website
            type: Directory
~~~

#### phpfpm-clusterip.yml

~~~
---
kind: Service 
apiVersion: v1 
metadata:
  name: phpfpm
  labels:
    app: phpfpm
    layer: backend
        
spec:

  type: ClusterIP
  selector:
    
    app: phpfpm

  ports:
    
    - port: 9000
      targetPort: 9000
~~~

## Results: 

We will get below results once we run the above deployments.

![Alt Text](https://i.ibb.co/ncCK0SK/pic1.png)

![Alt Text](https://i.ibb.co/54CyQ13/pic2.png)

![Alt Text](https://i.ibb.co/8DmX6yQ/pic3.png)

![Alt Text](https://i.ibb.co/L6Gtcy6/pic4.png)

## Author
Created by [@sebinxavi](https://www.linkedin.com/in/sebinxavi/) - feel free to contact me and advise as necessary!

<a href="mailto:sebin.xavi1@gmail.com"><img src="https://img.shields.io/badge/-sebin.xavi1@gmail.com-D14836?style=flat&logo=Gmail&logoColor=white"/></a>
<a href="https://www.linkedin.com/in/sebinxavi"><img src="https://img.shields.io/badge/-Linkedin-blue"/></a>
