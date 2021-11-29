2- [StatefulSet] Crear un StatefulSet con 3 instancias de MongoDB (ejemplo visto en clase) Se pide:
 • Habilitar el clúster de MongoDB 
Se crea el servicio headless para gestionar las Ip de los pod creados por el recurso:
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
  labels:
    name: mongo
spec:
  ports:
  - port: 27017
    targetPort: 27017
  clusterIP: None
  selector:
    role: mongo

Se crea el recurso ReplicaSet con un StorageClass asociado:
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongod
spec:
  serviceName: mongodb-service
  replicas: 3
  selector:
    matchLabels:
      role: mongo
  template:
    metadata:
      labels:
        role: mongo
        environment: test
        replicaset: MainRepSet
    spec:
      terminationGracePeriodSeconds: 10
      containers:
        - name: mongod-container
          image: mongo
          command:
            - "mongod"
            - "--bind_ip"
            - "0.0.0.0"
            - "--replSet"
            - "MainRepSet"
          resources:
            requests:
              cpu: 0.2
              memory: 200Mi
          ports:
            - containerPort: 27017
          volumeMounts:
            - name: mongodb-persistent-storage-claim
              mountPath: /data/db
  volumeClaimTemplates:
  - metadata:
      name: mongodb-persistent-storage-claim
      annotations:
        volume.beta.kubernetes.io/storage-class: "standard"   -- storageClass
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi

Se empiezan a crear los contenedores por orden
 
Finalmente se crean las 3 replicas
 
Se inicializa el cluster desde uno de los pod
kubectl exec -it mongod-0 mongo
Con el siguiente comando se configuran los pods para trabajar de manera conjunta
rs.initiate({_id: "MainRepSet", version: 1, members: [ { _id: 0, host: "mongod-0.mongodb-service.default.svc.cluster.local:27017" }, 
													   { _id: 1, host: "mongod-1.mongodb-service.default.svc.cluster.local:27017" },
													   { _id: 2, host: "mongod-2.mongodb-service.default.svc.cluster.local:27017" }]});
 
Stateful creados
 

Volúmenes de persistencia creados
 

• Realizar una operación en una de las instancias a nivel de configuración y verificar que el cambio se ha aplicado al resto de instancias 
Se actualiza la imagen de mongo de mongo a mongo:latest en el nodo mongod-0
 

 
Se verifica la version de mongo a mongo:latest:
 
 
Se realiza el replicaset dentro del cluster de uno de los nodos de mongoDB con el siguiente comando:
rs.initiate({_id: "MainRepSet", version: 1, members: [ { _id: 0, host: "mongod-0.mongodb-service.default.svc.cluster.local:27017" }, { _id: 1, host: "mongod-1.mongodb-service.default.svc.cluster.local:27017" },{ _id: 2, host: "mongod-2.mongodb-service.default.svc.cluster.local:27017" }]});
 

 
Se valida en uno de los nodos el cambio realizado:
 

• Diferencias que existiría si el montaje se hubiera realizado con el objeto de ReplicaSet
Los nombres de los pod tendrían un hash en vez de un estado creado de manera escalonada
 
Con el StateFulSet el estado se persiste por tanto se debe utilizar la instrucción volumeClaimTemplates la cual afirma en volúmenes persistentes para garantizar que puedan mantener el estado en los reinicios de los componentes.
