# Momo Store aka Пельменная №2 

URL:    momo-store.avolokhov.pro


<img width="900" alt="image" src="https://user-images.githubusercontent.com/9394918/167876466-2c530828-d658-4efe-9064-825626cc6db5.png">

Git flow: Trunk-Based Development

#   1 Локальное развертывание приложения (тестировалось сразу в Docker)

## Frontend

Node v16.9.1 (on runner works fine)
###   build frontend
```
npm install
npm npm run build
```
### Dockerfile
```
FROM nginx
ARG VERSION=${VERSION}
COPY dist/public_html /usr/share/nginx/html/
EXPOSE 80
```
# Build docker image
docker build -t frontend_image .
# Run container
docker run -d -it --network momo --expose 80 -p 80:80 --name frontend frontend_image


## Backend

Golang 1.17 (:latest works fine too)
###   build api
go build -o app ./...

#   Dockerfile
```
FROM gcr.io/distroless/base-debian10
ARG VERSION=${VERSION}
WORKDIR /
COPY app/api /api
EXPOSE 8081
USER nonroot:nonroot
ENTRYPOINT ["/api"]
```
# Build docker image
docker build -t backend_image .
# Run container
docker run -d -it --network momo --expose 8081 -p 8081:8081 --name backend backend_image

#   2   Репозитории  (См. diploma.pptx в репозитории приложения)

#   2.1  Структура репозитория приложения
```
git@gitlab.praktikum-services.ru:antinitrino/momo-store-antinitrino.git
https://gitlab.praktikum-services.ru/antinitrino/momo-store-antinitrino
.
├── backend
├── diploma.odp
├── frontend
├── .git
├── .gitignore
├── .gitlab-ci.yml
└── README.md
```
#   2.2 Структура репозитория инфраструктуры
```
git@gitlab.praktikum-services.ru:antinitrino/infra-momo-store-antinitrino.git
https://gitlab.praktikum-services.ru/antinitrino/infra-momo-store-antinitrino

.
├── Charts                      -   Чарты Grafana,Prometheus,Momo-store
│   ├── grafana
│   ├── momo-store-chart
│   └── prometheus
├── .git
├── .gitignore
├── .gitlab-ci.yml
├── KUBER_ARGO                  -   K8S манивесты для ArgoCD
│   ├── app_momo-store.yml
│   ├── cluster.yml
│   ├── ingress.yml
│   ├── policy.yaml
│   └── user.yml
├── README.md
└── terraform                    -  terraform манифесты для деплоя кластера k8s и сервисной ВМ Devops
    ├── devops-node
    └── k8s-cluster
```
#   2.3 Развертывание инфраструктуры
#   2.3.1  K8S
```
terraform init -backend-config=backend.conf
terraform apply -var-file secret.tfvars
```
#   2.3.2   Сервисная нода "DEVOPS"
```
terraform init -backend-config=backend.conf
terraform apply -var-file secret.tfvars
scp docker-compose.yaml devops.avolokhov.pro:/home/vav
```
#   2.3.3   ArgoCD
```
kubectl create namespace argocd
kubectl apply -n argocd -f argocd.yml
kubectl apply -n argocd -f ingress.yml
kubectl apply -n argocd -f user.yml
```
#   2.3.4   Grafana
helm upgrade --install grafana ./grafana --atomic
#   2.3.5   Loki-stack
```
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm upgrade --install loki grafana/loki-stack
```
#   2.3.6   Prometheus
```
helm upgrade --install prometheus ./prometheus --atomic
kubectl apply -f admin-user.yml (Чтобы видел экспортеров)
```
#   2.3.7   Deploy momo-store
```
В GUI ArgoCD добавить приложение через YAML

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: momo-store
spec:
  destination:
    name: ''
    namespace: default
    server: 'https://kubernetes.default.svc'
  source:
    path: momo-store-chart
    repoURL: >-
      https://gitlab.praktikum-services.ru/antinitrino/infra-momo-store-antinitrino.git
    targetRevision: HEAD
  project: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```


#   3   Правила внесения изменений в репозиторий
Все изменения производятся в новой ветке, создаваемой из main, после чего делается Merge Request

#   4   Версионирование
```
Версионирование для backend и frontend производится через переменную в пайплайнах
    variables:
    VERSION: 1.0.${CI_PIPELINE_ID}
Через эту переменную версия по цепочке передается до деплоя.
В Conainer Registry заливается 2 образа с тэгами: :${VERSION} и :latest ( :${VERSION} -eq :latest )
При первом запуске приложения из ArgoCD берется образ :latest. При последующих деплоях :${VERSION}
gitlab.praktikum-services.ru:5050/antinitrino/momo-store-antinitrino/momo-frontend:1.0.161068
gitlab.praktikum-services.ru:5050/antinitrino/momo-store-antinitrino/momo-backend:1.0.161068
```

