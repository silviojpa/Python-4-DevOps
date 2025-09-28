# Day-29 | Projeto Final: Pipeline DevOps Completo 🚀
A Missão: Criar um Projeto Full-Stack Automatizado
Você irá criar um pipeline CI/CD que pega um código-fonte (simulado), o transforma em uma Imagem (Dia 16/17), faz o deployment no Kubernetes (Dia 19) e garante a observabilidade (Dia 21/22).

O Cenário da Aplicação (App Simples)
Aplicação: Um microserviço Python simples (app.py) que expõe um endpoint HTTP e também um endpoint de métricas Prometheus (Dia 21).

Tecnologias: Python, Docker, Kubernetes, Prometheus.

Ambiente: Repositório Git e um cluster K8s local (Docker Desktop ou Minikube).

Fases do Projeto e Conexão com a Trilha
O projeto deve ser dividido em tarefas que refletem os ciclos que você completou.

Fase 1: Preparação do Código e Contêiner
Tarefa	Conceito da Trilha	Descrição
A1: Código	Python Básico (Dia 1-5), Métricas (Dia 21)	Crie um app.py que: 1. Responde em / (Olá mundo). 2. Expõe métricas customizadas em /metrics.
A2: Imagem	Docker (Dia 16)	Crie um Dockerfile para contêinerizar a aplicação, usando uma imagem base slim.

Exportar para as Planilhas
Fase 2: Automação CI/CD (O Coração Python)
Crie um script Python (pipeline.py) que orquestra todo o processo (reutilizando a lógica dos Dias 17 e 20).

Tarefa	Conceito da Trilha	Descrição
B1: Build & Tag	Automação Docker (Dia 17)	Use docker-py para buildar a imagem localmente e taggeá-la com uma tag de versão.
B2: Teste (Simulado)	CI/CD Melhores Práticas (Dia 25)	Adicione uma função de teste que verifica se o endpoint /metrics do contêiner recém-buildado está acessível localmente antes do push. (Use requests - Dia 15 - dentro do script).
B3: Push	Automação Docker (Dia 17)	Faça o push da imagem para o registro (pode ser o Docker Hub, ou apenas simule o comando se não quiser fazer o push real).

Exportar para as Planilhas
Fase 3: Deployment no Kubernetes
Tarefa	Conceito da Trilha	Descrição
C1: Manifestos K8s	Kubernetes (Dia 19)	Crie um deployment.yaml e um service.yaml. O Deployment deve usar a imagem taggeada na Fase B.
C2: Orquestração K8s	Kubernetes e Python (Dia 19)	Adicione ao pipeline.py a lógica para usar o cliente Kubernetes para aplicar (ou patchar) o Deployment e o Service no cluster.
C3: Validação de Health	Monitoramento (Dia 21)	Adicione um passo final ao pipeline.py que aguarda 60 segundos e depois verifica (com requests) se o Service K8s está respondendo com 200 OK.

Exportar para as Planilhas
Roteiro Detalhado do pipeline.py
Seu script final deve ter a seguinte estrutura e lógica fail-fast (Dia 20):

Python

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
O Último Passso: Apresentação
O último dia, o Dia 30, é dedicado a apresentar e demonstrar este projeto. Prepare-se para explicar:

Como o Python orquestra o build (Dia 17) e o deployment (Dia 19).

Como as métricas (Dia 21) foram integradas no código e verificadas no teste (Dia 25).

A importância da lógica fail-fast (Dia 20) para o CI/CD.
