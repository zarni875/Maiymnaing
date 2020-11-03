apiVersion: v1 #version of the API to use
kind: Pod #What kind of object we're deploying
metadata: #information about our object we're deploying
  name: zn-pod
spec: #specifications for our object
  containers:
  - name: zn-container #the name of the container within the pod
    image: yenaing/php72 #which container image should be pulled
    ports:
    - containerPort: 80 #the port of the container within the pod

///pod

======================================================

apiVersion: apps/v-1 #version of the API to use
kind: Deployment #What kind of object we're deploying
metadata: #information about our object we're deploying
  name: znphp-deployment #Name of the deployment
  labels: #A tag on the deployments created
    app: znphp
spec: #specifications for our object
  replicas: 2 #The number of pods that should always be running
  selector: #which pods the replica set should be responsible for
    matchLabels:
      app: znphp #any pods with labels matching this I'm responsible for.
  template: #The pod template that gets deployed
    metadata:
      labels: #A tag on the replica sets created
        app: znphp
    spec:
      containers:
      - name: znphp-container #the name of the container within the pod
        image: yenaing/php72 #which container image should be pulled
 ports:
        - containerPort: 80 #the port of the container within the pod
        volumeMounts:
        - name: hostvol
          mountPath: /var/www/html/
      volumes:
        - name: hostvol
          persistentVolumeClaim:
            # PVC name you created
            claimName: znphp-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: znphp-svc
  labels:
    run: znphp-svc
spec:
  type: LoadBalancer
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: znphp

znphpweb

====================================================================

apiVersion: v1
kind: PersistentVolume
metadata:
  # any PV name
  name: znphp-pv
spec:
  storageClassName: znphp07
  capacity:
    # storage size
    storage: 10Gi
  accessModes:
    # ReadWriteMany(RW from multi nodes), ReadWriteOnce(RW from a node), ReadOn$
    - ReadWriteMany
  persistentVolumeReclaimPolicy:
    # retain even if pods terminate
    Retain
  nfs:
    # NFS server's definition
    path: /mnt/cloudzarni/data
    server: 192.168.14.10
    readOnly: false

///phpweb-pv
=========================================
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  # any PVC name
  name: znphp-pvc
spec:
  storageClassName: znphp07
  accessModes:
  # ReadWriteMany(RW from multi nodes), ReadWriteOnce(RW from a node), ReadOnly$
    - ReadWriteMany
  resources:
    requests:
       # storage size to use
      storage: 5Gi

///phpweb-pvc

================================================
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: znmysql8
spec:
  selector:
    matchLabels:
      app: znmysql8
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: znmysql8
    spec:
      containers:
      - image: yenaing/mysql8
        name: znmysql
        env:
 - name: MYSQL_ROOT_PASSWORD
          value: zar
        - name: MYSQL_DATABASE
          value: my_db
        - name: MYSQL_USER
          value: db_user
        - name: MYSQL_PASSWORD
          value: .mypwd
        args: ["--default-authentication-plugin=mysql_native_password"]
        ports:
        - containerPort: 3306
          name: mysql8

////mysqlde
==============================

apiVersion: v1
kind: PersistentVolume
metadata:
  # any PV name
  name: znmysql8-pv
spec:
  storageClassName: zn21
  capacity:
    # storage size
    storage: 10Gi
  accessModes:
    # ReadWriteMany(RW from multi nodes), ReadWriteOnce(RW from a node), ReadOn$
    - ReadWriteMany
  persistentVolumeReclaimPolicy:
    # retain even if pods terminate
    Retain
  nfs:
    # NFS server's definition
    path: /mnt/cloudzarni/data
    server: 192.168.14.10
    readOnly: false

msqlpv///
==========================

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  # any PVC name
  name: znmysql-pv
spec:
  storageClassName: zn21
  accessModes:
  # ReadWriteMany(RW from multi nodes), ReadWriteOnce(RW from a node), ReadOnly$
    - ReadWriteMany
  resources:
    requests:
       # storage size to use
      storage: 5Gi

mysqlpvc//
============================================
apiVersion: v1
kind: Service
metadata:
  name: mysql8-service
  labels:
    app: znmysql8
spec:
  type: NodePort
  ports:
  - port: 3306
    protocol: TCP
  selector:
    app: znmysql8


servicesql///
========================

apiVersion: v1
kind: Service
metadata:
  name: znphp-svc
  labels:
    run: znphp-svc
spec:
  type: LoadBalancer
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: znphp


php service///
===================================
