# dio-azure-cloud-native-lab02
Desafio Microsoft Azure Cloud Native - Criando um Blog com Container Apps
---

# Deploy de Blog Estático com Azure Container Apps (Plano Gratuito)

Este projeto demonstra como criar um blog estático básico com HTML, CSS e JS, empacotá-lo com Docker e publicar usando **Azure Container Apps** com recursos gratuitos.

## Estrutura do Projeto

📁 blog\
├── `Dockerfile`: Empacotamento da aplicação com Nginx.\
├── `index.html`: Página inicial do blog com conteúdo do curso.\
├── `create-post.html`: Interface para criar novos posts.\
├── `post-detail.html`: Página de visualização e comentários dos posts.\
├── `deployment.yaml`: Manifesto para o Kubernetes Deployment.\
├── `service.yaml`: Serviço para expor a aplicação na internet.\
└── `comandos.txt`: Lista com todos os comandos CLI utilizados.

---

## Pré-requisitos

* Conta na [Azure](https://azure.microsoft.com/)
* Docker instalado
* [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) instalada
* Repositório GitHub criado (opcional)

---

## Etapa 1: Criar e testar localmente o Docker

### 1. Criar o `Dockerfile`

```dockerfile
# Utiliza a imagem oficial do Nginx
FROM nginx:alpine

# Copia todos os arquivos HTML (e outros estáticos, como CSS/JS) para o diretório público do Nginx
COPY . /usr/share/nginx/html
```

> Você pode adaptar para incluir outros arquivos como `create-post.html` e `post-detail.html`.

### 2. Build e teste local

```bash
docker build -t landingpage/curso-ia:latest .
docker run -d -p 80:80 landingpage/curso-ia:latest
```

Acesse via [http://localhost](http://localhost) e confira o funcionamento.

---

## Etapa 2: Preparar Recursos no Azure

### 3. Login na Azure

```bash
az login
```

### 4. Criar Resource Group

```bash
az group create --name rg-blog --location eastus
```

### 5. Criar Azure Container Registry (ACR)

```bash
az acr create --resource-group rg-blog --name acrhsouzalab02 --sku Basic --admin-enabled true
```

### 6. Login no ACR

```bash
az acr login --name acrhsouzalab02
```

---

## Etapa 3: Enviar a imagem para o ACR

```bash
docker tag landingpage/curso-ia:latest acrhsouzalab02.azurecr.io/landingpage/curso-ia:latest
docker push acrhsouzalab02.azurecr.io/landingpage/curso-ia:latest
```

---

## Etapa 4: Criar e publicar Container App

Você pode optar por:

### **(Recomendado)** Usar Azure Container Apps (Plano Gratuito)

```bash
az containerapp env create \
  --name blog-env \
  --resource-group rg-blog \
  --location eastus
```

```bash
az containerapp create \
  --name blog-container \
  --resource-group rg-blog \
  --environment blog-env \
  --image acrhsouzalab02.azurecr.io/landingpage/curso-ia:latest \
  --target-port 80 \
  --ingress external \
  --registry-login-server acrhsouzalab02.azurecr.io \
  --registry-username <USERNAME> \
  --registry-password <PASSWORD>
```

Recupere credenciais do ACR:

```bash
az acr credential show --name acrhsouzalab02
```

---

## (Opcional) Etapa 5: Usar AKS com YAMLs

Se preferir usar AKS (Azure Kubernetes Service), siga os passos abaixo:

### 1. Aplicar `deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: curso-ia-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: curso-ia
  template:
    metadata:
      labels:
        app: curso-ia
    spec:
      containers:
      - name: curso-ia
        image: acrhsouzalab02.azurecr.io/landingpage/curso-ia:latest
        ports:
        - containerPort: 80
```

### 2. Aplicar `service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: curso-ia-service
spec:
  selector:
    app: curso-ia
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```

### 3. Comandos AKS

```bash
az aks get-credentials --resource-group <SEU_RESOURCE_GROUP> --name <SEU_CLUSTER_AKS>

kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

kubectl get svc curso-ia-service
```

---

## Recursos Gratuitos Utilizados

* **Azure Container Registry (ACR)** — SKU Basic (grátis por 1 mês ou cota limitada)
* **Azure Container Apps** — Plano Gratuito disponível
* **Azure CLI e Docker Desktop**

---

## Links Úteis

* [Documentação Azure Container Apps](https://learn.microsoft.com/azure/container-apps/)
* [Documentação Docker](https://docs.docker.com/)
* [Portal Azure](https://portal.azure.com)


