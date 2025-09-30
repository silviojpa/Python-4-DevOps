# Day-28 | Estudos de Caso e Lições Aprendidas 🌐

1. Netflix: A Cultura de Desapego e o Caos
   
A Netflix é o exemplo clássico de engenharia de confiabilidade e scaling em Cloud.

- Problema: Garantir que o serviço permaneça disponível mesmo com a falha de qualquer componente da AWS (Dia 24).

- Solução: O Chaos Monkey (e o Chaos Engineering mais amplo). Chaos Monkey é uma ferramenta que desliga aleatoriamente instâncias de produção (VMs/contêineres) durante o horário comercial.

- Lição DevOps: Se a equipe de Dev (Dia 26) sabe que o Chaos Monkey pode "matar" seu serviço a qualquer momento, eles são forçados a construir resiliência (microsserviços, auto-scaling no Dia 23, health checks robustos) desde o início. O medo da falha é substituído pela confiança na resiliência do sistema.

- Conexão Python: Ferramentas de Chaos Engineering são frequentemente scripts Python ou serviços que utilizam o Boto3 (Dia 24) para interagir com a API da AWS e desprovisionar recursos programaticamente.

2. Google: O Pilar da Engenharia de Confiabilidade (SRE)
   
O Google é o criador da disciplina SRE (Site Reliability Engineering), que é a aplicação dos princípios de software para as tarefas de Operações (Ops). O SRE define como as equipes devem trabalhar para garantir a confiabilidade.

- Problema: Manter serviços globais como Busca e Gmail com uma disponibilidade próxima de 100% enquanto se mantém uma alta taxa de inovação.

- Solução: Uso rigoroso de SLOs, SLIs e Orçamentos de Erro (Dia 27). Se o Orçamento de Erro de uma equipe for esgotado, eles param de lançar novos recursos (CI/CD) e se dedicam exclusivamente ao trabalho de confiabilidade.

- Lição DevOps: O SRE fornece o framework de medição que permite à equipe Dev (feature teams) e à equipe Ops (SREs) falar a mesma língua. Ele transforma a confiabilidade de uma meta subjetiva em um dado mensurável.

- Conexão Python: A maioria das ferramentas internas de automação e monitoramento do Google é escrita em Python. O Python é usado para coletar, processar e aplicar lógica aos SLOs, garantindo que as regras de deployment sejam aplicadas de forma consistente.

3. Spotify: Feature Flags e Deployments Graduais
   
O Spotify popularizou a ideia de Small Batches (pequenos lotes) e decoupling (desacoplamento) no deployment.

- Problema: Lançar novas funcionalidades para milhões de usuários em várias plataformas (mobile, desktop) com risco zero.

- Solução:

  1-Microsserviços e Kubernetes: Desacoplar o aplicativo em centenas de serviços (gerenciados por K8s, Dia 19).

  2-Feature Flags (Alternância de Recursos): A nova funcionalidade é implantada no código de produção desligada. Ela é ligada apenas para um subconjunto de usuários/países, permitindo Canary Releases e testes A/B instantâneos (Dia 25).

- Lição DevOps: O deployment deve ser separado do lançamento do recurso. Isso permite um rollback instantâneo (flip the switch) se algo der errado, em vez de exigir um rollback do código inteiro.

- Conexão Python: O Python é frequentemente usado para construir os serviços de API que gerenciam essas Feature Flags, decidindo, com base na lógica (Dia 15), se um usuário verá o recurso novo ou antigo.

Lições Chave Consolidadas para a Sua Trilha
<img width="645" height="214" alt="image" src="https://github.com/user-attachments/assets/223952cf-b84c-473f-bd66-8b9189e6eea2" />

Com o conhecimento destes estudos de caso, você agora tem a perspectiva completa: não apenas como fazer (as ferramentas), mas por que fazer (a cultura) e o que acontece quando se faz bem (a escala global).
