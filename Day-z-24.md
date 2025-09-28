# Day-24 | Integração com Serviços Cloud ☁️
A integração se dá através dos SDKs (Software Development Kits) oficiais que cada provedor de nuvem oferece para Python. Vamos usar a AWS (Amazon Web Services) e seu SDK, o Boto3, como exemplo principal, pois os conceitos se aplicam a todos os outros (Azure SDK ou Google Cloud SDK).

1. A Ferramenta Essencial: Boto3 (AWS)
O Boto3 é o SDK oficial da AWS para Python. Ele permite que você use o Python para interagir com centenas de serviços da AWS, como EC2, S3, RDS, Lambda e muito mais.

Instalação:

Bash

pip install boto3
Pré-requisitos de Autenticação:
Para o Boto3 funcionar, você precisa configurar suas credenciais AWS. A maneira mais comum e segura é:

Instalar e configurar o AWS CLI (aws configure).

Ou configurar as variáveis de ambiente: AWS_ACCESS_KEY_ID e AWS_SECRET_ACCESS_KEY.

2. Conceitos-Chave do Boto3
O Boto3 oferece dois tipos principais de acesso:

Clients (Clientes): Acesso de baixo nível, mapeando 1:1 para a API de serviços da AWS (mais rápido e detalhado).

Resources (Recursos): Abstrações de alto nível orientadas a objetos (mais fáceis de usar).

Para automação DevOps, os Clients são frequentemente preferidos por oferecerem controle granular.

3. Exemplo Prático: Gerenciando Recursos AWS (S3 e EC2)
A. Provisionamento e Desprovisionamento de Storage (S3)
Vamos criar e, em seguida, remover um bucket S3, uma tarefa comum de provisionamento de recursos.

Python

import boto3
from botocore.exceptions import ClientError
import time

REGION = 'us-east-1'
BUCKET_NAME = 'python-devops-bucket-2025' # Deve ser globalmente único

# Cria o Client de S3
s3_client = boto3.client('s3', region_name=REGION)

def create_s3_bucket(bucket_name):
    """Cria um bucket S3."""
    print(f"\n[PASSO 1] -> Tentando criar bucket: {bucket_name}")
    try:
        if REGION == 'us-east-1':
            s3_client.create_bucket(Bucket=bucket_name)
        else:
            s3_client.create_bucket(
                Bucket=bucket_name,
                CreateBucketConfiguration={'LocationConstraint': REGION}
            )
        print(f"   SUCESSO: Bucket '{bucket_name}' criado.")
    except ClientError as e:
        if e.response['Error']['Code'] == 'BucketAlreadyOwnedByYou':
            print(f"   AVISO: Bucket '{bucket_name}' já existe e pertence a você.")
        else:
            print(f"   ERRO ao criar bucket: {e}")
            return False
    return True

def clean_and_delete_s3_bucket(bucket_name):
    """Esvazia e deleta o bucket S3 (Automação de limpeza de recursos)."""
    print(f"\n[PASSO 2] -> Esvaziando e deletando bucket: {bucket_name}")
    try:
        # 1. Lista e deleta todos os objetos (o bucket deve estar vazio para ser deletado)
        response = s3_client.list_objects_v2(Bucket=bucket_name)
        if 'Contents' in response:
            objects_to_delete = [{'Key': obj['Key']} for obj in response['Contents']]
            s3_client.delete_objects(
                Bucket=bucket_name,
                Delete={'Objects': objects_to_delete}
            )
            print("   Objetos internos deletados.")
        
        # 2. Deleta o bucket
        s3_client.delete_bucket(Bucket=bucket_name)
        print("   SUCESSO: Bucket deletado.")

    except ClientError as e:
        print(f"   ERRO ao deletar bucket (pode não existir): {e}")


# --- Orquestração Principal ---
if __name__ == "__main__":
    if create_s3_bucket(BUCKET_NAME):
        # Simula o uso do recurso por um tempo
        print("\nRecurso em uso... (Aguardando 5 segundos)")
        time.sleep(5)
        clean_and_delete_s3_bucket(BUCKET_NAME)
B. Monitoramento e Automação de Scaling (EC2/CloudWatch)
O Python pode atuar como um dashboard customizado ou como o motor de decisão para o scaling (reutilizando a lógica do Dia 23).

Python

# Reutilizando o client AWS (EC2 e CloudWatch)
ec2_client = boto3.client('ec2', region_name=REGION)
cw_client = boto3.client('cloudwatch', region_name=REGION)

# Exemplo de consulta de métricas (Dia 21)
def get_cpu_metric_from_cloudwatch(instance_id):
    """Busca a métrica de CPU no CloudWatch."""
    response = cw_client.get_metric_data(
        MetricDataQueries=[
            {
                'Id': 'm1',
                'MetricStat': {
                    'Metric': {
                        'Namespace': 'AWS/EC2',
                        'MetricName': 'CPUUtilization',
                        'Dimensions': [{'Name': 'InstanceId', 'Value': instance_id}]
                    },
                    'Period': 60, # Média a cada 60 segundos
                    'Stat': 'Average'
                },
                'ReturnData': True
            },
        ],
        StartTime=int(time.time()) - 300, # 5 minutos atrás
        EndTime=int(time.time())
    )
    # Em um script real, você processaria response['MetricDataResults']
    print("\n[Consulta CloudWatch] -> Dados brutos recebidos. O Python os processaria para Auto-Scaling.")


# Exemplo de Ação de Gestão (Iniciar/Parar VM)
def stop_ec2_instance(instance_id):
    """Para uma instância EC2 (economia de custos)."""
    print(f"\n[Automação EC2] -> Tentando parar instância {instance_id}")
    try:
        ec2_client.stop_instances(InstanceIds=[instance_id])
        print("   Comando de parada enviado. Verifica o status em seguida.")
    except ClientError as e:
        print(f"   ERRO ao parar instância: {e}")

# Exemplo de uso:
# stop_ec2_instance('i-0abcdef1234567890') 
# get_cpu_metric_from_cloudwatch('i-0abcdef1234567890')
4. Integração com Outras Clouds
Os conceitos são idênticos, apenas o SDK e os nomes dos serviços mudam:

Provedor	SDK Python	Serviço de Storage (exemplo S3)
Microsoft Azure	Azure SDK para Python	azure-storage-blob
Google Cloud (GCP)	Google Cloud Client Libraries	google-cloud-storage

Exportar para as Planilhas
Em todos os casos, o Python atua como o código que chama a função API do SDK (ex: s3_client.create_bucket ou blob_service_client.create_container).

Resumo do Dia 24
SDKs Cloud: Você aprendeu que a automação em Nuvem exige o uso de SDKs oficiais (ex: Boto3 para AWS).

Clients vs. Resources: Entendeu as duas formas de interagir com a API (baixo nível Client vs. alto nível Resource).

Provisionamento Programático: Usou Python para orquestrar o ciclo de vida de recursos de infraestrutura (criação, limpeza e exclusão de um bucket S3).

CloudOps: Viu como Python se integra com serviços de monitoramento (CloudWatch) para construir lógica de Auto-Scaling customizada.
