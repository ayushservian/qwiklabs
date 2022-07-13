# Build a Website on Google Cloud: Challenge Lab

Quest: [#115 - GSP139](https://partner.cloudskillsboost.google/focuses/13346?parent=catalog)

## Solution for Task 1: Download the monolith code and build your container

```sh
git clone https://github.com/googlecodelabs/monolith-to-microservices.git
cd ~/monolith-to-microservices
./setup.sh
nvm install --lts
```

Build and Push monolith app (in monolith folder) to GCR - dockerfile in ~/monolith-to-microservices/monolith
To go through gcloud build and push to GCR follow below process:
```
cd ~/monolith-to-microservices/monolith
gcloud services enable cloudbuild.googleapis.com
gcloud services enable container.googleapis.com
gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/fancy-monolith-149:1.0.0 .
```

## Solution for Task 2: Create a kubernetes cluster and deploy the application

create GKE cluster `fancy-production-810` for us-central1-a zone, node count 3

Create cluster
```
gcloud config set compute/zone us-central1-a
gcloud container clusters create fancy-production-810 --num-nodes 3
gcloud compute instances list
```

deploy app on GCR to `fancy-production-810` - app runs on 8080, but exposed to port 80 -> deployment must be named `fancy-monolith-149`
    Cluster name: fancy-production-810
    Container name: fancy-monolith-149
    Container version: 1.0.0
    Application port: 8080
    Externally accessible port: 80
```
kubectl create deployment fancy-monolith-149 --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/fancy-monolith-149:1.0.0
kubectl expose deployment fancy-monolith-149 --type=LoadBalancer --port 80 --target-port 8080
```

## Solution for Task 3: Create a containerized version of your Microservices

Build and push to GCR
1. Orders service
    Service root folder: ~/monolith-to-microservices/microservices/src/orders
    GCR Repo: gcr.io/${GOOGLE_CLOUD_PROJECT}
    Image name: fancy-orders-663
    Image version: 1.0.0
2. Products service
    Service root folder: ~/monolith-to-microservices/microservices/src/products
    GCR Repo: gcr.io/${GOOGLE_CLOUD_PROJECT}
    Image name: fancy-products-124
    Image version: 1.0.0

Hint: Make sure that you submit a build named `fancy-orders-663` with a version of "1.0.0", AND a build named `fancy-products-124` with a version of "1.0.0".

```
cd ~/monolith-to-microservices/microservices/src/orders
gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/fancy-orders-663:1.0.0 .
cd ~/monolith-to-microservices/microservices/src/products
gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/fancy-products-124:1.0.0 .
```


## Solution for Task 4: Deploy the new microservices

1. Orders build
    Cluster name: fancy-production-810
    Container name: fancy-orders-663
    Container version: 1.0.0
    Application port: 8081
    Externally accessible port: 80

    NOTE IP address of the exposed service
2. Products build
    Cluster name: fancy-production-810
    Container name: fancy-products-124
    Container version: 1.0.0
    Application port: 8082
    Externally accessible port: 80

    NOTE IP address of the exposed service

http://ORDERS_EXTERNAL_IP/api/orders
http://PRODUCTS_EXTERNAL_IP/api/products


IF THIS IS GCLOUD RUN
```
kubectl create deployment fancy-orders-663 --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/fancy-orders-663:1.0.0
kubectl expose deployment fancy-orders-663 --type=LoadBalancer --port 80 --target-port 8081

kubectl create deployment fancy-products-124 --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/fancy-products-124:1.0.0
kubectl expose deployment fancy-products-124 --type=LoadBalancer --port 80 --target-port 8082

kubectl get service fancy-orders-663
kubectl get service fancy-products-124
```
## Solution for Task 5: Configure the Frontend microservice

Update IP address of new microservice 
```
cd ~/monolith-to-microservices/react-app
nano .env
```

| Before                                                    | After                                                            |
| ---                                                       | ----                                                             |
| REACT_APP_ORDERS_URL=http://localhost:8081/api/orders     | REACT_APP_ORDERS_URL=http://<ORDERS_IP_ADDRESS>/api/orders       |  34.67.117.226
| REACT_APP_PRODUCTS_URL=http://localhost:8082/api/products | REACT_APP_PRODUCTS_URL=http://<PRODUCTS_IP_ADDRESS>/api/products |  34.134.125.134
 
Press CTRL+O, press ENTER, then CTRL+X to save the file in the nano editor.
```
npm run build
```
## Solution for Task 6: Create a containerized version of the Frontend microservice

build and push to GCR ->
    Service root folder: ~/monolith-to-microservices/microservices/src/frontend
    GCR Repo: gcr.io/${GOOGLE_CLOUD_PROJECT}
    Image name: Frontend Identifier
    Image version: 1.0.0

```
cd ~/monolith-to-microservices/microservices/src/frontend
gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/fancy-frontend-510:1.0.0 .
```
## Solution for Task 7: Deploy the Frontend microservice

Deploy frontend
    Cluster name: fancy-production-810
    Container name: fancy-frontend-510
    Container version: 1.0.0
    Application port: 8080
    Externally accessible port: 80
```
kubectl create deployment fancy-frontend-510 --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/fancy-frontend-510:1.0.0
kubectl expose deployment fancy-frontend-510 --type=LoadBalancer --port 80 --target-port 8080
```