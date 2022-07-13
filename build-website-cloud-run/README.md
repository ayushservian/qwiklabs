# Build a Website on Google Cloud: Challenge Lab

Quest: [#115](https://partner.cloudskillsboost.google/focuses/13346?parent=catalog)

## Solution for Task 1: Download the monolith code and build your container

on cloud shell git clone
./setup.sh
nvm install --lts

Build and Push monolith app (in monolith folder) to GCR - dockerfile in ~/monolith-to-microservices/monolith


## Solution for Task 2: Create a kubernetes cluster and deploy the application

create GKE cluster `Cluster Name` for us-central1-a zone, node count 3
deploy app on GCR to `Cluster Name` - app runs on 8080, but exposed to port 80 -> deployment must be named `Monolith Identifier`
    Cluster name: Cluster Name
    Container name: Monolith Identifier
    Container version: 1.0.0
    Application port: 8080
    Externally accessible port: 80



## Solution for Task 3: Create a containerized version of your Microservices

Build and push to GCR
1. Orders service
    Service root folder: ~/monolith-to-microservices/microservices/src/orders
    GCR Repo: gcr.io/${GOOGLE_CLOUD_PROJECT}
    Image name: Orders Identifier
    Image version: 1.0.0
2. Products service
    Service root folder: ~/monolith-to-microservices/microservices/src/products
    GCR Repo: gcr.io/${GOOGLE_CLOUD_PROJECT}
    Image name: Products Identifier
    Image version: 1.0.0

Hint: Make sure that you submit a build named Orders Identifier with a version of "1.0.0", AND a build named Products Identifier with a version of "1.0.0".

## Solution for Task 4: Deploy the new microservices

1. Orders build
    Cluster name: Cluster Name
    Container name: Orders Identifier
    Container version: 1.0.0
    Application port: 8081
    Externally accessible port: 80

    NOTE IP address of the exposed service
2. Products build
    Cluster name: Cluster Name
    Container name: Products Identifier
    Container version: 1.0.0
    Application port: 8082
    Externally accessible port: 80

    NOTE IP address of the exposed service

http://ORDERS_EXTERNAL_IP/api/orders
http://PRODUCTS_EXTERNAL_IP/api/products

## Solution for Task 5: Configure the Frontend microservice

Update IP address of new microservice 
cd ~/monolith-to-microservices/react-app
nano .env

| Before                                                    | After                                                            |
| ---                                                       | ----                                                             |
| REACT_APP_ORDERS_URL=http://localhost:8081/api/orders     | REACT_APP_ORDERS_URL=http://<ORDERS_IP_ADDRESS>/api/orders       |
| REACT_APP_PRODUCTS_URL=http://localhost:8082/api/products | REACT_APP_PRODUCTS_URL=http://<PRODUCTS_IP_ADDRESS>/api/products |
 
Press CTRL+O, press ENTER, then CTRL+X to save the file in the nano editor.

npm run build

## Solution for Task 6: Create a containerized version of the Frontend microservice

build and push to GCR ->
    Service root folder: ~/monolith-to-microservices/microservices/src/frontend
    GCR Repo: gcr.io/${GOOGLE_CLOUD_PROJECT}
    Image name: Frontend Identifier
    Image version: 1.0.0

## Solution for Task 7: Deploy the Frontend microservice

Deploy frontend
    Cluster name: Cluster Name
    Container name: Frontend Identifier
    Container version: 1.0.0
    Application port: 8080
    Externally accessible port: 80