# Kubernetes HW

Hello world deployment/svc/config/ingress for full-stack TS/JS applications with oauth2 proxy

# Getting Started

#### You will need the following already installed:

- docker 
- node
- kubectl cli
- minikuibe (optional)
- hyperkit (optional)

#### Install dependencies

```bash
cd api && npm i 
```

```bash
cd client && npm ci 
```

#### Env Vars
Add the proper .env variables to a .env file (see .env.example)

- PROJECT=<whatever you want>
- REPO=<name of docker repo you will push your image to>
- CLIENT_ID=<client id for oauth2 provider>
- CLIENT_SECRET=<client secret for oauth2 provider>
- COOKIE_SECRET=<32 byte cookie>
- DB_PASSWORD=<whatever postgres password you choose>
- DB_NAME=<whatever postgres db name you choose>
- DB_USER=<whatever postgres username you choose>
- ENV=<dev, prod, etc>
- PROXY_PORT=<port for proxy (4180)>
- PROXY_HOST=<localhost or deployed host>
- UPSTREAM_SVC=<upstream for proxy redirect>
- UPSTREAM_PORT=<port of the upstream svc>


To generate a cookie secret: 

```bash
python3 -c 'import os,base64; print(base64.urlsafe_b64encode(os.urandom(32)).decode())'

```

To get the client_id and client_secret:
- [Github Provider](#github.com/settings/developers)
   - click "new oauth project"
   - set the name to whatever you want
   - set the project homepage to your url (
   - ie: local: `http://localhost:4180`
   - ie: cloud-provider: `http://<your endpoint>`
   - ie: minikube:  `http://<minikubeip>`
        - see notes on minikube below

# Running the Project

## Deploy To AZ

```bash
docker-compose up -d --build 
```

```bash
az login 
```

```bash
az aks get-credentials 
``` 

```bash
az acr login --name <acrName>
``` 

```bash
make docker-push-prod
```
```bash
kubectl create namespace <namespace>
```
```bash
kubectl config set-context --current --namespace=<insert-namespace-name-here>
```
```bash
kubectl create secret generic oauthproxy-secret \ --from-literal=client_id=<client_id> --from-literal=client-secret=<client_secret> --from-literal=cookie-secret=<cookie_secret>
``` 
```bash
kubectl create secret generic db-secret \ --from-literal=password=<db_password> 
``` 

```bash
kubectl apply -f k8s
``` 

## Locally

#### Minikube
- ensure minikube is installed
- otherwise 'brew install minikube'
- you will need to run minikube with hyperkit to expose the url

```bash
minikube start --driver=hyperkit
```
- you will need to enable the addons:
```bash
minikube addons enable ingress
```
```bash
minikube addons enable ingress-dns
```
***note:*** this steps only apply to intel chip macs. So far this has not worked out on an m1 chip. If you find a way to make it work, please let me know.         
- run `kubectl apply -f ./k8s` 
- run `minikube ip`
- copy the output to your provider set up as the homepage/redirect         

#### Docker

- Copy the env.example files and fill in the correct values. 
- Make sure that iun your make file you are pointing to the correct env file

`make run-local`

#### Skaffold
- ensure skaffold is installed
- otherwise brew install skaffold
- Ensure minikube is running with the hyperkit driver
- Ensure ingress and ingress-dns addons have been enabled in minikube 
- Ensure correct namespace and secrets see [Kubectl Commands](#kubectl_commands)
- Ensure you images have been built: 'make build-local' and pushed: "make docker-push" 
- running `skaffold dev` is intended for development, and your project should reload if you make changes to the src files in api/client

```bash
skaffold dev
```

#### Kubectl Commands
Create and set namespace:
- kubectl create namespace <namespace>
- kubectl config set-context --current --namespace=<insert-namespace-name-here>
Create secrets:
- kubectl create secret generic oauthproxy-secret \ --from-literal=client_id=<client_id> --from-literal=client-secret=<client_secret> --from-literal=cookie-secret=<cookie_secret>
- kubectl create secret generic db-secret \ --from-literal=password=<db_password> 