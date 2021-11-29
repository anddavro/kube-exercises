1. [Ingress Controller / Secrets] Crea los siguientes objetos de forma declarativa con las siguientes especificaciones:
• Imagen: nginx Version: 1.19.4
• 3 replicas
• Label: app: nginx-server
• Exponer el puerto 80 de los pods
• Limits:
CPU: 20 milicores
Memoria: 128Mi
• Requests:
CPU: 20 milicores
Memoria: 128Mi
Codigo Fuente:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-server
  labels:
    app: nginx-server-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
       app: nginx-server
  template:
    metadata:
      labels:
         app: nginx-server
    spec:
      containers:
      - name: nginx-server
        image: nginx:1.19.4
        ports:
         - containerPort: 80
        resources:
          requests:
            memory: "128Mi"
            cpu: "20m"
          limits:
            memory: "128Mi"
            cpu: "20m"
---
apiVersion: v1
kind: Service
metadata:
  name: service-nginx
spec:
  selector:
    app: nginx-server
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  namespace: default
spec:
  rules:
   - host: ad.rodriguez.student.lasalle.com
     http: 
      paths:
        - path: "/"
          pathType: Prefix
          backend:
           service:
            name: service-nginx
            port:
              number: 80

a.	A continuación, tras haber expuesto el servicio en el puerto 80, se deberá acceder a la página principal de Nginx a través de la siguiente:
URL http://ad.rodriguez.student.lasalle.com

Se adjunta documento en word con las imagenes del resultado

 
•	Resultado ingresando por consola y ejecutando curl -k
Se ingresa a uno de los pod creados en el replicaset del deployment a través del siguiente comando:
kubectl exec -it nginx-server-c8c84c768-dpp82 sh
y se ejecuta
curl -k http://ad.rodriguez.student.lasalle.com
 


b.	Una vez realizadas las pruebas con el protocolo HTTP, se pide acceder al servicio mediante la utilización del protocolo HTTPS, para ello:

•	Crear un certificado mediante la herramienta OpenSSL u otra similar 
•	Crear un secret que contenga el certificado

Certificado generado
 
Secret generado
 

Se modofica el Ingress con el secret creado, se adiciona a la etiqueta host una etiue ta secretName para relacionar la llave creada:
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  namespace: default
spec:
  
-tls:
  - hosts:
    - ad.rodriguez.student.lasalle.com
    secretName: nginx-certs-keys
  rules:
   - host: ad.rodriguez.student.lasalle.com
     http: 
      paths:
        - path: "/"
          pathType: Prefix
          backend:
           service:
            name: service-nginx
            port:
              number: 80
Validación: 
curl -k https://ad.rodriguez.student.lasalle.com
 
