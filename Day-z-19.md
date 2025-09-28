# Day-19 | Kubernetes e Python 🚢
1. O Cliente Python para Kubernetes
A maneira oficial de interagir com a API do Kubernetes a partir do Python é usando a biblioteca Kubernetes Python Client. Ela permite que você execute qualquer operação que faria com kubectl, mas diretamente do seu código.

Instalação:

Bash

pip install kubernetes
Pré-requisitos:
Você precisa ter acesso a um cluster K8s (pode ser um cluster local como o Minikube ou o cluster do Docker Desktop) e um arquivo de configuração válido (~/.kube/config). O cliente Python usará este arquivo automaticamente.

2. Conexão e Configuração
Você deve carregar a configuração do cluster antes de interagir com ele.

Python

from kubernetes import client, config

def load_kube_config():
    """Carrega a configuração do Kubernetes (local ou in-cluster)."""
    try:
        # Tenta carregar a configuração do arquivo padrão ~/.kube/config
        config.load_kube_config() 
        print("--- Configuração do K8s carregada do arquivo kubeconfig. ---")
    except config.ConfigException:
        # Se não estiver em um PC local, tenta carregar de dentro de um Pod K8s
        try:
            config.load_incluster_config()
            print("--- Configuração do K8s carregada in-cluster. ---")
        except config.ConfigException:
            raise Exception("Não foi possível carregar a configuração do K8s.")

# Carrega a configuração
load_kube_config()

# Cria a API Core (para gerenciar Nodes, Pods, Namespaces)
v1 = client.CoreV1Api()
# Cria a API Apps (para gerenciar Deployments, StatefulSets, DaemonSets)
apps_v1 = client.AppsV1Api()
3. Operações de Leitura (Monitoramento)
A primeira etapa da automação DevOps é a leitura de status e o monitoramento.

Python

# --- Listando Pods no Namespace 'default' ---
print("\n--- Pods Ativos no Namespace 'default' ---")

# Chama o método que equivale a 'kubectl get pods'
pod_list = v1.list_namespaced_pod(namespace="default", watch=False)

if pod_list.items:
    for pod in pod_list.items:
        print(f"  > Nome: {pod.metadata.name}, Status: {pod.status.phase}, IP: {pod.status.pod_ip}")
else:
    print("Nenhum Pod encontrado no namespace 'default'.")


# --- Obtendo o status dos Nodes ---
print("\n--- Nodes do Cluster ---")
node_list = v1.list_node(watch=False)

for node in node_list.items:
    # Obtém a condição de 'Ready' do Node
    ready_status = next((c.status for c in node.status.conditions if c.type == "Ready"), "Unknown")
    print(f"  > Node: {node.metadata.name}, Status Ready: {ready_status}")
4. Operações de Escrita (Deployment e Escala)
A automação de deployment e a mudança de escala (scaling) são tarefas comuns de SRE/DevOps.

A. Criando um Deployment
Você define o manifest (definição YAML) como um dicionário Python e o envia para a API.

Python

DEPLOYMENT_NAME = "python-auto-nginx"

# 1. Definição do Manifest (YAML traduzido para Python Dict)
deployment_manifest = {
    "apiVersion": "apps/v1",
    "kind": "Deployment",
    "metadata": {"name": DEPLOYMENT_NAME},
    "spec": {
        "replicas": 1,
        "selector": {"matchLabels": {"app": DEPLOYMENT_NAME}},
        "template": {
            "metadata": {"labels": {"app": DEPLOYMENT_NAME}},
            "spec": {
                "containers": [
                    {
                        "name": "nginx-container",
                        "image": "nginx:latest", # Imagem do nosso Dia 17
                        "ports": [{"containerPort": 80}]
                    }
                ]
            }
        }
    }
}

print(f"\n--- Criando Deployment: {DEPLOYMENT_NAME} ---")
try:
    # Chama o método que equivale a 'kubectl apply -f deployment.yaml'
    apps_v1.create_namespaced_deployment(
        body=deployment_manifest,
        namespace="default"
    )
    print(f"Deployment '{DEPLOYMENT_NAME}' criado com sucesso.")
except client.ApiException as e:
    # 409 Conflict significa que o Deployment já existe
    if e.status == 409:
        print(f"Deployment '{DEPLOYMENT_NAME}' já existe.")
    else:
        print(f"Erro ao criar Deployment: {e}")
B. Alterando a Escala (Scaling)
Vamos aumentar o número de réplicas de 1 para 3.

Python

NEW_REPLICAS = 3

print(f"\n--- Alterando a escala para {NEW_REPLICAS} réplicas ---")

# 1. Cria um objeto Patch (apenas as mudanças)
scale_patch = {"spec": {"replicas": NEW_REPLICAS}}

# 2. Chama o método 'patch'
try:
    apps_v1.patch_namespaced_deployment_scale(
        name=DEPLOYMENT_NAME,
        namespace="default",
        body=scale_patch
    )
    print(f"Escala do Deployment '{DEPLOYMENT_NAME}' alterada para {NEW_REPLICAS} réplicas.")
except client.ApiException as e:
    print(f"Erro ao escalar Deployment: {e}")
Resumo do Dia 19
Cliente Oficial: Você aprendeu a usar a biblioteca kubernetes para automatizar tarefas.

Configuração: Dominou o carregamento de configurações com config.load_kube_config().

APIs: Entendeu a diferença entre as APIs Core (client.CoreV1Api()) e Apps (client.AppsV1Api()).

Automação de Cluster: Praticou o monitoramento (listing Pods/Nodes) e a manipulação de recursos (creating e scaling Deployments) usando objetos e métodos Python.

A partir daqui, você pode construir painéis de monitoramento personalizados, scripts de recuperação automática (autorepair) e pipelines de CI/CD que gerenciam todo o ciclo de vida da aplicação no K8s.
