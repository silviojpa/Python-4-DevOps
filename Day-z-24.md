# Day-24 | Integração com Serviços Cloud ☁️

A automação e gestão de infraestrutura na nuvem (CloudOps) são realizadas através dos SDKs (Software Development Kits) oficiais. Usaremos o AWS SDK (Boto3) como principal exemplo, já que os conceitos se aplicam a todos os outros SDKs (Azure ou Google Cloud).

1. A Ferramenta Essencial: Boto3 (AWS)
O Boto3 é a biblioteca oficial da AWS para Python. Ele permite que você interaja programaticamente com praticamente todos os serviços AWS, como EC2, S3, Lambda, e RDS.

Instalação e Autenticação
Para instalar, use pip:

Bash
````
pip install boto3
````
Para o Boto3 se conectar à sua conta, ele precisa de credenciais. As formas mais comuns e seguras de configurá-las são:

1- AWS CLI (aws configure): Configura um perfil local de credenciais.

2- Variáveis de Ambiente: Exportar `AWS_ACCESS_KEY_ID` e `AWS_SECRET_ACCESS_KEY`.

Perfis de IAM: A forma mais segura para produção, onde a infraestrutura AWS gerencia o acesso (ex: Lambda ou EC2 com um Role anexado).

2. Conceitos-Chave do Boto3: Clients vs. Resources
O Boto3 oferece duas interfaces principais para a interação com os serviços:

<img width="882" height="181" alt="image" src="https://github.com/user-attachments/assets/464e1139-b5ed-4035-a893-09cadf04b7bf" />

Para automação e controle detalhado, o acesso via Clients é o mais recomendado.

3. Exemplo Prático: Gerenciando Recursos AWS

A. Provisionamento e Desprovisionamento de Storage (S3)
Este exemplo demonstra o ciclo de vida completo de um recurso de storage (Bucket S3), incluindo a etapa crítica de limpeza (excluir objetos) antes de deletar o bucket.

Python
````
import boto3
from botocore.exceptions import ClientError
import time

REGION = 'us-east-1'
BUCKET_NAME = 'python-devops-bucket-2025' # Deve ser globalmente único

# Cria o Client de S3 (acesso de baixo nível)
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
        print(f"    SUCESSO: Bucket '{bucket_name}' criado.")
    except ClientError as e:
        if e.response['Error']['Code'] == 'BucketAlreadyOwnedByYou':
            print(f"    AVISO: Bucket '{bucket_name}' já existe e pertence a você.")
        else:
            print(f"    ERRO ao criar bucket: {e}")
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
            print("    Objetos internos deletados.")
            
        # 2. Deleta o bucket
        s3_client.delete_bucket(Bucket=bucket_name)
        print("    SUCESSO: Bucket deletado.")

    except ClientError as e:
        print(f"    ERRO ao deletar bucket (pode não existir): {e}")


# --- Orquestração Principal ---
if __name__ == "__main__":
    if create_s3_bucket(BUCKET_NAME):
        # Simula o uso do recurso por um tempo
        print("\nRecurso em uso... (Aguardando 5 segundos)")
        time.sleep(5)
        clean_and_delete_s3_bucket(BUCKET_NAME)
````
B. Monitoramento e Ações de CloudOps (EC2/CloudWatch)
O Python também é o motor de decisão para operações de nuvem, integrando-se ao CloudWatch para monitoramento e ao EC2 para ações de gestão (como Scaling ou economia de custos).

Python
````
# Reutilizando o client AWS (EC2 e CloudWatch)
ec2_client = boto3.client('ec2', region_name=REGION)
cw_client = boto3.client('cloudwatch', region_name=REGION)

# Exemplo de consulta de métricas
def get_cpu_metric_from_cloudwatch(instance_id):
    """Busca a métrica de CPU no CloudWatch (base para Auto-Scaling)."""
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
    # response['MetricDataResults'] conteria os dados de CPU.
    print("\n[Consulta CloudWatch] -> Dados brutos recebidos. O Python os processaria para Auto-Scaling.")


# Exemplo de Ação de Gestão (Iniciar/Parar VM)
def stop_ec2_instance(instance_id):
    """Para uma instância EC2 (economia de custos)."""
    print(f"\n[Automação EC2] -> Tentando parar instância {instance_id}")
    try:
        ec2_client.stop_instances(InstanceIds=[instance_id])
        print("    Comando de parada enviado. Verifica o status em seguida.")
    except ClientError as e:
        print(f"    ERRO ao parar instância: {e}")
````
4. Integração com Outras Clouds
Os conceitos de usar um SDK para chamar a API são universais. Para outras clouds, você apenas troca o SDK:
<img width="752" height="141" alt="image" src="https://github.com/user-attachments/assets/ee1fa658-54b1-47ca-8c24-809953395643" />

Em todos os casos, o Python atua como o código que chama a função API do SDK (ex: s3_client.create_bucket ou blob_service_client.create_container).

Resumo do Dia 24
- SDKs Cloud: Automação na Nuvem exige o uso de SDKs oficiais (ex: Boto3).

- Clients vs. Resources: Você tem a escolha entre o controle granular de Clients e a facilidade de uso de Resources.

- Provisionamento Programático: Usamos Python para orquestrar o ciclo de vida completo de recursos (criação, limpeza e exclusão de um bucket S3).

- CloudOps: Python é o motor que integra monitoramento (CloudWatch) e ações de gestão (EC2) para criar lógicas de Auto-Scaling customizadas.
