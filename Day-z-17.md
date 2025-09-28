# Day-17 | Automação Docker com Python 🚀
A biblioteca oficial e recomendada para interagir com a API do Docker a partir do Python é a docker-py (ou, simplesmente, docker).

1. Instalação e Conexão
Primeiro, instale a biblioteca:

Bash

pip install docker
A biblioteca se comunica com o Docker Engine (Daemon) do seu sistema. Se você estiver usando o Docker Desktop, a conexão geralmente é automática.

2. Conectando e Verificando o Docker Engine
Você começa criando um cliente que se conecta ao seu Docker local.

Python

import docker

try:
    # Cria um cliente Docker (usa as configurações padrão do ambiente)
    client = docker.from_env()
    
    # Verifica a conexão e o status do Docker Engine
    version_info = client.version()
    print(f"--- Conexão com Docker Engine Estabelecida ---")
    print(f"Versão da API: {version_info['ApiVersion']}")
    print(f"Versão do Docker: {version_info['Version']}")
    
except Exception as e:
    print(f"Erro ao conectar ao Docker Engine: {e}")
    # Finaliza se não puder conectar
    exit(1)
3. Gerenciando Imagens
A automação envolve listar, puxar (pull) e remover imagens.

Python

# --- Puxando (Pull) uma Imagem ---
print("\n--- Puxando a Imagem Nginx ---")
try:
    # Puxa a imagem 'nginx:latest'. Equivale a 'docker pull nginx'
    image = client.images.pull('nginx', tag='latest')
    print(f"Imagem puxada: {image.tags[0]}")
except docker.errors.ImageNotFound:
    print("Imagem não encontrada.")


# --- Listando Imagens ---
print("\n--- Imagens Atualmente Disponíveis ---")
# Equivale a 'docker images'
images = client.images.list()
for img in images:
    # Filtra e mostra apenas as imagens que têm tags
    if img.tags:
        print(f"  > {img.tags[0]}")
        
# --- Removendo uma Imagem ---
# Vamos remover a imagem que criamos no Dia 16 (se ela não estiver sendo usada por um contêiner)
print("\n--- Tentando remover a imagem meu-script-python ---")
try:
    client.images.remove('meu-script-python')
    print("Imagem 'meu-script-python' removida com sucesso.")
except docker.errors.ImageNotFound:
    print("Imagem 'meu-script-python' não encontrada para remoção.")
except docker.errors.APIError as e:
    # A APIError 409 geralmente significa que o contêiner está em uso
    print(f"Não foi possível remover a imagem (Pode estar em uso): {e}")

4. Gerenciando Contêineres
Onde a automação brilha: iniciar, parar e obter logs de contêineres.

A. Criando e Executando um Contêiner
Vamos iniciar um contêiner Nginx e mapear a porta:

Python

CONTAINER_NAME = "web-service-auto"

# --- Rodando um Contêiner ---
print(f"\n--- Iniciando o Contêiner: {CONTAINER_NAME} ---")
try:
    container = client.containers.run(
        'nginx:latest',
        name=CONTAINER_NAME,
        detach=True,       # Roda em segundo plano (como -d)
        ports={'80/tcp': 8080} # Mapeia a porta 80 do contêiner para a 8080 do host
    )
    print(f"Contêiner '{CONTAINER_NAME}' iniciado. ID: {container.short_id}")
    print(f"Acesse: http://localhost:8080")
except docker.errors.ContainerError as e:
    print(f"Erro ao rodar o contêiner: {e}")
except docker.errors.APIError as e:
    # Se o contêiner já existir (nome duplicado)
    if 'Conflict' in str(e):
        print(f"Contêiner {CONTAINER_NAME} já existe. Tentando iniciar/remover...")
    else:
        print(f"Erro de API: {e}")


# --- Listando Contêineres ---
print("\n--- Contêineres Ativos ---")
# Equivale a 'docker ps'
running_containers = client.containers.list()
for cont in running_containers:
    print(f"  > {cont.name} ({cont.status})")

B. Parando e Removendo um Contêiner
É essencial limpar recursos após o uso.

Python

# --- Parando e Removendo o Contêiner ---
print(f"\n--- Parando e Removendo o Contêiner: {CONTAINER_NAME} ---")
try:
    # Pega a referência do contêiner pelo nome
    container_to_stop = client.containers.get(CONTAINER_NAME) 
    
    # 1. Para (stop) o contêiner
    container_to_stop.stop()
    print("Contêiner parado.")
    
    # 2. Remove (remove) o contêiner
    container_to_stop.remove() 
    print("Contêiner removido com sucesso.")

except docker.errors.NotFound:
    print(f"Contêiner {CONTAINER_NAME} não encontrado (já foi removido).")
except Exception as e:
    print(f"Erro ao manipular o contêiner: {e}")

Resumo do Dia 17
Conexão Programática: Você aprendeu a usar docker.from_env() para estabelecer uma conexão com o Docker Engine.

API de Alto Nível: Utilizou os clientes client.images e client.containers para realizar operações Docker sem precisar chamar o CLI (docker).

Operações Essenciais: Automatizou tarefas críticas de DevOps, como puxar imagens (.pull()), listar recursos (.list()), rodar contêineres com mapeamento de portas (.run()) e garantir a limpeza dos recursos (.stop(), .remove()).

A partir daqui, qualquer fluxo de trabalho Docker pode ser incorporado e gerenciado por um script Python, tornando-o ideal para automação de testes ou implantações simples.
