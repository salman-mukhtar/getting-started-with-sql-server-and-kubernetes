# Getting-Started-with-SQL-Server-and-Kubernetes

For this exercise we are going to create following components.

- **Persisted Disk**

	* Inherently containers have no persisted storage. Obviously, this is a problem for database containers. We are going to define a persistent volume claim to map our storage account.

- **Deployment**

	* This refers to our container and volume. You will see this defined in the code below.

- **Service**

	* We will expose our deployment by using a service as NodePort

# Let’s Start

This assumes that you have minikube up and running. The first thing you’re going to do is build a secret to pass into your deployment, for your SA password.

```
kubectl create secret generic mssql --from-literal=SA_PASSWORD="YourPassword"
```

The next thing we are going to do is build you persistent volume claim.

```
apiVersion: v1 
kind: PersistentVolumeClaim 
metadata: 
  name: "pvc0001" 
spec: 
  accessModes: 
  - "ReadWriteOnce" 
  resources: 
    requests: 
      storage: "5Gi"
```

We will save this text in a file. For the purposes of this posts, we will call it 01-pvc.yaml. We will then run the kubectl apply -f 01-pvc.yaml command. You will see the message “persistentvolumeclaim “mssql-data-claim” created

PICTURE

Next we are going to build our deployment

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mssql-deployment
spec:
  selector:
    matchLabels:
      app: mssql
  replicas: 1
  template:
    metadata:
      labels:
        app: mssql
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: mssql
        image: mcr.microsoft.com/mssql/server:2017-latest
        ports:
        - containerPort: 1433
        env:
        - name: ACCEPT_EULA
          value: "Y"
        - name: SA_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mssql
              key: SA_PASSWORD
        volumeMounts:
        - name: mssqldb
          mountPath: /var/opt/mssql
      volumes:
      - name: mssqldb
        persistentVolumeClaim:
          claimName: pvc0001
```
we are defining our deployment, which specifies the container we’re going to use, which in this case it is the latest release of SQL Server 2017, and it picks up our predefined SA password. Finally, we are defining our volume mount and its path for where it will be mounted in the VM. Save this off to a file called 02-deployment.yaml. We will run the same kubectl apply -f deployment.yaml to deploy this. You will see deployment “mssql-deployment” created.

PICTURE 

Last but not the least we will expose our deployment as NodePort service. Save the following code as 03-service.yaml and execute it on terminal as kubectl apply -f 03-service.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: mssql-deployment
spec:
  selector:
    app: mssql
  ports:
    - protocol: TCP
      port: 1433
      targetPort: 1433
  type: NodePort

```

After the service is exposed run the following command.

```
minikube service mssql-deployment --url
```

The output will give you the IP address and port you can connect to.


**SQL Script**

```
create database Lab1
go
use Lab1
go
create table person
(
firsrname varchar(25),
lastname varchar(25)

)
insert into person values('salman','mukhtar')
insert into person values('kamran','azeem')
insert into person values('yasir','abbasi')
insert into person values('khalid','hameed')
insert into person values('shahid','hussain')
select * from person
```
