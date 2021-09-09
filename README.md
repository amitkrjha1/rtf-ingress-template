# rtf-ingress-template

## HAProxy Ingress template
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rtf-ingress-haproxy
  namespace: rtf
  annotations:
    kubernetes.io/ingress.class: rtf-haproxy
    haproxy-ingress.github.io/rewrite-target: /
    haproxy-ingress.github.io/ssl-redirect: "false"
  labels:
    business-group: business-group-id
    environment: environment-id
spec:
  rules:
  - host: meetupdemortfhaproxy.eastus.cloudapp.azure.com
    http:
      paths: 
      - pathType: Prefix
        path: /app-name
        backend: 
          service:
            name: service-name
            port:
              number: 80
```

## Create Secret
```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -out demo-nginx-ingress-tls.crt -keyout demo-nginx-ingress-tls.key -subj "/CN=sometthinghere.com/O=ingress-tls"
```
```
kubectl create secret tls nginx-ingress-tls --namespace rtf --key demo-nginx-ingress-tls.key --cert demo-nginx-ingress-tls.crt
```

## Nginx Ingress Template
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: aks-rtf-nginx
  namespace: rtf
#<1>  
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
#<2>    
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
#<3>    
  labels:
    environment: environment-id
    business-group: business-group-id
spec:
  ingressClassName: rtf-nginx
#<4>  
  tls:
#<5>  
  - hosts:
      - meetupdemortfnginx.eastus.cloudapp.azure.com
    secretName: nginx-ingress-tls
  rules:
  - host: meetupdemortfnginx.eastus.cloudapp.azure.com
    http:
      paths:
#<6>      
      - pathType: ImplementationSpecific
        path: /app-name/(.*)
#<7>        
        backend:
#<8>        
          service:
            name: service
            port:
              number: 80
```
Note the following about this example:
<1>  The template must use the rtf namespace.

<2>  The rewrite annotation is specific to Nginx as the ingress controller. The final version of the template varies depending on the controller you use.

<3>  Use of ssl-redirect varies depending on your ingress controller. Setting it to "true" redirects http traffic to https.

<4>  ingressClassName must be prefixed with rtf-, for example, rtf-nginx. 
This is how Runtime Fabric recognizes the object as a template. A template that uses the rtf- prefix in the ingressClassName, for example, rtf-nginx, is consumed by the Runtime Fabric agent only and not by the actual ingress controller. Ingress controllers discover only resources with an ingressClassName value that uses the vendor-specific name, for example, nginx or haproxy.

<5> TLS is optional. By default, Nginx uses SSL. if not using TLS, the annotation for ssl-redirect should be changed to nginx.ingress.kubernetes.io/ssl-redirect: "false"

<6> A template can include multiple hosts, but Runtime Manager displays the first path rule only.

<7> To ensure that each endpoint name is unique, use the 'app-name' placeholder in the host or path (if there is not a wildcard in the subdomain). 

<8>  These are placeholder values required for Kubernetes validation, but the actual values are not used by Runtime Fabric. 
