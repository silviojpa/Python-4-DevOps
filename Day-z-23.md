Day-23 | Estratégias de Scaling de Infraestrutura ⚙️
O scaling refere-se à capacidade de uma aplicação e sua infraestrutura de lidar com o aumento da carga de trabalho. Existem duas abordagens principais, e a escolha afeta diretamente a automação:

1. Scaling Vertical (Scale Up) vs. Horizontal (Scale Out)
Estratégia	Descrição	Vantagens	Desvantagens
Vertical	Adicionar mais recursos (CPU, RAM, disco) a um único servidor.	Mais simples de configurar.	Ponto único de falha; limite de recursos físicos.
Horizontal	Adicionar mais servidores (máquinas, VMs ou contêineres).	Resiliência (alta disponibilidade); escalabilidade infinita.	Complexidade na gestão de estado e balanceamento de carga.

Exportar para as Planilhas
No DevOps moderno (especialmente com contêineres e Cloud), o Scaling Horizontal é a estratégia preferida e a que você deve focar na automação.

2. Estratégias de Scaling Horizontal
A. Manual Scaling (Scaling sob Demanda)
Simplesmente adicionar ou remover recursos manualmente, geralmente em resposta a um alerta de métrica.

Automação Python: Usar o Cliente Kubernetes (Dia 19) ou SDKs de Cloud (Dia 24) para alterar o número de réplicas ou instâncias via um script disparado manualmente.

B. Auto-Scaling Baseado em Agendamento (Scheduled Scaling)
Ajustar a capacidade com base em padrões de tráfego previsíveis (ex: aumentar a capacidade antes da Black Friday ou no início de cada dia de trabalho).

Automação Python: Um script Python rodando no Cron (ou serviço equivalente na nuvem) que chama a API de scaling (K8s ou Cloud) em horários pré-definidos.

C. Auto-Scaling Baseado em Eventos/Métricas (Reactive Scaling)
Ajustar a capacidade em tempo real com base em métricas de desempenho (CPU, requisições por segundo, tamanho da fila, etc.).

Ferramenta Nativa: No Kubernetes, o HPA (Horizontal Pod Autoscaler) lida com isso. No Cloud, é o Auto Scaling Group.

Automação Python: Usar Python como um motor de alerta customizado que interage com o Prometheus (Dia 21) ou CloudWatch (Dia 24) e, em seguida, chama a API de scaling.

3. Exemplo de Automação com Python e Kubernetes (Dia 19)
Embora o Kubernetes tenha o HPA, o Python pode ser usado para automatizar cenários de scaling mais complexos que o HPA não suporta nativamente, como o scaling baseado no tamanho de uma fila de mensagens (SQS, Kafka).

Vamos simular um script de Auto-Scaling que monitora uma métrica e ajusta a escala de um Deployment K8s.

Python

from kubernetes import client, config
import time
import random

# --- VARIÁVEIS DE SCALING ---
K8S_DEPLOYMENT_NAME = "web-service-to-scale"
K8S_NAMESPACE = "default"
MAX_REPLICAS = 10
MIN_REPLICAS = 2
CPU_THRESHOLD = 70 # Se a CPU passar de 70%, escalamos para cima

# Inicializa o cliente K8s (reutilizando a lógica do Dia 19)
try:
    config.load_kube_config()
    apps_v1 = client.AppsV1Api()
except Exception as e:
    print(f"Erro ao carregar K8s config: {e}")
    exit(1)


def get_current_cpu_metric():
    """Simula a leitura de uma métrica de CPU do Prometheus (Dia 21)."""
    # Em um cenário real, você faria uma requisição HTTP para o Prometheus (PromQL)
    # Ex: requests.get(f"http://prometheus:9090/api/v1/query?query=avg(container_cpu_usage_seconds_total)")
    
    # Simulação: retorna um valor de CPU médio aleatório entre 10 e 90
    return random.randint(10, 90)

def set_deployment_replicas(new_replicas):
    """Ajusta o número de réplicas no Deployment K8s."""
    print(f"-> Ação de Scaling: Alterando réplicas para {new_replicas}")
    try:
        # 1. Cria um objeto Patch (apenas as mudanças)
        scale_patch = {"spec": {"replicas": new_replicas}}

        # 2. Chama o método 'patch'
        apps_v1.patch_namespaced_deployment_scale(
            name=K8S_DEPLOYMENT_NAME,
            namespace=K8S_NAMESPACE,
            body=scale_patch
        )
        print("   Sucesso: Solicitação de scaling enviada ao K8s.")
    except client.ApiException as e:
        print(f"   ERRO K8S ao escalar: {e.reason}")

def run_autoscaler():
    """Loop principal do Autoscaler customizado."""
    while True:
        cpu_usage = get_current_cpu_metric()
        
        # Obtém o status atual do Deployment
        deployment = apps_v1.read_namespaced_deployment(K8S_DEPLOYMENT_NAME, K8S_NAMESPACE)
        current_replicas = deployment.spec.replicas
        
        print(f"--- [Monitor] CPU Média: {cpu_usage}%. Réplicas Atuais: {current_replicas} ---")

        # Lógica de Scaling UP
        if cpu_usage > CPU_THRESHOLD and current_replicas < MAX_REPLICAS:
            new_replicas = min(current_replicas + 1, MAX_REPLICAS)
            print(f"ATENÇÃO: Alta utilização de CPU. Scaling UP necessário!")
            set_deployment_replicas(new_replicas)

        # Lógica de Scaling DOWN (para economizar recursos)
        elif cpu_usage < (CPU_THRESHOLD - 20) and current_replicas > MIN_REPLICAS:
            new_replicas = max(current_replicas - 1, MIN_REPLICAS)
            print("RELAX: Baixa utilização de CPU. Scaling DOWN para economizar.")
            set_deployment_replicas(new_replicas)
            
        else:
            print("Status OK. Manter réplicas.")

        time.sleep(10) # Verifica a cada 10 segundos


if __name__ == '__main__':
    # Você precisará garantir que o Deployment 'web-service-to-scale' exista
    # Exemplo: set_deployment_replicas(MIN_REPLICAS)
    print("Iniciando Autoscaler Customizado...")
    run_autoscaler()
Resumo do Dia 23
Tipos de Scaling: Entendeu a diferença entre Scaling Vertical (mais recursos em um único servidor) e Scaling Horizontal (mais servidores).

Estratégias: Conheceu as abordagens Manual, Scheduled e Reactive para dimensionamento.

Python como Motor de Decisão: Utilizou o Python para implementar a lógica de Auto-Scaling baseada em um threshold de métrica (simulada) e para orquestrar a mudança de escala no Kubernetes (patch_namespaced_deployment_scale).
