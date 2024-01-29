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
```

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


## ServiceMonitors no Kube-Prometheus

O Kube-Prometheus utiliza o recurso `ServiceMonitor` para configurar o Prometheus para monitorar serviços específicos. Já vem configurado com vários `ServiceMonitors`, incluindo os do API Server, Node Exporter, Blackbox Exporter, etc.

Para listar os `ServiceMonitors`:

```bash
kubectl get servicemonitors -n monitoring
```
Para ver detalhes de um ServiceMonitor, como o do Prometheus:

```bash
kubectl get servicemonitor prometheus-k8s -n monitoring -o yaml
```
Um exemplo simples de um ServiceMonitor:

```bash
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  labels:
    app.kubernetes.io/component: prometheus
    app.kubernetes.io/instance: k8s
    app.kubernetes.io/name: prometheus
    app.kubernetes.io/part-of: kube-prometheus
    app.kubernetes.io/version: 2.41.0
  name: prometheus-k8s
  namespace: monitoring
spec:
  endpoints:
  - interval: 30s
    port: web
  - interval: 30s
    port: reloader-web
  selector:
    matchLabels:
      app.kubernetes.io/component: prometheus
      app.kubernetes.io/instance: k8s
      app.kubernetes.io/name: prometheus
      app.kubernetes.io/part-of: kube-prometheus
```

 Agora pode ser criado os ServiceMonitors para monitorar serviços adicionais do cluster! 🛠️

 # Configuração do ServiceMonitor para Nginx e Nginx Exporter no Kubernetes

## ConfigMap para Nginx
Primeiro, criamos um ConfigMap para a configuração do Nginx. O arquivo YAML é o seguinte:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    # Conteúdo do arquivo de configuração do Nginx
    server {
      listen 80;
      location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
      }
      location /metrics {
        stub_status on;
        access_log off;
      }
    }
```

Agora, aplicamos o ConfigMap:
```bash
kubectl apply -f nginx-config.yaml
```

Deployment da Aplicação Nginx

```yaml
apiVersion: apps/v1 # versão da API
kind: Deployment # tipo de recurso, no caso, um Deployment
metadata: # metadados do recurso 
  name: nginx-server # nome do recurso
spec: # especificação do recurso
  selector: # seletor para identificar os pods que serão gerenciados pelo deployment
    matchLabels: # labels que identificam os pods que serão gerenciados pelo deployment
      app: nginx # label que identifica o app que será gerenciado pelo deployment
  replicas: 3 # quantidade de réplicas do deployment
  template: # template do deployment
    metadata: # metadados do template
      labels: # labels do template
        app: nginx # label que identifica o app
      annotations: # annotations do template
        prometheus.io/scrape: 'true' # habilita o scraping do Prometheus
        prometheus.io/port: '9113' # porta do target
    spec: # especificação do template
      containers: # containers do template 
        - name: nginx # nome do container
          image: nginx # imagem do container do Nginx
          ports: # portas do container
            - containerPort: 80 # porta do container
              name: http # nome da porta
          volumeMounts: # volumes que serão montados no container
            - name: nginx-config # nome do volume
              mountPath: /etc/nginx/conf.d/default.conf # caminho de montagem do volume
              subPath: nginx.conf # subpath do volume
        - name: nginx-exporter # nome do container que será o exporter
          image: 'nginx/nginx-prometheus-exporter:0.11.0' # imagem do container do exporter
          args: # argumentos do container
            - '-nginx.scrape-uri=http://localhost/metrics' # argumento para definir a URI de scraping
          resources: # recursos do container
            limits: # limites de recursos
              memory: 128Mi # limite de memória
              cpu: 0.3 # limite de CPU
          ports: # portas do container
            - containerPort: 9113 # porta do container que será exposta
              name: metrics # nome da porta
      volumes: # volumes do template
        - configMap: # configmap do volume, nós iremos criar esse volume através de um configmap
            defaultMode: 420 # modo padrão do volume
            name: nginx-config # nome do configmap
          name: nginx-config # nome do volume
 ```

Agora, aplicamos o Deployment:
```bash
kubectl apply -f nginx-deployment.yaml
```

Service para Expor a Aplicação
```yaml
apiVersion: v1 # versão da API
kind: Service # tipo de recurso, no caso, um Service
metadata: # metadados do recurso
  name: nginx-svc # nome do recurso
  labels: # labels do recurso
    app: nginx # label para identificar o svc
spec: # especificação do recurso
  ports: # definição da porta do svc 
  - port: 9113 # porta do svc
    name: metrics # nome da porta
  selector: # seletor para identificar os pods/deployment que esse svc irá expor
    app: nginx # label que identifica o pod/deployment que será exposto
```
Aplicamos o Service:
```bash
kubectl apply -f nginx-service.yaml
```
Verificação
```bash
curl http://<EXTERNAL-IP-DO-SERVICE>:80
curl http://<EXTERNAL-IP-DO-SERVICE>:80/nginx_status
curl http://<EXTERNAL-IP-DO-SERVICE>:80/metrics
```
Com isso concluimos a configuração do ServiceMonitor para monitorar a aplicação Nginx no Kubernetes.


