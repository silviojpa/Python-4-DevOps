# Day-18 | Ansible e Python ⚙️

1. O que é Ansible?

O Ansible é uma ferramenta agentless (sem agente) que automatiza provisionamento de software, gerenciamento de configuração e implantação de aplicativos.

- Agentless: Ele se conecta aos servidores remotos via SSH (ou WinRM) e executa os comandos neles. Não há software cliente para instalar.

- Playbooks: Arquivos YAML que descrevem o estado desejado dos seus sistemas.

2. A Integração Natural

O Python é usado no Ansible de duas maneiras principais:

1- Módulos Ansible: Quase todos os módulos do Ansible (como `apt`, `yum`, `shell`, `docker_container`) são escritos em Python.

2- Scripts de Automação: Python é excelente para gerar inventários dinâmicos ou executar o próprio Ansible.

Para este dia, vamos focar em usar Python para chamar e orquestrar o Ansible (Método 2).

3. Usando `subprocess` para Rodar Playbooks
   
O método mais simples e robusto para usar o Ansible a partir do Python é chamar os comandos da CLI (`ansible-playbook`) usando a biblioteca padrão `subprocess`. Isso permite que seu script Python execute o playbook e capture o resultado.

Pré-requisitos:

1- Ansible instalado no seu sistema (pip install ansible).

2- Crie um arquivo de inventário e um playbook simples:

`inventory.ini`

````Ini, TOML

[webservers]
localhost ansible_connection=local
# Ou use um servidor real: 
# prod_server ansible_host=192.168.1.10
````
`simple_check.yml`

````YAML

---
- name: Verifica a conectividade basica
  hosts: webservers
  tasks:
    - name: Executa o comando 'uptime'
      command: uptime
      register: uptime_result
    
    - name: Exibe o resultado
      debug:
        msg: "Uptime em {{ inventory_hostname }}: {{ uptime_result.stdout }}"
````
Script Python (`run_ansible.py`):

````Python

import subprocess
import json

INVENTORY_FILE = "inventory.ini"
PLAYBOOK_FILE = "simple_check.yml"

def run_ansible_playbook(playbook, inventory):
    """Executa um playbook Ansible e retorna o status."""
    print(f"--- Executando Playbook: {playbook} ---")
    
    # Comando a ser executado
    command = [
        'ansible-playbook',
        '-i', inventory, # Inventário
        playbook        # Nome do playbook
    ]
    
    try:
        # Executa o comando e captura a saída
        result = subprocess.run(command, 
                                capture_output=True, 
                                text=True, 
                                check=True) # check=True levanta exceção em caso de erro

        print("Playbook executado com sucesso!")
        # print("Saída Padrão:\n", result.stdout)
        
    except subprocess.CalledProcessError as e:
        print(f"ERRO ao executar o playbook (Return Code: {e.returncode})")
        print("Erro Padrão:\n", e.stderr)
        return False
    except FileNotFoundError:
        print("ERRO: O comando 'ansible-playbook' não foi encontrado. Verifique sua instalação.")
        return False
        
    return True

# Chamada principal
if run_ansible_playbook(PLAYBOOK_FILE, INVENTORY_FILE):
    print("\nOrquestração Ansible concluída com sucesso pelo Python.")
````
4. Inventário Dinâmico com Python
   
Uma das integrações mais poderosas é usar Python para gerar o inventário em tempo de execução. Isso é crucial quando sua infraestrutura é gerenciada por serviços de Cloud (AWS, Azure, Google Cloud) ou sistemas de virtualização, onde os IPs mudam constantemente.

O Ansible pode receber um script Python como inventário, desde que ele retorne dados no formato JSON específico.

Script Python para Inventário (`dynamic_inventory.py`):

````Python

#!/usr/bin/env python3
# Este script deve ser executado pelo Ansible

import json
import sys

def get_inventory():
    """Simula a busca de servidores em um provedor de Cloud."""
    
    # Em um cenário real, você faria chamadas de API (Dia 15) para AWS, Azure, etc.
    # Exemplo: servers = boto3.client('ec2').describe_instances(...)
    
    inventory = {
        "_meta": {
            "hostvars": {
                # Dados específicos para um host (opcional)
                "db_server_01": {
                    "db_port": 5432
                }
            }
        },
        "databases": {
            "hosts": ["db_server_01", "db_server_02"],
            "vars": {
                "ansible_user": "ubuntu" 
            }
        },
        "load_balancers": {
            "hosts": ["lb_node_01"]
        }
    }
    
    # O Ansible espera o inventário completo na saída JSON
    print(json.dumps(inventory, indent=4))

if __name__ == '__main__':
    # Se o script é chamado com --list, ele retorna o inventário completo
    if len(sys.argv) == 2 and sys.argv[1] == '--list':
        get_inventory()
    # Para buscas específicas (opcional, mas recomendado)
    elif len(sys.argv) == 3 and sys.argv[1] == '--host':
        # Retorna dados do host específico, se necessário
        print(json.dumps({})) # Retorna um objeto vazio
    else:
        # Padrão para quando o Ansible chama sem argumentos
        get_inventory()
````
Como o Ansible usa isso:

1- Torne o script executável (`chmod +x dynamic_inventory.py`).

2- Execute o Ansible usando o script como inventário:

````Bash

ansible all -i dynamic_inventory.py --list-hosts
# Você verá a lista de hosts gerada pelo seu script Python!
````
Resumo do Dia 18

- Orquestração via subprocess: Você aprendeu a disparar playbooks e capturar o feedback usando a biblioteca padrão subprocess.

- Inventário Dinâmico: Entendeu o formato JSON necessário e a lógica por trás de um script Python para gerar um inventário em tempo de execução.

Essa integração permite que seus scripts Python tomem decisões de alto nível (por exemplo, "Se a métrica X cair, execute o playbook de recuperação Y") e usem o poder do Ansible para realizar a ação na infraestrutura.
