
# Aprendizado de Kubernetes para quem já conhece Docker

Este guia é voltado para quem já tem experiência com Docker e deseja dar o próximo passo aprendendo Kubernetes. Aqui você encontrará explicações claras, exemplos práticos e instruções para começar a usar Kubernetes localmente com Minikube.

---

## 🧠 O que é Kubernetes?

Kubernetes (ou K8s) é uma plataforma open-source para **orquestração de containers**. Ele automatiza o deployment, o scaling e a operação de aplicações em containers.

Imagine que você tem vários containers Docker rodando sua aplicação. Gerenciar isso manualmente seria complicado. O Kubernetes entra para:

- Distribuir os containers entre servidores (nós)
- Reiniciar containers que falham
- Escalar automaticamente (mais ou menos instâncias)
- Fazer deploys sem downtime

---

## 🧩 Componentes principais do Kubernetes

### 1. **Pod**
- Menor unidade do Kubernetes.
- Pode conter um ou mais containers que compartilham rede e armazenamento.

### 2. **Node**
- Máquina (física ou virtual) onde os Pods rodam.

### 3. **Cluster**
- Conjunto de Nodes gerenciados pelo Kubernetes.

### 4. **Deployment**
- Define como os Pods devem ser criados e atualizados.

### 5. **Service**
- Expõe os Pods para comunicação interna ou externa.

### 6. **Ingress**
- Gerencia o acesso HTTP/HTTPS externo aos Services.

---

## 📦 Exemplo prático: Deploy de uma aplicação Node.js com MongoDB

### Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: node-app
  template:
    metadata:
      labels:
        app: node-app
    spec:
      containers:
      - name: node-container
        image: node:14
        ports:
        - containerPort: 3000
```
Com o argumento `replicas: 2` é criada duas Instâncias desse container.

### Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: node-service
spec:
  selector:
    app: node-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: LoadBalancer
```

Esse Service escuta na porta 80 e redireciona para a porta 3000 dos Pods, fazendo balanceamento de carga entre eles.

---

## 🧪 O que é Minikube?

**Minikube** é uma ferramenta que permite rodar um cluster Kubernetes localmente. Ideal para testes, aprendizado e desenvolvimento.

### Vantagens:
- Fácil de instalar e usar
- Ideal para aprender Kubernetes sem gastar com cloud
- Permite testar Deployments, Services, Ingress, Volumes, etc.

### Exemplo de uso:
```bash
minikube start
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
minikube service node-service
```

---

## 🧭 Fluxo de trabalho com Minikube

1. Inicia o cluster com `minikube start`
2. Cria arquivos YAML (Deployment, Service, etc.)
3. Aplica com `kubectl apply -f arquivo.yaml`
4. Testa os serviços com `minikube service nome-do-service`
5. Monitora com `kubectl get pods`, `kubectl logs`, etc.

---

## 📚 Sugestões de aprendizado

- Entender bem YAML e estrutura dos manifests
- Aprender sobre ConfigMaps e Secrets
- Explorar Helm (gerenciador de pacotes para Kubernetes)
- Estudar StatefulSets (para bancos de dados)
- Praticar com Minikube ou Kind (Kubernetes in Docker)

