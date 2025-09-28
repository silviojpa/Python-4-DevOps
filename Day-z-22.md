Day-22 | Gerenciamento de Logs com ELK Stack 📜
O ELK Stack é um acrônimo para três projetos open-source:

Elasticsearch: Um motor de busca distribuído, escalável e full-text (motor de armazenamento e pesquisa).

Logstash: Um pipeline de processamento de dados que ingere dados de diversas fontes, transforma-os e os envia para o Elasticsearch.

Kibana: Uma ferramenta de visualização (dashboard) que permite explorar e analisar os dados armazenados no Elasticsearch.

1. O Fluxo do Log Centralizado
No DevOps moderno, os logs não ficam mais em arquivos de texto em servidores individuais. Eles seguem este fluxo:

Aplicação (Python) → Filebeat (ou Logstash) → Elasticsearch → Kibana

O foco da integração com Python é na primeira etapa: garantir que sua aplicação gere logs de forma estruturada.

2. Python e Logs Estruturados
A melhor prática é gerar logs em formato JSON (logs estruturados). Isso facilita enormemente o trabalho do Logstash e do Elasticsearch, pois eles não precisam adivinhar onde o timestamp ou o nível do log estão.

A biblioteca padrão do Python, logging, pode ser estendida para gerar JSON facilmente. Vamos usar a biblioteca popular python-json-logger para isso.

Instalação:

Bash

pip install python-json-logger
Exemplo de Script Python (app_logging.py):

Python

import logging
import sys
import time
import random
from python_json_logger import jsonlogger

# 1. Configuração do Logger
LOG_FILE_PATH = 'app.log'
LOG_FORMAT = '%(asctime)s %(levelname)s %(name)s %(module)s %(funcName)s %(lineno)d %(message)s'

# Cria o logger
logger = logging.getLogger('devops-app-logger')
logger.setLevel(logging.INFO)

# Configuração do Formatter JSON
formatter = jsonlogger.JsonFormatter(LOG_FORMAT)

# 2. Handler para o Arquivo
# O Logstash ou Filebeat (Filebeat é preferível) monitorará este arquivo
file_handler = logging.handlers.RotatingFileHandler(
    LOG_FILE_PATH,
    maxBytes=1024*1024*5, # 5 MB
    backupCount=5
)
file_handler.setFormatter(formatter)
logger.addHandler(file_handler)

# 3. Handler para a Saída Padrão (útil para containers Docker!)
# No Docker, o Filebeat pode ler o STDOUT/STDERR diretamente do container.
stream_handler = logging.StreamHandler(sys.stdout)
stream_handler.setFormatter(formatter)
logger.addHandler(stream_handler)


# 4. Uso do Logger
def process_data(user_id):
    # Log de início de tarefa
    logger.info("Iniciando processamento de dados", extra={'user_id': user_id, 'task': 'data_fetch'})
    
    if random.random() < 0.2:
        # Log de erro (adiciona campos customizados)
        logger.error("Falha na conexão com o DB", extra={'user_id': user_id, 'status_code': 500})
    else:
        # Log de sucesso
        logger.info("Processamento concluído", extra={'user_id': user_id, 'records_processed': 100})

# Loop principal (simula a atividade)
if __name__ == '__main__':
    print(f"Gerando logs estruturados no arquivo: {LOG_FILE_PATH}...")
    for i in range(10):
        process_data(i + 100)
        time.sleep(0.5)

Saída de Log (Exemplo):

Ao rodar, a saída será uma linha JSON que o Filebeat/Logstash pode ler e entender facilmente:

JSON

{"asctime": "2025-09-28 08:30:30,123", "levelname": "INFO", "name": "devops-app-logger", "module": "app_logging", "funcName": "process_data", "lineno": 44, "message": "Iniciando processamento de dados", "user_id": 100, "task": "data_fetch"}
{"asctime": "2025-09-28 08:30:30,654", "levelname": "ERROR", "name": "devops-app-logger", "module": "app_logging", "funcName": "process_data", "lineno": 38, "message": "Falha na conexão com o DB", "user_id": 101, "status_code": 500}
3. O Papel do Filebeat e Logstash (Orquestração)
Em um ambiente DevOps, você usaria o Docker Compose ou Kubernetes para rodar o ELK Stack junto com sua aplicação.

Filebeat: Um shipper (transportador) leve da Elastic que é implantado em cada servidor ou contêiner de aplicação. Sua função é monitorar o arquivo app.log ou o stdout e enviar cada nova linha JSON diretamente para o Elasticsearch (ou, menos comum hoje em dia, para o Logstash).

Logstash (Opcional, mas Útil): Em cenários complexos, Logstash pode ser usado para enriquecer ou filtrar os logs antes de enviá-los ao Elasticsearch (ex: adicionar localização geográfica baseada no IP, ou descartar logs de debug).

4. Visualização no Kibana
Uma vez que o Elasticsearch recebe o JSON:

Cada linha JSON se torna um documento no Elasticsearch.

Os campos customizados (user_id, status_code) são automaticamente indexados.

O Kibana permite que você pesquise por logs com status_code: 500 e crie dashboards mostrando a contagem desses erros ao longo do tempo.

Resumo do Dia 22
ELK Stack: Entendeu o papel de cada componente: Elasticsearch (armazenamento), Logstash/Filebeat (coleta/processamento) e Kibana (visualização).

Logs Estruturados: Aprendeu a importância de logs em formato JSON para facilitar a análise.

Python com jsonlogger: Implementou o logging estruturado em Python, o ponto de partida para a centralização.

Integração com Contêineres: Viu como a saída padrão (sys.stdout) é a forma ideal de coletar logs em ambientes conteinerizados.
