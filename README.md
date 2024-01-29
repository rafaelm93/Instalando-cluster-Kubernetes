# Instalando o nosso cluster Kubernetes

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

## Comandos úteis do eksctl

- Listar todos os clusters EKS:

    ```bash
    eksctl get cluster -A
    ```

- Listar clusters EKS em uma região específica:

    ```bash
    eksctl get cluster -r us-east-1
    ```

- Aumentar o número de nós do cluster:

    ```bash
    eksctl scale nodegroup --cluster=eks-cluster --nodes=3 --nodes-min=1 --nodes-max=3 --name=eks-cluster-nodegroup -r us-east-1
    ```

- Diminuir o número de nós do cluster:

    ```bash
    eksctl scale nodegroup --cluster=eks-cluster --nodes=1 --nodes-min=1 --nodes-max=3 --name=eks-cluster-nodegroup -r us-east-1
    ```

- Deletar o cluster EKS:

    ```bash
    eksctl delete cluster --name=eks-cluster -r us-east-1
    ```
