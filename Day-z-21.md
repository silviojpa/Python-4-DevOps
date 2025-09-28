Exatamente! Após construir e automatizar seu pipeline de CI/CD, o passo lógico é garantir que tudo está funcionando bem. O Dia 21 marca o início do ciclo de Monitoramento e Scaling, começando com a ferramenta de coleta de métricas de fato no mundo DevOps: Prometheus.

Day-21 | Monitoramento com Prometheus 📊
O Prometheus é um sistema open-source de monitoramento e alertas. Ele coleta e armazena métricas como séries temporais, ou seja, as métricas são acompanhadas do timestamp em que foram gravadas e de pares chave-valor chamados rótulos (labels).

1. Conceitos Fundamentais
Séries Temporais: A métrica em si (cpu_usage), o valor, o timestamp e os rótulos (ex: instance="web-01", environment="prod").

Pull Model: Diferente de outros sistemas, o Prometheus puxa as métricas das aplicações em intervalos definidos, acessando endpoints HTTP expostos pelas próprias aplicações (geralmente /metrics).

Exporters: Serviços intermediários que convertem métricas de sistemas não nativos (como bancos de dados, servidores de SO) para o formato Prometheus.

PromQL: A poderosa linguagem de consulta do Prometheus, usada para fazer cálculos, agregações e análises nas séries temporais.

2. Python como Exporter de Métricas
O Python tem uma biblioteca excelente para expor métricas no formato que o Prometheus entende. Este é o ponto crucial de integração.

Instalação:

Bash

pip install prometheus_client
Exemplo de Script Python (app_with_metrics.py):

Vamos criar um pequeno servidor web Python que expõe duas métricas personalizadas: um contador e um medidor (gauge).

Python

from prometheus_client import start_http_server, Counter, Gauge
import random
import time
import requests

# 1. Definindo as Métricas
# COUNTER: Um valor que só aumenta (ex: total de requisições, erros).
REQUESTS_TOTAL = Counter(
    'app_http_requests_total', # Nome da métrica
    'Total de requisições HTTP para a aplicação', # Descrição
    ['endpoint'] # Rótulos (labels) para diferenciar as requisições por endpoint
)

# GAUGE: Um valor que pode subir ou descer (ex: temperatura, número de itens na fila).
IN_QUEUE_ITEMS = Gauge(
    'app_queue_size', 
    'Número atual de itens aguardando na fila de processamento'
)


def process_request(endpoint):
    """Simula o processamento de uma requisição com alguma lógica."""
    
    # Incrementa o contador para este endpoint específico
    REQUESTS_TOTAL.labels(endpoint).inc()
    
    # Simula a alteração do valor do Gauge
    current_queue = random.randint(1, 10)
    IN_QUEUE_ITEMS.set(current_queue)
    
    print(f"Processando {endpoint}. Fila atual: {current_queue}")
    time.sleep(random.random() * 0.5) # Simula o tempo de trabalho


if __name__ == '__main__':
    # Porta onde o servidor de métricas será exposto.
    # Esta porta será configurada no Prometheus para ser "puxada" (scraped).
    METRICS_PORT = 8000
    
    # 2. Inicia o servidor HTTP para expor as métricas
    start_http_server(METRICS_PORT)
    print(f"Servidor de Métricas Prometheus rodando na porta {METRICS_PORT}")
    
    # Loop principal da aplicação (simula tráfego)
    while True:
        process_request('/')
        process_request('/api/v1/data')
3. Verificando as Métricas
Rode o script Python: python app_with_metrics.py

Abra seu navegador e acesse: http://localhost:8000/metrics

Você verá a saída das métricas no formato Prometheus:

# HELP app_http_requests_total Total de requisições HTTP para a aplicação
# TYPE app_http_requests_total counter
app_http_requests_total{endpoint="/"} 35.0
app_http_requests_total{endpoint="/api/v1/data"} 34.0
# HELP app_queue_size Número atual de itens aguardando na fila de processamento
# TYPE app_queue_size gauge
app_queue_size 7.0 
...
4. Configurando o Prometheus (Target)
Para que o Prometheus colete essas métricas, você precisa adicionar um job de scrape na sua configuração (prometheus.yml):

YAML

# ... outras configurações
scrape_configs:
  - job_name: 'python-app-metrics'
    # Configura o intervalo de coleta
    scrape_interval: 5s 
    static_configs:
      - targets: ['localhost:8000'] # Onde o seu script Python está rodando
Ao recarregar o Prometheus, ele começará a acessar http://localhost:8000/metrics a cada 5 segundos, armazenando os dados na sua base de séries temporais.

Resumo do Dia 21
Fundamentos: Entendeu o Pull Model e a importância dos Exporters e Rótulos no Prometheus.

Python como Exporter: Aprendeu a usar a biblioteca prometheus_client para expor métricas customizadas diretamente da sua aplicação.

Tipos de Métricas: Diferenciou e implementou Counter (aumenta) e Gauge (sobe/desce).

Integração: Viu como configurar o prometheus.yml para que o Prometheus comece a coletar (scrape) os dados do seu endpoint Python.
