# Day-14 | ORM com SQLAlchemy 🐍
O que é ORM?
ORM (Object-Relational Mapping) é uma técnica de programação que converte dados entre sistemas de tipos incompatíveis (como a classe User no Python) e bancos de dados relacionais (como uma tabela users no PostgreSQL).

Em vez de escrever código SQL como:
INSERT INTO users (name, age) VALUES ('Maria', 35);

Você escreverá código Python como:
session.add(User(name='Maria', age=35))

O SQLAlchemy se encarrega de traduzir o código Python para o SQL apropriado.

1. Instalação e Configuração
Você precisará do SQLAlchemy e do driver de PostgreSQL que já estava usando (psycopg2-binary é uma alternativa comum se o psycopg2 puro tiver problemas de compilação, mas se o psycopg2 do Dia 12 funcionou, mantenha-o, ou use o binário para simplificar):

Bash

pip install sqlalchemy
# Se precisar, use: pip install psycopg2-binary
2. Elementos Principais do SQLAlchemy
O SQLAlchemy é dividido em três partes principais para esta abordagem:

Engine (Motor): O ponto de partida. Ele estabelece a conexão com o banco de dados.

Base (Declaração Base): Usada para definir as classes do seu modelo (as tabelas).

Session (Sessão): O objeto de conversação. É a interface primária para interagir com o banco de dados (o seu "tradutor" de Python para SQL).

3. Criando a Conexão e o Modelo de Dados
Vamos replicar a tabela my_table dos dias anteriores usando um modelo Python:

Python

from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.orm import sessionmaker, declarative_base

# 1. Configuração da Conexão (Engine)
# Formato do DSN (Data Source Name) para PostgreSQL:
# postgresql://<user>:<password>@<host>:<port>/<database>
# Substitua os placeholders com as suas credenciais
DB_URL = "postgresql://user:password@host:5432/your_database_name" 

# Cria o Engine (motor de conexão)
engine = create_engine(DB_URL, echo=True) # 'echo=True' mostra o SQL gerado no console

# 2. Base Declarativa
# A classe Base irá mapear as classes Python para tabelas do banco de dados
Base = declarative_base()

# 3. Definição do Modelo de Dados (ORM Class)
class User(Base):
    # Mapeamento para o nome da tabela no BD
    __tablename__ = 'my_table' 
    
    # Definição das colunas da tabela
    id = Column(Integer, primary_key=True)
    name = Column(String(100))
    age = Column(Integer)

    def __init__(self, name, age):
        self.name = name
        self.age = age

    def __repr__(self):
        return f"<User(id={self.id}, name='{self.name}', age={self.age})>"

# Criação das Tabelas
# Este comando verifica a Base e cria as tabelas que ainda não existem
Base.metadata.create_all(engine)

# 4. Configuração da Sessão
# Cria uma classe de Sessão que será usada para interagir com o BD
Session = sessionmaker(bind=engine)
4. Operações CRUD com ORM
Agora, com o modelo User e a Session prontos, podemos realizar operações CRUD (Create, Read, Update, Delete) de forma totalmente orientada a objetos.

Python

# Cria uma instância de sessão para a transação
session = Session()

# --- 1. CREATE (Criar/Inserir) ---

# Cria objetos Python
user1 = User(name='Maria Silva', age=35)
user2 = User(name='Carlos Souza', age=28)

# Adiciona os objetos à sessão (prepara a transação)
session.add(user1)
session.add(user2)

# Commit: Envia os comandos INSERT para o banco de dados e salva
session.commit()
print("Usuários inseridos com sucesso!")


# --- 2. READ (Ler/Consultar) ---

print("\n--- Consultando todos os usuários ---")
# Query: SELECT * FROM my_table;
all_users = session.query(User).all()

for user in all_users:
    print(user) # Usa o método __repr__ definido na classe

print("\n--- Consultando usuário específico com filtro ---")
# Query: SELECT * FROM my_table WHERE name = 'Maria Silva';
maria = session.query(User).filter(User.name == 'Maria Silva').first()

if maria:
    print(f"Usuário encontrado: {maria.name}, ID: {maria.id}")


# --- 3. UPDATE (Atualizar) ---

print("\n--- Atualizando a idade de Carlos Souza ---")
# Busca o objeto (pode usar 'filter' também)
carlos = session.query(User).filter_by(name='Carlos Souza').one() 

# Modifica o atributo do objeto Python
carlos.age = 29

# O SQLAlchemy detecta a mudança. Basta commitar
session.commit()

print(f"Idade de {carlos.name} atualizada para {carlos.age}.")


# --- 4. DELETE (Deletar) ---

print("\n--- Deletando Maria Silva ---")
# Busca o objeto
user_to_delete = session.query(User).filter(User.name == 'Maria Silva').first()

if user_to_delete:
    # Deleta o objeto da sessão
    session.delete(user_to_delete)
    # Commit: Envia o comando DELETE para o BD
    session.commit()
    print("Usuário deletado com sucesso.")

# Verifica se a exclusão funcionou
remaining_users = session.query(User).all()
print(f"Total de usuários restantes: {len(remaining_users)}")

# Fecha a sessão
session.close()
Resumo do Dia 14
ORM e SQLAlchemy: Você começou a usar um ORM para interagir com o banco de dados usando objetos Python, o que é mais intuitivo e menos propenso a erros de sintaxe SQL.

Componentes-Chave: Dominou o uso do Engine (conexão), Base (modelagem) e Session (interação/transações).

Modelagem Declarativa: Aprendeu a mapear classes Python (User) para tabelas de banco de dados (__tablename__) usando Column para definir os atributos.

CRUD Pythonico: Realizou as operações básicas de banco de dados (CREATE, READ, UPDATE, DELETE) usando métodos do objeto session.query(User) sem escrever uma linha de SQL.
