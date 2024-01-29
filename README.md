# 🚀 Bem-vindo ao Nosso Cluster Kubernetes! 🌟

## Configurando a máquina

1. Instale a ferramenta `eksctl`:

    ```bash
    curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
    sudo mv /tmp/eksctl /usr/local/bin
    ```

2. Instale o AWS CLI:

    ```bash
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    unzip awscliv2.zip
    sudo ./aws/install
    ```

3. Configure as credenciais da AWS:

    ```bash
    aws configure
    ```

## Instalando o EKS

1. Baixe e instale o `eksctl` e o AWS CLI de acordo com o seu sistema operacional.

2. Crie o cluster EKS:

    ```bash
    eksctl create cluster --name=eks-cluster --version=1.24 --region=us-east-1 --nodegroup-name=eks-cluster-nodegroup --node-type=t3.medium --nodes=2 --nodes-min=1 --nodes-max=3 --managed
    ```

3. Instale o `kubectl`:

    ```bash
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    chmod +x ./kubectl
    sudo mv ./kubectl /usr/local/bin/kubectl
    ```

4. Configure o `kubectl` para usar o cluster EKS:

    ```bash
    aws eks --region us-east-1 update-kubeconfig --name eks-cluster
    ```

5. Verifique se o `kubectl` está funcionando corretamente:

    ```bash
    kubectl get nodes
    ```

## Instalando o Kube-Prometheus

1. Clone o repositório e aplique os manifests:

    ```bash
    git clone https://github.com/prometheus-operator/kube-prometheus
    cd kube-prometheus
    kubectl create -f manifests/setup
    ```

2. Verifique a conclusão da instalação dos CRDs:

    ```bash
    kubectl get servicemonitors -A
    ```

3. Instale Prometheus e Alertmanager:

    ```bash
    kubectl apply -f manifests/
    ```

4. Verifique a conclusão da instalação:

    ```bash
    kubectl get pods -n monitoring
    ```

Agora que já temos o nosso Kube-Prometheus instalado, vamos acessar o nosso Grafana e verificar se está tudo funcionando corretamente. Para isso, vamos utilizar o `kubectl port-forward` para acessar o Grafana localmente. Execute o seguinte comando:

```bash
kubectl port-forward -n monitoring svc/grafana 33000:3000
Acesse o Grafana através do navegador: http://localhost:33000

Utilize o usuário admin e a senha admin para o primeiro login. O Grafana solicitará a alteração da senha.

Dashboards do Kube-Prometheus
O Kube-Prometheus configura vários Dashboards no Grafana para monitorar o seu cluster EKS.
Alguns exemplos incluem:
Detalhes do API Server e componentes do Kubernetes (Node, Pod, Deployment, etc.).
Detalhes do cluster EKS, como uso de CPU e memória por todos os nós.
Detalhes por namespace, incluindo pods, deployments, statefulsets e daemonsets.
Uso de CPU e memória por nó do cluster EKS.
Uso de CPU e memória por container em todos os pods do cluster EKS.

Brinque com os Dashboards e aproveite a quantidade enorme de informações fornecidas pelo Kube-Prometheus! 🚀
