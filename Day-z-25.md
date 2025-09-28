Day-25 | CI/CD Best Practices 🚦
As Melhores Práticas de CI/CD garantem que seu pipeline seja seguro, rápido e confiável, transformando a automação em valor de negócio.

1. Testes: O Coração da Integração Contínua (CI)
A CI não é apenas construir código; é garantir que ele funcione antes de ser integrado ao tronco principal (main branch).

Shift-Left Testing (Testar Cedo): Integre testes o mais cedo possível no ciclo de desenvolvimento. Não espere a fase de QA.

Tipos de Testes no Pipeline:

Testes Unitários: O primeiro e mais rápido passo. Devem ser executados em cada commit. (Automatizado no Python, Dia 20).

Testes de Integração: Verificam se os componentes (ex: sua aplicação Python e o banco de dados PostgreSQL do Dia 12) funcionam juntos.

Testes de Contêiner: Garanta que a imagem Docker (Dia 16) não apenas builda, mas também funciona no ambiente isolado.

Cobertura de Código (Code Coverage): Defina um limite mínimo de cobertura (ex: 80%). Se o teste falhar, o pipeline para imediatamente (lógica fail-fast do Dia 20).

2. Segurança: Integrando o DevSecOps
A segurança deve ser automatizada e integrada em cada etapa do pipeline, não apenas no final.

Análise Estática de Código (SAST): Ferramentas que escaneiam seu código Python (antes de rodar) em busca de vulnerabilidades comuns ou erros de coding style.

Análise de Vulnerabilidade de Imagens: Escaneie suas imagens Docker (Dia 17) em busca de bibliotecas ou pacotes do sistema operacional com vulnerabilidades conhecidas (CVEs).

Gerenciamento de Segredos (Secrets): NUNCA armazene chaves, senhas ou tokens (Dias 12, 15, 24) diretamente no código ou no pipeline. Use ferramentas como HashiCorp Vault, AWS Secrets Manager ou Kubernetes Secrets (Dia 19).

3. Deployment: Práticas Avançadas de Entrega Contínua (CD)
O CD trata de como o código chega à produção de forma segura, minimizando o risco de inatividade (downtime).

Estratégia	Descrição	Vantagens	Automação Python
Rolling Deployment	Substitui gradualmente as réplicas antigas pelas novas (padrão no K8s).	Minimiza downtime.	patch_namespaced_deployment (Dia 19).
Blue/Green	Mantém dois ambientes idênticos (Blue = Antigo, Green = Novo). Após testes, o Load Balancer é alternado.	Zero downtime e fácil rollback.	Scripts Python para alterar a configuração do Load Balancer (Dia 24).
Canary Release	O novo código (Canário) é lançado para um pequeno subconjunto de usuários/servidores. Se as métricas do Prometheus (Dia 21) estiverem OK, o deployment continua.	Risco mínimo de exposição.	Python (Dia 23) pode monitorar métricas e controlar o scaling gradualmente.

Exportar para as Planilhas
Observabilidade: Depois de qualquer deployment, seu pipeline deve esperar alguns minutos e verificar se as métricas de saúde (Prometheus/CloudWatch) e logs de erro (ELK) (Dias 21-24) estão dentro dos limites aceitáveis.

4. Infraestrutura como Código (IaC)
A infraestrutura (servidores, redes, K8s) deve ser tratada como código.

Imutabilidade: NUNCA faça alterações manuais (ssh) em servidores de produção. Se uma mudança for necessária, atualize o código IaC (Terraform, Ansible no Dia 18) e re-execute o pipeline. Seus contêineres Docker (Dia 16) são o exemplo máximo de imutabilidade.

Versionamento: Armazene todo o código IaC no Git, versionado e sujeito a revisões (Pull Requests), assim como o código da sua aplicação Python.

Resumo do Dia 25
Qualidade: Testes Unitários e de Integração são a primeira linha de defesa.

Segurança: Segurança (scanning, secrets) deve ser integrada e automatizada (Shift-Left DevSecOps).

Risco: Estratégias como Canary e Blue/Green usam a observabilidade (Métricas e Logs) para mitigar o risco de downtime na produção.

Princípio Base: A consistência é alcançada tratando tudo como código versionado (código Python, Dockerfile, Manifestos K8s, Ansible Playbooks).

Exercício Final do Ciclo (Dia 21-25):

Exercício: Configure Prometheus para monitorar uma amostra de aplicação e visualize os dados no Grafana.

Você tem os seguintes passos:

Use o app_with_metrics.py (Dia 21).

Configure o Prometheus (prometheus.yml) para scrapear este script (Dia 21).

Instale o Grafana.

Configure uma Data Source no Grafana apontando para o Prometheus.

Crie um dashboard simples para visualizar o app_queue_size (Gauge) e o rate(app_http_requests_total[5m]) (a taxa de requisições nos últimos 5 minutos, usando PromQL).
