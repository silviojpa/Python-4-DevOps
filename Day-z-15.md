# Day-15 | Integrando Python com Serviços Web 📡
A integração com serviços web geralmente se dá através de requisições HTTP para APIs RESTful que retornam dados no formato JSON.

1. A Biblioteca Essencial: `requests`

Embora o Python tenha uma biblioteca nativa para HTTP (urllib), a comunidade DevOps prefere e utiliza massivamente a biblioteca de terceiros requests pela sua simplicidade e facilidade de uso.

* Se você ainda não a tem instalada, comece por aqui:

Bash
````
pip install requests
````
2. Realizando Requisições Básicas (CRUD via HTTP)
No REST, as operações são mapeadas para métodos HTTP. Vamos usar a API pública JSONPlaceholder para simular a interação com um serviço externo.
<img width="652" height="182" alt="image" src="https://github.com/user-attachments/assets/44885d72-9e5f-465f-a524-e5ae72012a87" />


A. READ: Buscar Dados (`GET`)
O método GET é o mais comum, usado para buscar informações.

Python
````
import requests

BASE_URL = "https://jsonplaceholder.typicode.com/posts"

# 1. Requisição GET simples
print("--- 1. Buscando uma lista de posts (GET) ---")
try:
    response = requests.get(BASE_URL)
    
    # 2. Verificar o status da resposta (200 = OK)
    if response.status_code == 200:
        # 3. Converter a resposta JSON para um objeto Python (lista de dicionários)
        data = response.json()
        
        print(f"Status Code: {response.status_code}")
        print(f"Total de itens retornados: {len(data)}")
        
        # Exibir o primeiro item
        if data:
            print("\nPrimeiro Post:")
            print(data[0])
    else:
        print(f"Falha na requisição. Status Code: {response.status_code}")

except requests.exceptions.RequestException as e:
    print(f"Erro ao conectar: {e}")
````
B. CREATE: Criar um Recurso (`POST`)
O método POST é usado para enviar novos dados ao servidor.

Python
````
print("\n--- 2. Criando um novo post (POST) ---")
new_post_data = {
    "title": "Automatizando com Python",
    "body": "Este post foi criado via script Python.",
    "userId": 101 
}

# O requests automaticamente converte o dicionário em JSON
response = requests.post(BASE_URL, json=new_post_data)

if response.status_code == 201: # 201 Created é o esperado para um POST bem-sucedido
    new_post = response.json()
    print(f"Status Code: {response.status_code}")
    print(f"Post criado com sucesso. ID retornado: {new_post.get('id')}")
else:
    print(f"Falha ao criar post. Status Code: {response.status_code}")
````
C. UPDATE: Atualizar um Recurso (`PUT`/`PATCH`)
Vamos atualizar o post recém-criado (assumindo que o ID seja, por exemplo, 101, já que o JSONPlaceholder simula a criação).

Python
````
# Vamos tentar atualizar o post de ID 1 (exemplo)
POST_ID = 1 
UPDATE_URL = f"{BASE_URL}/{POST_ID}"

print(f"\n--- 3. Atualizando o post de ID {POST_ID} (PUT) ---")
updated_data = {
    "title": "Título Atualizado pelo Python",
    "body": "Conteúdo modificado em 15/09.",
    "userId": 1
}

# PUT substitui o recurso inteiro
response = requests.put(UPDATE_URL, json=updated_data) 

if response.status_code == 200:
    print(f"Status Code: {response.status_code}")
    print("Post atualizado com sucesso.")
else:
    print(f"Falha ao atualizar post. Status Code: {response.status_code}")
````
3. Usando Headers e Autenticação
Em cenários reais de DevOps, você quase sempre precisará enviar headers (cabeçalhos) para informar o tipo de conteúdo ou fornecer tokens de autenticação (chaves API, tokens JWT).

Python
````
print("\n--- 4. Requisição com Headers (Simulando Autenticação) ---")

# Simulando um token de autenticação
AUTH_TOKEN = "seu_token_jwt_aqui_12345" 

# Definindo cabeçalhos
custom_headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {AUTH_TOKEN}", # Padrão comum de autenticação
    "X-Request-Source": "Python-DevOps-Script" 
}

# Tentativa de GET em um endpoint que exigiria autenticação (usando o mesmo endpoint de exemplo)
response = requests.get(BASE_URL, headers=custom_headers)

print(f"Status Code retornado (com headers): {response.status_code}")
# Em um sistema real, um 401 (Não Autorizado) seria esperado se o token fosse inválido.
````
Resumo do Dia 15
- Ferramenta-Chave: Você aprendeu a usar a biblioteca `requests` para interagir com serviços web.

- Mapeamento REST: Entendeu como os métodos Python (requests.get, requests.post, etc.) mapeiam para as operações CRUD em APIs RESTful.

- JSON: Praticou a conversão de dados JSON da resposta para objetos Python (response.json()) e vice-versa (enviando dicionários via json=...).

- Status Codes: Aprendeu a verificar os HTTP Status Codes (200, 201, etc.) para determinar o sucesso da operação.

- Headers: Viu como incluir cabeçalhos personalizados, o que é essencial para autenticação em ambientes de produção.

Exercício do Ciclo (Dia 11-15):

Exercício: Construa um script Python que interage com um banco de dados para gerenciar informações do usuário.

Você já tem todas as peças:

1- Use o conhecimento de SQLAlchemy (Dia 14) para modelar um banco de dados de usuários.

2- Use o conhecimento de requests (Dia 15) para buscar dados de uma API externa (como uma lista de funcionários, dados de clima, etc.).

3- Use a Sessão do SQLAlchemy para inserir esses dados buscados no seu banco de dados local.
