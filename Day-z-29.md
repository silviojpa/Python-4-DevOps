# Day-29 | Projeto Final: Pipeline DevOps Completo 🚀

A Missão: Criar um Projeto Full-Stack Automatizado

Você irá criar um pipeline CI/CD que pega um código-fonte (simulado), o transforma em uma Imagem (Dia 16/17), faz o deployment no Kubernetes (Dia 19) e garante a observabilidade (Dia 21/22).

O Cenário da Aplicação (App Simples)

1-Aplicação: Um microserviço Python simples (app.py) que expõe um endpoint HTTP e também um endpoint de métricas Prometheus (Dia 21).

2-Tecnologias: Python, Docker, Kubernetes, Prometheus.

3-Ambiente: Repositório Git e um cluster K8s local (Docker Desktop ou Minikube).

Fases do Projeto e Conexão com a Trilha

O projeto deve ser dividido em tarefas que refletem os ciclos que você completou.

Fase 1: Preparação do Código e Contêiner
<img width="704" height="148" alt="image" src="https://github.com/user-attachments/assets/4b5ee5d2-1189-484b-98a4-313639839475" />

Fase 2: Automação CI/CD (O Coração Python)

Crie um script Python (pipeline.py) que orquestra todo o processo (reutilizando a lógica dos Dias 17 e 20).

Tarefa	Conceito da Trilha	Descrição
<img width="711" height="241" alt="image" src="https://github.com/user-attachments/assets/ef30505b-1e1d-477a-bcc3-e6cf9cb72bee" />

Fase 3: Deployment no Kubernetes
<img width="705" height="286" alt="image" src="https://github.com/user-attachments/assets/75a41565-4812-42fb-af9f-ed07e6ba9007" />

Roteiro Detalhado do `pipeline.py`

Seu script final deve ter a seguinte estrutura e lógica fail-fast (Dia 20):

````Python

# pipeline.py

# Importa todas as libs: docker, kubernetes, requests, os, sys, time, subprocess
import docker
from kubernetes import client, config
import requests
import time

# 1. Variáveis Globais (Nomes da Imagem, Deployment, Service, Versão)
# ...

# 2. Função: load_configs()
#    Carrega o cliente Docker e a configuração do Kubernetes (Dia 17, Dia 19).

# 3. Função: build_and_tag(tag) (Dia 17)
#    - Tenta rodar docker_client.images.build().
#    - Retorna True/False.

# 4. Função: run_local_test(tag) (Dia 15, Dia 25)
#    - Roda a imagem localmente (docker_client.containers.run(..., detach=True, ports={'8000/tcp': 8001}))
#    - Faz um requests.get('http://localhost:8001/metrics').
#    - Para e remove o contêiner.
#    - Retorna True se o teste de métricas for 200.

# 5. Função: push_image(tag) (Dia 17)
#    - Tenta o push da imagem.

# 6. Função: deploy_to_k8s(tag) (Dia 19)
#    - Faz o patch do Deployment K8s com a nova tag da imagem.

# 7. Função: verify_health() (Dia 15, Dia 21)
#    - Aguarda 10 segundos.
#    - Faz um requests.get('http://K8S_SERVICE_IP:PORT/').
#    - Imprime sucesso ou falha.

# 8. Função: main_pipeline() (Orquestração - Dia 20)
#    - Chama as funções em ordem (build -> test -> push -> deploy -> health).
#    - Usa if not function_call(): sys.exit(1) para garantir o fail-fast.

if __name__ == '__main__':
    main_pipeline()
````
O Último Passso: Apresentação

O último dia, o Dia 30, é dedicado a apresentar e demonstrar este projeto. Prepare-se para explicar:

- Como o Python orquestra o build (Dia 17) e o deployment (Dia 19).

- Como as métricas (Dia 21) foram integradas no código e verificadas no teste (Dia 25).

- A importância da lógica fail-fast (Dia 20) para o CI/CD.
