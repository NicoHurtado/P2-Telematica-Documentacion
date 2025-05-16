# BookStore Monolithic App - Nicolas Hurtado - Jacobo Restrepo

## Descripción General
El objetivo principal de este proyecto fue **ejecutar la versión monolítica de BookStore en un clúster de Kubernetes**, como parte del **Objetivo 3**, Luego de realizar los objetivos **1** y **2** utilizando AWS EC2 y MicroK8s, con dominio, SSL y base de datos externa en RDS MySQL.

---

##  Tecnologías Utilizadas

- **Flask**: framework backend de la app monolítica  
- **Docker**: para empaquetar la aplicación  
- **MicroK8s (Kubernetes)**: para orquestar el despliegue de la app  
- **AWS EC2**: instancia virtual Ubuntu
- **AWS RDS (Aurora MySQL)**: base de datos externa administrada  
- **MicroK8s local registry**: almacenamiento de imagen de la app  

---

## Objetivos Alcanzados

### Objetivo 1
- Despliegue inicial de la aplicación monolítica en Docker
- Proxy inverso con NGINX y dominio `projectnj.space`
- Certificado SSL vía Let's Encrypt

### Objetivo 2
- Escaamiento horizontal con **Auto Scaling Group** y **ALB** en AWS
- Migración de base de datos a **Amazon RDS Aurora MySQL**

### Objetivo 3 (fase 1)
- Despliegue de la **aplicación monolítica** en Kubernetes (MicroK8s)
- Uso de **registry local** (`localhost:32000`) para publicar imagen Docker
- Conexión de la app con base de datos externa en AWS RDS
- Exposición del servicio vía `NodePort` sin dominio (puerto `30080`)

---

## Estructura del Proyecto
![image](https://github.com/user-attachments/assets/48afd489-e18a-41e9-97a6-2436f927ff69)

## Deployment.yml 
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: bookstore-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: bookstore
  template:
    metadata:
      labels:
        app: bookstore
    spec:
      containers:
        - name: bookstore
          image: localhost:32000/bookstore:latest
          ports:
            - containerPort: 5000
          env:
            - name: DB_HOST
              value: "database-1.cx0mogqekh4h.us-east-1.rds.amazonaws.com"
            - name: DB_PORT
              value: "3306"
            - name: DB_USER
              value: "admin"
            - name: DB_PASSWORD
              value: "proyectotelematica"
            - name: DB_NAME
              value: "bookstore"
            - name: SECRET_KEY
              value: "291e2e196899f7c7e6e086b231b07ca356a6d5e486a00172226f3da120339950"
```

## service.yml
```yml
apiVersion: v1
kind: Service
metadata:
  name: bookstore-service
spec:
  type: NodePort
  selector:
    app: bookstore
  ports:
    - port: 80
      targetPort: 5000
      nodePort: 30080

```
### Imagen Docker
- Construida localmente con `Dockerfile`
- Tagueada como: `localhost:32000/bookstore:latest`
- Push realizado a **MicroK8s local registry** (`microk8s enable registry`)

### Base de Datos RDS
- **Endpoint**: `database-1.cx0mogqekh4h.us-east-1.rds.amazonaws.com`
- **Usuario**: `admin`
- **Contraseña**: `proyectotelematica`
- **Nombre DB**: `bookstore`

![image](https://github.com/user-attachments/assets/64440e0e-42c6-482c-9448-5c261f7b714b)


