# AKS Storage - Azure Disks

## Step-01: Introduction
- We are going to use Azure AKS provisioned storage class as part of this section

## Step-02: Use AKS Provisioned Azure Disks
- Copy all templates from previous section
- Remove Storage Class Manifest
- **Question-1:** Why do we need to remove storage class Manifests?
- Azure AKS provisions two types of storage classes well in advance during the cluster creation process
  - managed-premium
  - default-
- We can leverage Azure AKS provisioned disk storage classes instead of what we created manually.
- **Question-2:** If that is the case why did we use custom storate class in previous section?
- That is for us to learn the `kind: StorageClass` concept.  

## Step-03: Review PVC Manifest 01-persistent-volume-claim.yml
- Primarily we are going to focus on `storageClassName: managed-premium` which tells us that we are going to use Azure AKS provisioned Azure Disks storage class.
```yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: azure-managed-disk-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: managed-premium 
  resources:
    requests:
      storage: 5Gi  
```

## Step-04: Deploy and Test
```
# Create MySQL Database
kubectl apply -f kube-manifests/

# List Storage Classes
kubectl get sc

# List PVC
kubectl get pvc 

# List PV
kubectl get pv

# List pods
kubectl get pods 
```

## Step-05: Connect to MySQL Database
```
# Connect to MYSQL Database
kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h mysql -pdbpassword11

# Verify usermgmt schema got created which we provided in ConfigMap
mysql> show schemas;
```

## Step-06: Clean-Up
```
# Delete all manifests
kubectl delete -f kube-manifests/
```
###############################
use below YAML code and explain each line as a teacher in simple way and I am new to this code
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec: 
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: Recreate 
  template: 
    metadata: 
      labels: 
        app: mysql
    spec: 
      containers:
        - name: mysql
          image: mysql:5.6
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: dbpassword11
          ports:
            - containerPort: 3306
              name: mysql    
          volumeMounts:
            - name: mysql-persistent-storage
              mountPath: /var/lib/mysql    
            - name: usermanagement-dbcreation-script
              mountPath: /docker-entrypoint-initdb.d #https://hub.docker.com/_/mysql Refer Initializing a fresh instance                                            
      volumes: 
        - name: mysql-persistent-storage
          persistentVolumeClaim:
            claimName: azure-managed-disk-pvc
        - name: usermanagement-dbcreation-script
          configMap:
            name: usermanagement-dbcreation-script
Kubernetes YAML Explanation for Beginners
Let me explain this YAML file line by line in simple terms. This file defines a MySQL database deployment in Kubernetes.

Basic Structure
yaml
Copy
apiVersion: apps/v1
kind: Deployment
apiVersion: apps/v1 → Specifies which Kubernetes API version to use (here, it's for Deployments).

kind: Deployment → Defines that this YAML will create a Deployment, which manages Pods (containers).

Metadata Section
yaml
Copy
metadata:
  name: mysql
metadata → Contains information about the Deployment.

name: mysql → The name of this Deployment is mysql.

Spec Section (Deployment Configuration)
yaml
Copy
spec: 
  replicas: 1
replicas: 1 → Only 1 copy (Pod) of MySQL will run. If it crashes, Kubernetes restarts it.

yaml
Copy
  selector:
    matchLabels:
      app: mysql
selector → Helps the Deployment find which Pods it manages.

matchLabels: app: mysql → The Deployment will control Pods with the label app: mysql.

yaml
Copy
  strategy:
    type: Recreate 
strategy: Recreate → When updating, Kubernetes will first kill the old Pod before creating a new one (good for databases).

Pod Template (Inside Deployment)
yaml
Copy
  template: 
    metadata: 
      labels: 
        app: mysql
template → Defines how the Pod should look.

labels: app: mysql → The Pod will have this label (matches the selector above).

Container Definition
yaml
Copy
    spec: 
      containers:
        - name: mysql
          image: mysql:5.6
containers → List of containers in this Pod (here, just one).

name: mysql → The container's name.

image: mysql:5.6 → Uses the MySQL 5.6 Docker image from Docker Hub.

Environment Variables
yaml
Copy
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: dbpassword11
env → Sets environment variables inside the container.

MYSQL_ROOT_PASSWORD → Sets MySQL's root password to dbpassword11.

Port Configuration
yaml
Copy
          ports:
            - containerPort: 3306
              name: mysql
containerPort: 3306 → MySQL runs on port 3306 inside the container.

name: mysql → Names this port (useful for services).

Volume Mounts (Persistent Storage)
yaml
Copy
          volumeMounts:
            - name: mysql-persistent-storage
              mountPath: /var/lib/mysql    
            - name: usermanagement-dbcreation-script
              mountPath: /docker-entrypoint-initdb.d
volumeMounts → Attaches storage to the container.

mysql-persistent-storage → Stores MySQL data at /var/lib/mysql (so data isn't lost if the Pod restarts).

usermanagement-dbcreation-script → Mounts a configMap (SQL scripts) to /docker-entrypoint-initdb.d (auto-runs scripts when MySQL starts).

Volumes (Storage Definitions)
yaml
Copy
      volumes: 
        - name: mysql-persistent-storage
          persistentVolumeClaim:
            claimName: azure-managed-disk-pvc
        - name: usermanagement-dbcreation-script
          configMap:
            name: usermanagement-dbcreation-script
volumes → Defines storage sources.

persistentVolumeClaim: azure-managed-disk-pvc → Uses an Azure Disk for permanent storage.

configMap: usermanagement-dbcreation-script → Injects SQL scripts (like CREATE DATABASE) from a ConfigMap.

#########################################################
