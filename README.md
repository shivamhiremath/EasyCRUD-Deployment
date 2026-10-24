🚀 EasyCRUD – Cloud-Native Student Registration System on AWS (EKS + RDS + Docker)

EasyCRUD is a full-stack CRUD web application deployed using modern DevOps and cloud-native technologies.

This project demonstrates containerization with Docker, Kubernetes orchestration on AWS EKS, managed database integration with Amazon RDS, and a scalable microservices architecture.

🚀 Deployed Using

Docker

AWS EKS (Kubernetes)

AWS RDS (MariaDB)

Kubernetes Services

Load Balancer

🛠️ Technologies Used

Frontend: React

Backend: Spring Boot

Database: Amazon RDS (MariaDB)

Containers: Docker & Docker Hub

Orchestration: Kubernetes (AWS EKS)

Cloud Platform: AWS EC2, EKS, RDS

📂 This Repository Contains

✔ Database setup on AWS RDS

✔ Backend Dockerization and deployment

✔ Frontend Dockerization and deployment

✔ Kubernetes pod and service configurations

✔ Complete production-ready DevOps workflow

🟢 1: Database Setup (Amazon RDS – MariaDB)

📘 Database Setup Documentation: Provision a MariaDB database using Amazon RDS. Ensure your Security Groups are configured to allow inbound traffic on port 3306 from your EKS cluster.

🟢 2: Create AWS EKS Cluster

📘 AWS EKS Setup: Provision an Amazon EKS Cluster with managed node groups. Ensure kubectl is configured on your local machine or bastion node to communicate with the cluster.

🔵 3: Backend Deployment Steps (Docker + DockerHub + EKS)

Step 1: Install Docker

Install Docker on your EC2 instance and start the Docker service.

sudo apt install docker.io -y
sudo systemctl start docker


Step 2: Clone Project Repository

Clone your EasyCRUD project repository and navigate to the backend directory.

git clone https://github.com/<your-username>/EasyCRUD.git
cd EasyCRUD/backend/


Step 3: Configure Backend Application

Copy the application.properties file to the backend root directory and edit it.

cp src/main/resources/application.properties .
nano application.properties


Update the following values:

server.port=8080
spring.datasource.url=jdbc:mariadb://<RDS-EndPoint>:3306/student_db
spring.datasource.username=<username>
spring.datasource.password=<password>
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true


Step 4: Create Dockerfile for Backend

nano Dockerfile


Add the following content:

FROM maven:3.8.5-openjdk-17
COPY . /opt/
WORKDIR /opt
RUN rm -rf src/main/resources/application.properties
RUN cp -r application.properties src/main/resources/
RUN mvn clean package
WORKDIR target/
CMD ["java","-jar","student-registration-backend-0.0.1-SNAPSHOT.jar"]


Step 5: Build Backend Docker Image

docker build -t backend:v1 .
docker images


Step 6: Run Backend Container Locally (Testing)

docker run -d -p 8080:8080 backend:v1
docker ps


✅ Verify backend in browser: http://<BACKEND_EC2_PUBLIC_IP>:8080

Step 7: Push Backend Image to Docker Hub

docker tag backend:v1 <your-dockerhub-username>/backend:v1
docker login -u <username>
docker push <your-dockerhub-username>/backend:v1


✅ Backend image is now available for deployment on EKS.

✅ Deploy Backend on EKS Kubernetes Cluster

Step 8: Login to EKS Master / Bastion Node

Verify EKS cluster connectivity:

kubectl get nodes


Step 9: Create Backend Pod YAML

nano backend-deployment.yml


Note: Update the image name to match your Docker Hub repository.

apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-dep
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      name: backend-pod
      labels:
        app: backend
    spec:
      containers:
        - name: backend
          image: <your-dockerhub-username>/backend:v1
          ports:
            - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: backend-svc
spec:
  type: LoadBalancer
  selector:
    app: backend
  ports:
    - name: lb
      protocol: TCP
      port: 8080
      targetPort: 8080


Step 10: Deploy Backend Pod and Service

kubectl apply -f backend-deployment.yml
kubectl get pods -o wide
kubectl get svc


Step 11: Verify Backend on EKS

Retrieve the LoadBalancer external endpoint:

kubectl get svc backend-svc


Open in your browser: http://<LOADBALANCER_EXTERNAL_IP>:8080
🎉 Backend pod setup on EKS is complete!

🔵 4: Frontend Deployment Steps (Docker + DockerHub + EKS)

Step 1: Navigate to Frontend Directory

cd ../frontend/
ls -a


Step 2: Edit .env File (Add Backend LoadBalancer URL)

nano .env


Add the backend API endpoint:

# Backend API LoadBalancer URL
REACT_APP_BACKEND_URL=http://<BACKEND_LOADBALANCER_DNS>:8080


Step 3: Create Frontend Dockerfile

nano Dockerfile


FROM node:25-alpine
COPY . /opt/
WORKDIR /opt
RUN npm install
RUN npm run build
RUN apk update && apk add apache2
RUN cp -rf dist/* /var/www/localhost/htdocs/
EXPOSE 80
CMD ["httpd","-D","FOREGROUND"]


Step 4: Build Frontend Docker Image

docker build -t <your-dockerhub-username>/frontend:v1 .
docker images


Step 5: Run Frontend Container Locally (Testing)

docker run -d -p 80:80 <your-dockerhub-username>/frontend:v1
docker ps


✅ Verify frontend in browser: http://<FRONTEND_EC2_PUBLIC_IP>

Step 6: Push Frontend Image to Docker Hub

docker push <your-dockerhub-username>/frontend:v1


✅ Deploy Frontend on EKS (Kubernetes)

Step 7 & 8: Create Frontend Deployment and Service YAML

nano frontend-deployment.yml


apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-dep
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      name: frontend-pod
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend-pod
          image: <your-dockerhub-username>/frontend:v1
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
    - name: frontend-lb
      protocol: TCP
      port: 80
      targetPort: 80


Step 9: Deploy Frontend Pod and Service

kubectl apply -f frontend-deployment.yml
kubectl get pods -o wide
kubectl get svc


Step 10: Get LoadBalancer Endpoint and Verify

Retrieve the LoadBalancer external endpoint for the frontend service:

kubectl get svc frontend-svc


Copy the LoadBalancer DNS and open in your browser: http://<FRONTEND_LOADBALANCER_DNS>
🎉 Your EasyCRUD application is now live on Kubernetes (AWS EKS)! ✅

📖 Project Overview

EasyCRUD is a cloud-native student registration system built using a modern DevOps workflow. The application is fully containerized with Docker and deployed on AWS using Kubernetes (EKS), with a managed MariaDB database hosted on Amazon RDS.

This project demonstrates real-world practices such as:

Kubernetes orchestration

Cloud infrastructure on AWS

CI-style container workflows

LoadBalancer-based service exposure

🧪 Verification Checklist

[x] Backend reachable via EKS LoadBalancer

[x] Frontend reachable via EKS LoadBalancer

[x] Database connected via RDS endpoint

[x] Pods in Running state

[x] Services exposed properly

🛠️ Common Troubleshooting

Pod not running?

kubectl describe pod <pod-name>
kubectl logs <pod-name>


Service not getting EXTERNAL-IP?
Wait for 2–3 minutes (AWS LoadBalancer provisioning takes time), or run:

kubectl get svc -w


Backend not connecting to Amazon RDS?
Check the following:
✔ Security Group inbound rules allow database access
✔ Correct RDS endpoint in the application.properties file
