# Day-16 | Introdução ao Docker 🐬

O Docker transformou o DevOps ao popularizar a contêinerização. Um contêiner é um pacote leve, autônomo e executável de um software, que inclui tudo o que é necessário para rodar: código, runtime, bibliotecas, variáveis de ambiente e arquivos de configuração.

O objetivo deste dia é entender o conceito do Docker e como criar e rodar seu primeiro contêiner.

1. Por que Docker?
O problema clássico do desenvolvimento é: "Funciona na minha máquina!"

- Docker resolve isso: Ele garante que seu ambiente de desenvolvimento seja idêntico ao ambiente de teste e produção, pois todos rodam o mesmo contêiner.

- Portabilidade: Um contêiner Docker pode rodar em qualquer lugar (laptop, servidor cloud, VM) que tenha o Docker Engine instalado.

- Isolamento: Contêineres são isolados uns dos outros e do sistema host, garantindo segurança e evitando conflitos de dependências.

- Eficiência de Recursos: Eles são muito mais leves e rápidos de inicializar do que Máquinas Virtuais (VMs).

2. Conceitos-Chave
<img width="710" height="291" alt="image" src="https://github.com/user-attachments/assets/454beaa5-4eaa-4878-bd1f-f22b4c2f22ff" />

3. Instalando o Docker
Para começar, você precisa ter o Docker Desktop (ou Docker Engine no Linux/Servidor) instalado no seu sistema.

Ação: Baixe e instale o Docker Desktop a partir do site oficial do Docker, se ainda não o fez.

Após a instalação, verifique se o Docker está rodando:

````Bash

docker --version
docker run hello-world
````
Se você ver a mensagem de boas-vindas do `hello-world`, está pronto!

4. O Primeiro Contêiner (Pull & Run)
   
A forma mais rápida de usar o Docker é baixando uma imagem pronta e executando-a.

````Bash

# 1. Puxar a imagem (Download)
# docker pull ubuntu:latest
# O 'run' fará o pull automaticamente se a imagem não existir

# 2. Rodar um contêiner Ubuntu e executar um comando
# -i: modo interativo
# -t: aloca um TTY
docker run -it ubuntu:latest /bin/bash
````
O que acontece: Você entra em um shell BASH isolado dentro do contêiner Ubuntu. Tente comandos como `ls` /, `apt update` ou `python3 --version`. Quando sair (`exit`), o contêiner para.

5. Construindo sua Própria Imagem (Dockerfile)
Agora vamos criar uma imagem simples para uma aplicação Python.

Passo 1: Crie os arquivos

Crie uma pasta chamada `python-app` com dois arquivos dentro:

`app.py`

````Python

import time
print("Iniciando aplicação Python no Docker...")
for i in range(5):
    print(f"Executando... (Iteração {i+1})")
    time.sleep(1)
print("Aplicação finalizada.")
````

`Dockerfile`

````Dockerfile

# 1. Imagem base: Um ambiente leve com Python 3.10
FROM python:3.10-slim

# 2. Definir o diretório de trabalho dentro do contêiner
WORKDIR /app

# 3. Copiar o arquivo da aplicação para o contêiner
# O ponto indica o diretório atual do build
COPY app.py /app/

# 4. Comando a ser executado quando o contêiner for iniciado
# Usamos o formato executável (lista de strings) que é preferível
CMD ["python", "app.py"]
````
Passo 2: Construir a Imagem

Navegue até a pasta `python-app` no seu terminal e execute:

````Bash

# -t: Tag (nome) para sua imagem
# .: Onde está o Dockerfile (diretório atual)
docker build -t meu-script-python .
````
Você verá o Docker executando cada step (passo) do seu `Dockerfile`.

Passo 3: Rodar o Contêiner

Use a imagem que você acabou de criar para rodar um contêiner:

````Bash

# Roda o contêiner e executa o comando CMD (python app.py)
docker run meu-script-python
````
Resultado: Sua aplicação Python é executada de forma isolada dentro do contêiner, provando que ela roda exatamente como você a configurou no `Dockerfile`.

Resumo do Dia 16
Fundamentos: Você compreendeu a diferença crucial entre Imagens (o molde) e Contêineres (a instância).

Comandos Essenciais: Aprendeu a usar docker run para executar contêineres e docker build para criar suas próprias imagens.

Contêinerização Básica: Criou um Dockerfile e empacotou com sucesso um script Python dentro de uma imagem, garantindo que ele possa rodar em qualquer ambiente Docker.

