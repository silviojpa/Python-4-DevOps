# Day-20 | Automação CI/CD com Python 🛡️

CI/CD é o processo de automatizar as etapas de desenvolvimento, desde a integração do código (merge) até o deployment em produção. O Python é ideal para esta camada de orquestração porque pode interagir com todas as ferramentas envolvidas (Git, Docker, Kubernetes, ferramentas de teste).

O Papel do Python no CI/CD

Python pode automatizar qualquer etapa que exija lógica complexa ou integração entre sistemas:

1- Orquestração de Build: Executar docker build (Dia 17).

2- Testes e Cobertura: Rodar testes unitários e de integração.

3- Análise de Código: Chamar ferramentas de linting ou segurança.

4- Deployment: Interagir com a API do Kubernetes (Dia 19) ou Ansible (Dia 18).

Exemplo Prático: Um Script de Deployment Completo

Vamos criar um pipeline simulado onde um script Python:

    1- Builda a imagem Docker.

    2- Taggeia a imagem.

    3- Pusha para um registro de imagens.

    4- Atualiza o Deployment no Kubernetes.

Pré-requisitos:

- `docker` e `kubernetes` instalados.

- Acesso a um cluster K8s (Dia 19).

- Uma imagem para buildar (pode ser o `Dockerfile` do Dia 16).

````Python

import docker
from kubernetes import client, config
import subprocess
import os

# --- 1. VARIÁVEIS DE CONFIGURAÇÃO ---
# Assume que a versão é obtida do Git ou de um arquivo de build
APP_VERSION = "v1.0.1" 
IMAGE_NAME = "minha-app-python"
REGISTRY = "meu-registro-docker" # Ex: hub.docker.com/usuario
FULL_IMAGE_NAME = f"{REGISTRY}/{IMAGE_NAME}:{APP_VERSION}"
K8S_DEPLOYMENT_NAME = "backend-api"
K8S_NAMESPACE = "default"

# --- 2. CONFIGURAÇÃO DE CLIENTES ---
docker_client = docker.from_env()

def load_kube_config():
    try:
        config.load_kube_config()
        return client.AppsV1Api()
    except Exception as e:
        print(f"ERRO: Falha ao carregar config K8s: {e}")
        exit(1)

k8s_apps_v1 = load_kube_config()

# --- 3. FUNÇÕES DO PIPELINE ---

def step_build_and_tag():
    """Builda a imagem Docker e a taggeia."""
    print(f"\n[PASSO 1/4] -> Construindo imagem {FULL_IMAGE_NAME}...")
    try:
        # Usa o diretório atual onde o script está como contexto de build
        docker_client.images.build(
            path='.', 
            tag=FULL_IMAGE_NAME,
            rm=True # Remove containers intermediários
        )
        print("Build concluído e imagem taggeada localmente.")
        return True
    except docker.errors.BuildError as e:
        print(f"ERRO DE BUILD: {e}")
        return False

def step_push_image():
    """Faz o push da imagem para o registro remoto."""
    print(f"\n[PASSO 2/4] -> Enviando imagem para o registro: {REGISTRY}...")
    try:
        # É necessário estar logado no Docker CLI para que isso funcione
        # Ex: docker login meu-registro-docker
        
        # O push é feito tag por tag, mas o objeto de retorno é complexo
        docker_client.images.push(REGISTRY, tag=f"{IMAGE_NAME}:{APP_VERSION}")
        print("Push concluído com sucesso.")
        return True
    except docker.errors.APIError as e:
        print(f"ERRO DE PUSH: Verifique se está logado no Docker: {e}")
        return False

def step_run_tests():
    """Simula a execução de testes unitários."""
    print(f"\n[PASSO 3/4] -> Executando testes unitários...")
    try:
        # Em CI/CD real, você chamaria: subprocess.run(['pytest', ...])
        # Aqui, simulamos que a função de teste é executada
        # Supondo que 1 seja sucesso e 0 seja falha
        test_result = os.system("python -m unittest tests/test_app.py") # Exemplo com a CLI
        
        if test_result == 0:
            print("Testes concluídos com sucesso.")
            return True
        else:
            print("ERRO DE TESTE: Falha nos testes unitários.")
            return False
    except Exception as e:
        print(f"ERRO ao rodar testes: {e}")
        return False

def step_update_kubernetes_deployment():
    """Atualiza a tag da imagem no Deployment K8s (Rolling Update)."""
    print(f"\n[PASSO 4/4] -> Aplicando Rolling Update no K8s...")
    
    # 1. Patch de atualização (Apenas o que queremos mudar)
    patch_body = {
        "spec": {
            "template": {
                "spec": {
                    "containers": [
                        {
                            "name": "nginx-container", # Deve corresponder ao nome do container no manifest
                            "image": FULL_IMAGE_NAME
                        }
                    ]
                }
            }
        }
    }
    
    # 2. Executa a operação Patch
    try:
        k8s_apps_v1.patch_namespaced_deployment(
            name=K8S_DEPLOYMENT_NAME,
            namespace=K8S_NAMESPACE,
            body=patch_body
        )
        print(f"Deployment '{K8S_DEPLOYMENT_NAME}' atualizado com a imagem {APP_VERSION}.")
        print("Kubernetes iniciou o Rolling Update.")
        return True
    except client.ApiException as e:
        print(f"ERRO K8S: Falha ao atualizar Deployment: {e.reason}")
        return False

# --- 5. ORQUESTRAÇÃO DO PIPELINE ---

def main_pipeline():
    """Orquestra as etapas de CI/CD."""
    
    # Se uma etapa falhar, o pipeline deve parar
    if not step_build_and_tag(): return False
    if not step_run_tests(): return False
    # O push é opcional, mas essencial para K8s
    if not step_push_image(): return False 
    if not step_update_kubernetes_deployment(): return False
    
    print("\n\n✅ SUCESSO! Pipeline de CI/CD concluído com Python.")
    return True

if __name__ == "__main__":
    main_pipeline()
````
Resumo do Dia 20

- Orquestração Centralizada: Você viu como o Python serve como o ponto central para orquestrar diversas ferramentas DevOps (Docker e Kubernetes).

- Encadeamento Lógico: Usou funções para modularizar as etapas do pipeline e return False para implementar a lógica de fail-fast (parar em caso de erro), essencial em CI/CD.

- Interação com APIs: Reutilizou o conhecimento do Dia 17 (docker_client.images.build) e do Dia 19 (k8s_apps_v1.patch_namespaced_deployment) para gerenciar o ciclo de vida do software.
