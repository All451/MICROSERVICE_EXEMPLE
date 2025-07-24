
# 🚀 Microservices Demo

> Um sistema modular de microservices com **Node.js, Docker, Kubernetes, Jenkins, Argo CD, Istio, Prometheus, Grafana, Secrets, Auto Scaling e GitOps**.

Um projeto completo para demonstrar boas práticas em arquitetura de microservices, pronto para desenvolvimento local e produção.

---

## 📦 Tecnologias Utilizadas

| Camada | Tecnologia |
|-------|-----------|
| 🖥️ Backend | Node.js + Express |
| 🐳 Containerização | Docker + Docker Compose |
| ☸️ Orquestração | Kubernetes (K8s) |
| 🔐 Segurança | Kubernetes Secrets, NetworkPolicy, PodSecurity |
| 🔄 Escalonamento | HorizontalPodAutoscaler (HPA) |
| 🌐 Gateway & Load Balancer | Nginx + Istio (Service Mesh) |
| 🔄 CI/CD | Jenkins (local) + GitHub Actions + Argo CD (GitOps) |
| 📊 Monitoramento | Prometheus + Grafana (`kube-prometheus-stack`) |
| 🧩 Infra como Código | YAML Kubernetes, Helm (futuro) |

---

## 🧩 Arquitetura

```
Usuário
   │
   ▼
[ Nginx / Istio Ingress ]
   │
   ▼
[ API Gateway ] → Roteia para:
   ├─→ [ Users Service ]
   ├─→ [ Orders Service ]
   ├─→ [ Products Service ]
   ├─→ [ Inventory Service ]
   └─→ [ Payments Service ]
```

- Todos os serviços são independentes.
- Comunicação via HTTP/REST.
- Deploy orquestrado com Kubernetes.
- CI/CD com GitOps (Argo CD).

---

## 📁 Estrutura do Projeto

```
microservices-demo/
├── gateway/                 # API Gateway (Node.js)
├── services/                # Microservices (users, orders, etc.)
├── load-balancer/           # nginx.conf para Docker
├── jenkins/                 # Persistência do Jenkins
├── k8s/                     # Manifestos Kubernetes
│   ├── 00-namespace.yaml
│   ├── 01-secrets/          # Senhas e tokens
│   ├── 02-services/         # Deploy dos serviços
│   ├── 03-gateway/          # Gateway no K8s
│   ├── 04-ingress/          # Ingress NGINX
│   ├── 05-monitoring/       # Configuração do Prometheus/Grafana
│   ├── 06-security/         # NetworkPolicy e segurança
│   └── 07-istio/            # Service Mesh (Istio)
├── argocd/                  # App of Apps (GitOps)
├── .github/workflows/       # GitHub Actions CI
├── docker-compose.yml       # Ambiente local
└── README.md                # Este arquivo
```

---

## 🚀 Como Executar

### 1. 🖥️ Ambiente Local (Docker)

```bash
# Subir todos os serviços
docker-compose up --build
```

Acessar:
- **API Gateway**: `http://localhost:3000/api/users`
- **Nginx Load Balancer**: `http://localhost/api/users`
- **Jenkins**: `http://localhost:8080`

> ✅ Útil para desenvolvimento e testes rápidos.

---

### 2. ☸️ Ambiente de Produção (Kubernetes)

#### Pré-requisitos

- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Minikube](https://minikube.sigs.k8s.io/docs/start/) ou cluster K8s
- [Helm](https://helm.sh/docs/intro/install/)
- [Istioctl](https://istio.io/latest/docs/setup/getting-started/#download) (opcional)
- [Argo CD CLI](https://argo-cd.readthedocs.io/en/stable/cli_installation/)

#### Passo a passo

```bash
# 1. Iniciar cluster
minikube start
minikube addons enable ingress

# 2. Instalar Istio (opcional, para service mesh)
istioctl install --set profile=demo -y
kubectl label namespace default istio-injection=enabled
kubectl create namespace microservices
kubectl label namespace microservices istio-injection=enabled

# 3. Aplicar configurações Kubernetes
kubectl apply -f k8s/00-namespace.yaml
kubectl apply -f k8s/01-secrets/
kubectl apply -f k8s/02-services/
kubectl apply -f k8s/03-gateway/
kubectl apply -f k8s/04-ingress/
kubectl apply -f k8s/06-security/
kubectl apply -f k8s/07-istio/

# 4. Instalar monitoramento
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install kube-prometheus \
  -f k8s/05-monitoring/values.yaml \
  prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace

# 5. Instalar Argo CD (GitOps)
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl apply -f argocd/app-of-apps.yaml
```

---

## 🔐 Acesso aos Serviços

| Serviço | URL |
|-------|-----|
| Users API | `http://demo.local/api/users` |
| Health Check | `http://demo.local/health` |
| Grafana | `http://<minikube-ip>:30081` (login: `admin/admin`) |
| Prometheus | `http://<minikube-ip>:30080` |
| Argo CD | `https://<minikube-ip>:30082` (use `argocd login`) |

> Adicione ao `/etc/hosts`:
> ```bash
> echo "$(minikube ip) demo.local" | sudo tee -a /etc/hosts
> ```

---

## 🔄 CI/CD Pipeline

### 🛠️ Local: Jenkins
- Monitora alterações nos serviços.
- Constrói imagens Docker.
- Reimplanta contêineres.

### ☁️ Produção: GitOps com Argo CD
1. Push no GitHub → GitHub Actions executa CI
2. Argo CD detecta mudança no repositório
3. Sincroniza automaticamente com o cluster K8s

> ✅ Imutabilidade, rastreabilidade e auto-healing.

---

## 🔒 Segurança

- **Secrets**: Senhas e tokens armazenados com `Kubernetes Secrets`.
- **NetworkPolicy**: Bloqueia tráfego não autorizado entre Pods.
- **Pod Security**: Namespace em modo `restricted`.
- **Service Mesh**: Istio para TLS, autenticação e observabilidade.

---

## 📊 Monitoramento

- **Prometheus**: Coleta métricas de CPU, memória, latência.
- **Grafana**: Dashboards para visualização.
- **AlertManager**: Alertas configuráveis (ex: alta carga).

> Útil para detectar problemas antes dos usuários.

---

## 📈 Auto Scaling

O serviço `users` escala automaticamente com base na CPU:

```yaml
# HPA (HorizontalPodAutoscaler)
targetCPUUtilization: 50%
minReplicas: 1
maxReplicas: 5
```

Teste com carga:
```bash
# Gerar carga (em outro terminal)
while true; do curl http://demo.local/api/users; sleep 0.1; done
```

Verifique:
```bash
kubectl get hpa -n microservices
```

---

## 🛠️ Comandos Úteis

| Função | Comando |
|------|--------|
| Listar Pods | `kubectl get pods -n microservices` |
| Logs de um Pod | `kubectl logs deploy/users -n microservices` |
| Acessar serviço | `kubectl port-forward svc/gateway-svc 3000:80 -n microservices` |
| Ver Secrets | `kubectl get secrets -n microservices` |
| Escalar manualmente | `kubectl scale deploy/users --replicas=3 -n microservices` |
| Acessar Grafana | `minikube service kube-prometheus-grafana -n monitoring` |

---

## 📂 Próximos Passos (Futuro)

- [ ] Adicionar banco de dados (PostgreSQL + Secret)
- [ ] Usar Helm Charts para implantação
- [ ] Canary Deployments com Istio
- [ ] Integração com Hashicorp Vault
- [ ] Sealed Secrets para GitOps seguro
- [ ] Testes automatizados (Jest)
- [ ] Dashboard customizado no Grafana

---

## 🙌 Contribuição

Contribuições são bem-vindas! Abra uma **Issue** ou **Pull Request** para:

- Corrigir bugs
- Melhorar documentação
- Adicionar novos serviços
- Integrar novas tecnologias
