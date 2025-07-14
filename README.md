# 📦 Cluster Kubernetes Local

Este repositório contém a configuração de um Cluster Kubernetes local utilizando [K3d](https://k3d.io/), uma forma leve e rápida de rodar [K3s](https://k3s.io/) dentro de containers Docker.  
O projeto inclui a implantação do Nginx, exposição via Service, e um Ingress configurado com Traefik (Ingress Controller padrão do K3s).

---

## 🛠️ Tecnologias utilizadas

- 🐳 [K3d](https://k3d.io/) – para criar clusters K3s em Docker
- ☁️ [K3s](https://k3s.io/) – distribuição leve do Kubernetes
- ⚙️ [kubectl](https://kubernetes.io/docs/reference/kubectl/) – para gerenciar recursos Kubernetes
- 🚦 [Traefik](https://traefik.io/) – como Ingress Controller
- 🌐 [Nginx](https://nginx.org/) – como aplicação de exemplo

---

## 📋 O que está incluído

- Cluster local com 1 server + 2 agentes
- Mapeamento de portas HTTP (80) e HTTPS (443)
- Deployment do Nginx com 2 réplicas
- Service para o Nginx
- Ingress para roteamento via domínio local `nginx.localhost`

---

## ✅ Pré-requisitos

- **Docker**: É necessário ter o [Docker](https://www.docker.com/products/docker-desktop/) instalado e em execução na sua máquina para utilizar o K3d.
- **Acesso de administrador**: Para editar o arquivo `hosts` e instalar ferramentas.

---

## 💻 Instalação

### 🪟 Windows

**Pré-requisito:** Instale o [Chocolatey](https://chocolatey.org/install) se ainda não tiver:

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; `
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; `
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

1. Instale o kubectl:
   ```bash
   choco install kubernetes-cli --version=1.22.1 -y
   kubectl version --client
   ```
2. Instale o k3d:
   ```bash
    choco install k3d --version=5.8.3 -y
    k3d version
   ```

---

### 🐧 Linux

1. Instale o kubectl:
   ```bash
   sudo apt-get update
   sudo apt-get install -y curl apt-transport-https
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
   sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
   kubectl version --client
   ```

2. Instale o k3d:
   ```bash
    curl -s https://raw.githubusercontent.com/rancher/k3d/main/install.sh | bash
    k3d version
    ```

---

## 🚀 Como usar

1. Criar o cluster:

```bash
k3d cluster create --config k3d-config.yaml
```

2. Aplicar recursos Kubernetes:

Execute os seguintes comandos para criar o Deployment, Service e Ingress:
```bash
kubectl apply -f k8s/01-deployment.yaml
kubectl apply -f k8s/02-service.yaml
kubectl apply -f k8s/03-ingress.yaml
```

Executando todos os arquivos de uma vez (ordem garantida pelos nomes dos arquivos):
```bash
kubectl apply -f k8s/
```

3. Configurar o arquivo hosts para acesso via nginx.localhost:

### 🪟 Windows

1. Abra o Bloco de Notas como administrador

2. Edite o arquivo: C:\Windows\System32\drivers\etc\hosts

3. Adicione a linha:

```bash
127.0.0.1 nginx.localhost
```

### 🐧 Linux

1. Abra o terminal e execute:

```bash
echo "127.0.0.1 nginx.localhost" | sudo tee -a /etc/hosts
```

---

## 🌐 Acesse no navegador:

[http://nginx.localhost](http://nginx.localhost)

---

## 🧰 Comandos úteis

| Comando                                              | Explicação                                                                                                |
|------------------------------------------------------|-----------------------------------------------------------------------------------------------------------|
| `k3d cluster create --config <arquivo>`              | Cria um cluster K3d com a configuração especificada no arquivo YAML.                                      |
| `k3d cluster list`                                   | Lista todos os clusters criados com o K3d no seu ambiente local.                                          |
| `kubectl get all`                                    | Lista todos os recursos no namespace atual, como pods, serviços, deployments, etc.                        |
| `kubectl get nodes`                                  | Mostra os nós (server e agents) do cluster Kubernetes. Útil para verificar se está ok.                    |
| `kubectl get pods`                                   | Lista todos os pods em execução no namespace atual (por padrão, default).                                 |
| `kubectl get svc`                                    | Lista os serviços (Service) disponíveis no cluster, como o do Nginx.                                      |
| `kubectl get ingress`                                | Lista todos os recursos Ingress no namespaces atual. Ideal para checar o roteamento.                      |
| `kubectl describe pod <nome>`                        | Mostra detalhes completos sobre um pod específico, incluindo eventos e status.                            |
| `kubectl logs <nome-do-pod>`                         | Exibe os logs de um pod. Útil para depuração de aplicações.                                               |
| `kubectl exec -it <nome-do-pod> -- /bin/bash`        | Abre um terminal interativo dentro de um pod.                                                             |
| `kubectl delete pod <nome>`                          | Exclui um pod específico. O Kubernetes pode recriá-lo se estiver em um deployment.                        |
| `kubectl apply -f <arquivo>`                         | Aplica configurações de um arquivo YAML ao cluster.                                                       |
| `kubectl delete -f <arquivo>`                        | Remove os recursos definidos em um arquivo YAML.                                                          |
| `kubectl get namespaces`                             | Lista todos os namespaces disponíveis no cluster.                                                         |
| `kubectl top nodes`                                  | Mostra o uso de recursos (CPU/memória) de cada nó no cluster.                                             |
| `kubectl top pods`                                   | Mostra o uso de recursos (CPU/memória) de cada pod no namespace atual.                                    |
| `kubectl port-forward svc/<nome-do-service> 8080:80` | Redireciona uma porta local para um serviço do cluster.                                                   |
| `k3d cluster delete multinodes`                      | Exclui completamente o cluster chamado multinodes. Use para limpar tudo.                                  |

## 🚦 Verificar ou atualizar Traefik utilizado no K3s

- **Verificar a imagem atual do Traefik:**
  ```bash
  kubectl -n kube-system get deployment traefik -o=jsonpath='{.spec.template.spec.containers[0].image}'
  ```
- **Atualizar a imagem do Traefik (exemplo para a versão 2.11.18):**
  ```bash
    kubectl -n kube-system set image deployment/traefik traefik=rancher/mirrored-library-traefik:2.11.18
    ```
- **Verificar o status do Traefik:**
- ```bash
  kubectl -n kube-system rollout status deployment/traefik
  ```