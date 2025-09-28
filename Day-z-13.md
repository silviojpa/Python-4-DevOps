# Day-13 | Consultas SQL com Python 🔍
No Dia 12, você aprendeu a estabelecer uma conexão com um banco de dados PostgreSQL usando a biblioteca psycopg2. No Dia 13, vamos aprofundar essa interação, focando na execução de consultas SQL para gerenciar e manipular dados dentro do banco de dados a partir do seu código Python.

A execução de consultas SQL é o coração da interação com bancos de dados relacionais e é uma habilidade crucial para a automação e tarefas de DevOps que envolvem gestão de dados.

Executando Consultas SQL para Gerenciamento de Dados
O fluxo básico para executar qualquer comando SQL no Python com psycopg2 é:

Conectar ao banco de dados.

Criar um objeto cursor.

Executar o comando SQL usando o método cursor.execute().

Se for um comando de modificação (como INSERT, UPDATE, DELETE), confirmar a transação (connection.commit()).

Se for um comando de consulta (SELECT), recuperar os resultados (com cursor.fetchone(), cursor.fetchall(), ou cursor.fetchmany()).

Fechar o cursor e a conexão.

Abaixo estão exemplos práticos de como usar comandos SQL essenciais no Python, partindo da estrutura de conexão que você já viu.

1. Preparação
Vamos usar a conexão e a tabela my_table que foi criada no Dia 12.

Python

import psycopg2

# Configurações de conexão (substitua pelos seus dados)
DB_CONFIG = {
    "database": "your_database_name",
    "user": "your_username",
    "password": "your_password",
    "host": "your_host",
    "port": "your_port"
}

def execute_query(query, params=None, fetch=False):
    """Função utilitária para conectar, executar a query e fechar."""
    conn = None
    try:
        # 1. Conectar ao BD
        conn = psycopg2.connect(**DB_CONFIG)
        # 2. Criar um objeto cursor
        cur = conn.cursor()

        # 3. Executar o comando SQL. Usamos 'params' para evitar SQL Injection.
        cur.execute(query, params)
        
        # 4. Confirmar transação (para INSERT, UPDATE, DELETE)
        if not fetch:
            conn.commit()
            print(f"Comando executado com sucesso: {query[:30]}...")
            return None
        
        # 5. Recuperar resultados (para SELECT)
        results = cur.fetchall()
        return results

    except (Exception, psycopg2.Error) as error:
        print(f"Erro ao interagir com o PostgreSQL: {error}")
        if conn:
            conn.rollback() # Desfaz as mudanças em caso de erro

    finally:
        # 6. Fechar cursor e conexão
        if conn:
            conn.close()
            # print("Conexão com PostgreSQL encerrada.")

2. Inserindo Dados com Segurança (INSERT)
Ao inserir dados, é crucial usar marcadores de posição (%s) e passar os valores separadamente. Isso previne ataques de SQL Injection.

Python

print("\n## Inserindo um novo usuário:")
insert_sql = "INSERT INTO my_table (name, age) VALUES (%s, %s);"
user_data = ('Alice Smith', 25)
execute_query(insert_sql, user_data) 
# Note: A função 'execute_query' chama conn.commit() para salvar a inserção.
3. Recuperando Dados (SELECT)
O comando SELECT requer a recuperação dos resultados após a execução, utilizando métodos do cursor.

A. Selecionar Todos os Registros (fetchall)
Python

print("\n## Selecionando todos os usuários:")
select_all_sql = "SELECT id, name, age FROM my_table;"
users = execute_query(select_all_sql, fetch=True)

if users:
    for row in users:
        print(f"ID: {row[0]}, Nome: {row[1]}, Idade: {row[2]}")
B. Selecionar Registros Específicos com Condição (WHERE)
Use o WHERE para filtrar e ainda use LIMIT para restringir o número de linhas, o que é útil em cenários de automação com grandes volumes de dados.

Python

print("\n## Selecionando usuário com ID = 1:")
select_specific_sql = "SELECT name, age FROM my_table WHERE id = %s;"
user_id_to_find = (1,) # Deve ser uma tupla, mesmo com um único elemento
specific_user = execute_query(select_specific_sql, user_id_to_find, fetch=True)

if specific_user:
    print(f"Usuário encontrado: {specific_user[0][0]} ({specific_user[0][1]} anos)")
else:
    print("Usuário não encontrado.")
4. Atualizando Dados (UPDATE)
O comando UPDATE é usado para modificar dados existentes. É fundamental usar a cláusula WHERE para especificar quais linhas devem ser atualizadas.

Python

print("\n## Atualizando a idade de Alice Smith (id = 2):")
update_sql = "UPDATE my_table SET age = %s WHERE name = %s;"
update_data = (26, 'Alice Smith') # Novo valor e condição
execute_query(update_sql, update_data)

# Verificar a atualização
print("\n## Verificando a atualização:")
select_updated_sql = "SELECT name, age FROM my_table WHERE name = 'Alice Smith';"
updated_user = execute_query(select_updated_sql, fetch=True)
if updated_user:
    print(f"Dados após update: {updated_user[0][0]} ({updated_user[0][1]} anos)")

5. Excluindo Dados (DELETE)
O comando DELETE remove linhas da tabela. Cuidado: um DELETE sem a cláusula WHERE apagará todas as linhas da tabela.

Python

print("\n## Excluindo o usuário John Doe:")
delete_sql = "DELETE FROM my_table WHERE name = %s;"
user_to_delete = ('John Doe',)
execute_query(delete_sql, user_to_delete)

# Verificar se a exclusão foi bem-sucedida
print("\n## Verificando registros restantes:")
select_all_sql = "SELECT COUNT(*) FROM my_table;"
count = execute_query(select_all_sql, fetch=True)
if count:
    print(f"Total de registros restantes: {count[0][0]}")

Resumo do Dia 13
Aprendeu a usar a função cursor.execute() para rodar comandos SQL no Python.

Dominou a importância de connection.commit() para comandos de modificação (INSERT, UPDATE, DELETE).

Entendeu a diferença entre comandos de modificação e de consulta, utilizando cursor.fetchall() para obter resultados de consultas (SELECT).

Praticou a prevenção de SQL Injection usando marcadores de posição (%s) e passando os valores como uma tupla separada.
