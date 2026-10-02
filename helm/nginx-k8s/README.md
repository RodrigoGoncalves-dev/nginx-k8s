# Chart Helm: nginx-k8s

Este chart empacota os manifests do repositório como uma instalação Helm: um
`Deployment` do NGINX, um `ConfigMap` com a configuração do servidor e, por
padrão, o exporter do NGINX para métricas Prometheus exposto por um `Service`.

## Pré-requisitos

- Kubernetes 1.19 ou superior
- Helm 3

## Instalação

Execute a partir da raiz do repositório:

```bash
helm install nginx ./helm/nginx-k8s
```

O nome `nginx` é o *release name*. Os recursos receberão nomes como
`nginx-nginx-k8s`, permitindo mais de uma instalação no mesmo cluster.

Para instalar em um namespace específico, criando-o se necessário:

```bash
helm install nginx ./helm/nginx-k8s --namespace web --create-namespace
```

## Operações usuais

```bash
# Validar os templates localmente
helm lint ./helm/nginx-k8s

# Conferir os manifests que serão gerados, sem aplicar
helm template nginx ./helm/nginx-k8s

# Alterar valores de uma instalação existente
helm upgrade nginx ./helm/nginx-k8s --set replicaCount=3

# Consultar os recursos e a configuração usada
helm status nginx
helm get values nginx

# Remover a instalação
helm uninstall nginx
```

## Personalização

Crie um arquivo, por exemplo `meus-valores.yaml`:

```yaml
replicaCount: 3

image:
  tag: "1.19.1"

exporter:
  resources:
    requests:
      cpu: "100m"
      memory: 64Mi

nginxConfig: |
  server {
    listen 80;
    location / {
      return 200 "NGINX configurado por Helm\\n";
    }
    location /metrics {
      stub_status on;
      access_log off;
    }
  }
```

Em seguida, instale ou atualize com esse arquivo:

```bash
helm upgrade --install nginx ./helm/nginx-k8s -f meus-valores.yaml
```

### Valores principais

| Valor | Padrão | Finalidade |
| --- | --- | --- |
| `replicaCount` | `2` | Número de pods NGINX. |
| `image.repository` / `image.tag` | `nginx` / `1.19.1` | Imagem do NGINX. |
| `exporter.enabled` | `true` | Cria o container exporter e o Service de métricas. |
| `exporter.port` | `9113` | Porta do exporter dentro do pod. |
| `exporter.resources` | 0.3 CPU e 128 Mi | Requests e limits do exporter. |
| `service.type` / `service.port` | `ClusterIP` / `9113` | Serviço que expõe as métricas. |
| `podAnnotations` | anotações Prometheus | Metadados do pod para descoberta de métricas. |
| `nginxConfig` | configuração padrão | Conteúdo de `/etc/nginx/conf.d/default.conf`. |

O `Service` criado por este chart expõe somente as métricas, como o manifesto
original. Para acessar HTTP dentro do cluster, use o IP/pod diretamente ou
estenda o chart com um Service HTTP conforme a necessidade.
