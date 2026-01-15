# Erros Comuns em AWS IAM
## Identificação, Impacto e Correção

---

## Introdução

Este documento compila os erros mais frequentes encontrados em configurações de **AWS Identity and Access Management (IAM)** em ambientes corporativos reais. Cada erro é analisado em profundidade, explicando por que representa um risco, como identificá-lo e como corrigi-lo seguindo as melhores práticas de segurança. A compreensão destes erros é fundamental para profissionais de cloud security, desenvolvedores e administradores de sistemas que trabalham com AWS.

Os erros documentados aqui foram identificados através de análises de segurança em múltiplos ambientes, relatórios de incidentes públicos, e recomendações de frameworks de segurança como CIS AWS Foundations Benchmark e AWS Well-Architected Framework. Muitos destes erros são responsáveis por violações de dados de alto perfil e incidentes de segurança que resultaram em perdas financeiras significativas.

---

## Erro 1: Uso Excessivo de Wildcards (*) em Políticas

### Descrição do Problema

O uso indiscriminado de wildcards (`*`) em campos `Action` e `Resource` de políticas IAM é provavelmente o erro mais comum e perigoso em configurações AWS. Uma política com `"Action": "*"` e `"Resource": "*"` concede permissões administrativas completas, permitindo qualquer operação em qualquer recurso da conta AWS. Este padrão é frequentemente utilizado por conveniência durante desenvolvimento ou por desconhecimento de suas implicações de segurança.

### Por Que É Perigoso

Políticas com wildcards violam fundamentalmente o princípio do menor privilégio e criam uma superfície de ataque extremamente ampla. Um usuário com estas permissões pode executar operações destrutivas como exclusão de buckets S3, terminação de instâncias EC2, modificação de configurações de segurança, criação de novos usuários administrativos, desabilitação de logs de auditoria e exfiltração de dados sensíveis. Em caso de comprometimento de credenciais, um atacante teria controle total sobre a infraestrutura cloud da organização.

Além do risco de segurança, o uso de wildcards dificulta auditorias e demonstração de conformidade com regulamentações. Auditores não conseguem determinar quais permissões específicas foram concedidas e se estas são apropriadas para as funções dos usuários. Frameworks de compliance como PCI-DSS e HIPAA exigem explicitamente controles de acesso granulares, que são impossíveis de demonstrar com políticas baseadas em wildcards.

### Como Identificar

A identificação de políticas com wildcards pode ser realizada através de múltiplas abordagens. Utilize o **IAM Policy Simulator** para analisar políticas anexadas a usuários, grupos e roles, buscando por `"Action": "*"` ou `"Resource": "*"`. O **AWS IAM Access Analyzer** também identifica políticas excessivamente permissivas e fornece recomendações de refinamento.

Para análise programática, utilize AWS CLI para extrair todas as políticas e analisá-las com ferramentas como `jq` ou scripts Python. Ferramentas de auditoria de segurança como **Prowler** e **ScoutSuite** automatizam esta verificação e geram relatórios detalhados de políticas problemáticas.

Revise regularmente o **IAM Credential Report** para identificar usuários com políticas administrativas anexadas. Políticas AWS gerenciadas como `AdministratorAccess` e `PowerUserAccess` também utilizam wildcards e devem ser utilizadas com extrema cautela.

### Como Corrigir

A correção requer substituição de políticas genéricas por políticas granulares e específicas. Analise as necessidades reais de acesso de cada usuário ou serviço através de entrevistas com stakeholders e análise de logs do CloudTrail para identificar quais ações são realmente utilizadas. Crie políticas que especificam exatamente quais ações são permitidas em quais recursos.

Por exemplo, ao invés de `"Action": "*"`, especifique ações individuais: `"Action": ["s3:GetObject", "s3:PutObject", "s3:ListBucket"]`. Ao invés de `"Resource": "*"`, especifique ARNs completos: `"Resource": "arn:aws:s3:::my-specific-bucket/*"`. Utilize condições IAM para adicionar restrições contextuais como IP de origem, tags de recursos e exigência de MFA.

Para casos legítimos que requerem acesso administrativo (como administradores de sistemas), implemente controles compensatórios rigorosos: MFA obrigatório, restrições por IP, limitação de horário de acesso, uso de roles com assume role policies restritivas ao invés de permissões diretas, e monitoramento contínuo com alertas para uso de permissões administrativas.

Consulte o arquivo `developer-least-privilege.json` neste repositório para exemplo concreto de política granular que substitui permissões administrativas.

---

## Erro 2: Ausência de Autenticação Multi-Fator (MFA)

### Descrição do Problema

A autenticação multi-fator não é habilitada por padrão em contas AWS e muitas organizações não a implementam obrigatoriamente. Usuários podem acessar recursos críticos utilizando apenas senha, que pode ser comprometida através de phishing, credential stuffing, keyloggers, vazamentos de dados ou engenharia social.

### Por Que É Perigoso

Senhas são o elo mais fraco em segurança de identidades. Estudos indicam que mais de 80% das violações de dados envolvem credenciais comprometidas ou fracas. Campanhas de phishing são cada vez mais sofisticadas e efetivas, e vazamentos de credenciais em breaches de terceiros são comuns. Uma vez que um atacante obtém uma senha, a ausência de MFA significa que não há barreira adicional para acesso não autorizado.

O risco é amplificado quando combinado com permissões excessivas. Um atacante que compromete credenciais de um usuário com permissões administrativas e sem MFA tem acesso irrestrito à infraestrutura. Mesmo para usuários com permissões limitadas, a ausência de MFA facilita significativamente ataques de credential stuffing, onde atacantes testam credenciais vazadas de outros serviços.

### Como Identificar

O **IAM Credential Report** fornece visão consolidada de todos os usuários e indica quais possuem MFA habilitado. Este relatório pode ser gerado via console AWS ou programaticamente via CLI com comando `aws iam generate-credential-report` seguido de `aws iam get-credential-report`. Analise a coluna `mfa_active` para identificar usuários sem MFA.

O **AWS Security Hub** inclui controles de conformidade que verificam automaticamente se MFA está habilitado para usuários, especialmente para a conta root e usuários com acesso à console. Ferramentas como **Prowler** também automatizam esta verificação e geram alertas para usuários sem MFA.

Revise políticas IAM para verificar se há condições que exigem MFA para operações sensíveis (`aws:MultiFactorAuthPresent`). A ausência destas condições indica que operações críticas podem ser realizadas sem MFA mesmo que o usuário o tenha configurado.

### Como Corrigir

Implemente MFA obrigatório para todos os usuários através de política organizacional comunicada claramente. Crie política IAM que negue todas as operações exceto habilitação de MFA para usuários que ainda não o configuraram, forçando-os a habilitar MFA antes de acessar recursos.

Adicione condições `"aws:MultiFactorAuthPresent": "true"` em todas as políticas que concedem permissões sensíveis, garantindo que mesmo usuários com MFA configurado só possam realizar operações críticas após autenticação multi-fator. Implemente MFA para acesso programático (API) através de tokens de sessão temporários obtidos via `aws sts get-session-token` com código MFA.

Para administradores e usuários com acesso a produção, considere hardware MFA tokens (YubiKey, AWS MFA devices) ao invés de aplicativos de smartphone, pois oferecem segurança superior contra phishing e malware. Estabeleça processo de onboarding que inclua configuração obrigatória de MFA antes de concessão de acessos.

Configure alertas no CloudWatch para logins sem MFA e implemente processo automatizado de desabilitação de usuários que não configurarem MFA em prazo definido (por exemplo, 7 dias). Documente procedimentos de recuperação de acesso em caso de perda de dispositivo MFA.

---

## Erro 3: Compartilhamento de Credenciais entre Usuários

### Descrição do Problema

Organizações frequentemente criam contas genéricas compartilhadas por múltiplos usuários, como "developer-shared", "team-access" ou "deployment-user". Access keys de longa duração são compartilhadas via email, chat ou documentos, e múltiplas pessoas utilizam as mesmas credenciais para acessar recursos AWS.

### Por Que É Perigoso

O compartilhamento de credenciais impossibilita a atribuição precisa de ações a indivíduos específicos, comprometendo completamente a auditoria e a não-repudiação. Em caso de incidente de segurança, torna-se impossível determinar quem realizou ações maliciosas ou erradas. Não há accountability individual, o que reduz incentivos para comportamento seguro.

A revogação de acesso torna-se problemática quando um membro da equipe deixa a organização, pois múltiplas pessoas conhecem as credenciais. Trocar a senha ou rotacionar access keys requer coordenação com todos os usuários, processo frequentemente negligenciado por conveniência. Isto resulta em ex-funcionários mantendo acesso não autorizado.

O risco de exposição acidental aumenta proporcionalmente ao número de pessoas que conhecem as credenciais. Credenciais compartilhadas são frequentemente armazenadas de forma insegura em documentos, wikis, repositórios de código ou mensagens de chat. Quanto mais pessoas têm acesso, maior a probabilidade de vazamento.

### Como Identificar

Análise de logs do CloudTrail pode revelar padrões suspeitos de uso de credenciais. Mesmas credenciais sendo utilizadas simultaneamente de endereços IP geograficamente distantes indicam compartilhamento. Mudanças bruscas em padrões de uso (horários, localizações, serviços acessados) também sugerem múltiplos usuários.

O **IAM Credential Report** indica access keys com mais de 90 dias sem rotação, que frequentemente são credenciais compartilhadas ou esquecidas. Revise nomes de usuários IAM buscando por padrões genéricos como "shared", "team", "common", "service" que sugerem contas compartilhadas.

Entrevistas com equipes técnicas frequentemente revelam práticas de compartilhamento de credenciais. Pergunte diretamente como acessos são gerenciados e se há contas compartilhadas. Revise documentação interna, wikis e repositórios de código buscando por credenciais armazenadas.

### Como Corrigir

Elimine todas as contas compartilhadas imediatamente, criando usuários individuais para cada pessoa. Cada usuário deve ter credenciais únicas e pessoais. Implemente política de rotação obrigatória de access keys a cada 90 dias, com alertas automatizados para keys próximas da expiração.

Prefira uso de **IAM roles** com credenciais temporárias ao invés de access keys de longa duração. Para aplicações rodando em EC2, anexe roles diretamente às instâncias. Para containers ECS, utilize task roles. Para funções Lambda, anexe execution roles. Credenciais temporárias são automaticamente rotacionadas e não podem ser compartilhadas efetivamente.

Implemente **AWS IAM Identity Center** (sucessor do AWS SSO) para autenticação federada com provedor de identidade corporativo (Active Directory, Okta, Azure AD). Isto centraliza gestão de identidades e elimina necessidade de credenciais AWS individuais. Usuários fazem login uma vez no provedor de identidade e obtêm acesso temporário a múltiplas contas AWS.

Estabeleça processo rigoroso de offboarding que inclua desabilitação imediata de credenciais quando colaboradores deixam a organização. Automatize este processo integrando sistemas de RH com gestão de identidades. Configure alertas para access keys antigas e uso simultâneo de credenciais de localizações diferentes.

---

## Erro 4: Permissões Anexadas Diretamente a Usuários

### Descrição do Problema

Políticas IAM são frequentemente anexadas diretamente a usuários individuais ao invés de serem gerenciadas através de grupos ou roles. Esta abordagem cria complexidade de gestão e inconsistências de permissões entre usuários com funções similares.

### Por Que É Perigoso

Gestão de permissões torna-se extremamente complexa e propensa a erros quando políticas são anexadas diretamente a usuários. Em uma organização com 50 desenvolvedores, se cada um tem políticas individuais, qualquer mudança de permissões requer modificação em 50 lugares diferentes. Isto é ineficiente e resulta em inconsistências onde alguns usuários têm permissões que outros não têm, sem justificativa clara.

Auditorias tornam-se significativamente mais difíceis. Para determinar quais permissões um desenvolvedor típico possui, é necessário examinar múltiplos usuários individuais e consolidar suas políticas. Não há visão clara de permissões padrão para cada função organizacional.

Onboarding de novos colaboradores é lento e propenso a erros. É necessário identificar um usuário existente com função similar, copiar suas políticas, e torcer para que este usuário de referência tenha as permissões corretas. Frequentemente, permissões são copiadas de usuários com privilégios excessivos, perpetuando o problema.

### Como Identificar

Utilize o console IAM ou AWS CLI para listar usuários e suas políticas anexadas. Comando `aws iam list-users` seguido de `aws iam list-attached-user-policies --user-name <username>` para cada usuário revela políticas anexadas diretamente. Grande número de políticas anexadas diretamente a usuários indica este problema.

O **AWS IAM Access Analyzer** e ferramentas como **Prowler** verificam conformidade com best practice de anexar políticas a grupos ao invés de usuários. CIS AWS Foundations Benchmark inclui controle específico (1.16) que verifica se políticas são anexadas apenas a grupos ou roles.

Revise a estrutura organizacional de IAM. Ausência de grupos IAM ou grupos com poucos membros enquanto muitos usuários têm políticas individuais indica gestão inadequada de permissões.

### Como Corrigir

Crie grupos IAM baseados em funções organizacionais: `Developers`, `DevOps`, `DataAnalysts`, `Auditors`, `Administrators`. Cada grupo deve ter políticas específicas anexadas que refletem as necessidades da função. Mova usuários para os grupos apropriados e remova políticas anexadas diretamente.

Para casos onde usuários precisam de permissões temporárias ou específicas, utilize roles IAM que usuários podem assumir temporariamente ao invés de anexar políticas permanentes. Por exemplo, crie role `ProductionAccess` que desenvolvedores podem assumir apenas quando necessário, com duração de sessão limitada.

Documente claramente quais grupos existem, quais permissões cada grupo possui, e quais funções organizacionais devem pertencer a cada grupo. Estabeleça processo de revisão trimestral onde gestores aprovam formalmente os membros de cada grupo.

Implemente processo de onboarding padronizado onde novos colaboradores são adicionados aos grupos apropriados baseado em sua função, garantindo consistência de permissões. Processo de offboarding deve remover usuários de todos os grupos e desabilitar suas credenciais.

---

## Erro 5: Access Keys Antigas e Não Rotacionadas

### Descrição do Problema

Access keys AWS (Access Key ID e Secret Access Key) são frequentemente criadas e nunca rotacionadas, permanecendo ativas por anos. Estas credenciais de longa duração aumentam significativamente o risco de comprometimento, pois há mais tempo e oportunidades para exposição acidental ou roubo.

### Por Que É Perigoso

Quanto mais tempo uma credencial permanece ativa, maior a probabilidade de ter sido exposta. Access keys podem vazar através de múltiplos vetores: commit acidental em repositórios públicos no GitHub (extremamente comum), armazenamento em arquivos de configuração não criptografados, logs de aplicações, backups não seguros, ou comprometimento de máquinas de desenvolvimento.

Credenciais antigas frequentemente são esquecidas e não são monitoradas. Organizações perdem rastreamento de onde estas keys estão sendo utilizadas e para quê. Em caso de necessidade de revogação, não há certeza de quais sistemas serão impactados, resultando em relutância em rotacionar por medo de quebrar aplicações.

Ferramentas automatizadas de scanning (bots) constantemente varrem repositórios públicos, logs e outras fontes buscando por credenciais AWS expostas. Quando encontradas, estas credenciais são imediatamente exploradas para mineração de criptomoedas, exfiltração de dados ou outros ataques. O tempo entre exposição e exploração pode ser medido em minutos.

### Como Identificar

O **IAM Credential Report** indica a idade de cada access key e quando foi utilizada pela última vez. Filtre por keys com mais de 90 dias de idade. Comando AWS CLI: `aws iam generate-credential-report && aws iam get-credential-report --output text | base64 -d | grep -v "N/A" | awk -F',' '{if ($11 > 90) print $1,$11}'`.

O **AWS Security Hub** inclui controles que verificam automaticamente se access keys são rotacionadas regularmente. Ferramentas como **Prowler** também automatizam esta verificação e geram relatórios de keys antigas.

Configure alarmes no CloudWatch para alertar quando access keys atingem 80 dias de idade, permitindo ação proativa antes de atingir 90 dias. Implemente dashboard de segurança que visualiza idade de todas as access keys da organização.

### Como Corrigir

Estabeleça política organizacional de rotação obrigatória de access keys a cada 90 dias. Implemente processo automatizado que envia alertas para usuários quando suas keys estão próximas da expiração. Para keys que excedem 90 dias, considere desabilitação automática com notificação ao usuário.

Prefira uso de **IAM roles** ao invés de access keys sempre que possível. Para aplicações rodando em AWS (EC2, ECS, Lambda), utilize roles anexadas aos recursos que fornecem credenciais temporárias automaticamente rotacionadas. Isto elimina completamente a necessidade de access keys de longa duração.

Para casos onde access keys são necessárias (aplicações rodando fora da AWS, ferramentas de CI/CD), implemente processo de rotação: criar nova key, atualizar aplicação para usar nova key, testar funcionamento, desabilitar key antiga, monitorar por período de segurança, excluir key antiga permanentemente.

Utilize **AWS Secrets Manager** ou **AWS Systems Manager Parameter Store** para armazenar credenciais de forma segura e criptografada, com controles de acesso granulares e auditoria de uso. Nunca armazene access keys em código-fonte, arquivos de configuração não criptografados ou variáveis de ambiente em sistemas compartilhados.

Implemente scanning automatizado de repositórios de código buscando por credenciais expostas. Ferramentas como **git-secrets**, **truffleHog** e **GitHub Secret Scanning** identificam credenciais em commits e previnem commits contendo secrets.

---

## Erro 6: Ausência de Monitoramento de Atividades IAM

### Descrição do Problema

CloudTrail está habilitado mas logs não são analisados proativamente. Não há alertas configurados para atividades suspeitas ou violações de políticas. Mudanças em configurações IAM (criação de usuários, modificação de políticas, criação de access keys) não são monitoradas em tempo real.

### Por Que É Perigoso

Sem monitoramento, incidentes de segurança podem passar despercebidos por longos períodos. Estudos indicam que o tempo médio de detecção de violações (dwell time) é frequentemente medido em meses. Durante este período, atacantes estabelecem persistência, exfiltram dados, e cobrem rastros. Quanto maior o dwell time, maior o impacto e o custo de recuperação.

Atividades maliciosas como criação de backdoors (novos usuários administrativos), escalação de privilégios (modificação de políticas), e preparação para ataques (reconnaissance de recursos) não são detectadas. Violações de compliance e desvios de políticas de segurança só são identificados em auditorias formais, quando já causaram impacto.

A ausência de monitoramento também impede análise forense efetiva após incidentes. Sem logs detalhados e alertas, é difícil determinar o escopo do comprometimento, quais dados foram acessados, e como o atacante obteve acesso inicial.

### Como Identificar

Verifique se CloudTrail está habilitado e configurado corretamente para logar eventos de gestão em todas as regiões. Confirme que logs são armazenados em bucket S3 com retenção adequada e criptografia. Verifique se há alarmes configurados no CloudWatch para eventos críticos de IAM.

Revise configuração do **AWS Security Hub** e **GuardDuty**. Se não estiverem habilitados, não há detecção automatizada de ameaças. Verifique se há integração com SIEM corporativo para análise centralizada de logs.

Teste o sistema de alertas realizando ação que deveria gerar alerta (como criar novo usuário IAM) e verificando se notificação é recebida. Ausência de alerta indica falha no monitoramento.

### Como Corrigir

Habilite **AWS GuardDuty** em todas as regiões para detecção de ameaças baseada em machine learning. GuardDuty analisa logs do CloudTrail, VPC Flow Logs e DNS logs para identificar atividades maliciosas como reconnaissance, comprometimento de instâncias, e exfiltração de dados.

Configure **AWS Security Hub** para centralização de descobertas de segurança de múltiplos serviços. Security Hub agrega alertas do GuardDuty, IAM Access Analyzer, Macie, Inspector e outros serviços, fornecendo visão unificada da postura de segurança.

Crie alarmes no CloudWatch para eventos críticos de IAM: criação ou exclusão de usuários (`CreateUser`, `DeleteUser`), modificação de políticas (`PutUserPolicy`, `AttachUserPolicy`), criação de access keys (`CreateAccessKey`), mudanças em configurações de MFA, tentativas de login falhadas, e uso de credenciais root.

Configure notificações via **Amazon SNS** para alertas críticos, integrando com sistemas de ticketing, email, Slack ou PagerDuty. Estabeleça processo de resposta a incidentes com runbooks documentados que especificam ações a serem tomadas para cada tipo de alerta.

Implemente análise automatizada de logs usando **AWS Lambda** para processar eventos do CloudTrail em tempo real. Crie funções Lambda que analisam eventos e geram alertas customizados para padrões específicos da organização.

Estabeleça processo de revisão semanal de descobertas do Security Hub e GuardDuty. Crie dashboards de segurança usando CloudWatch Dashboards ou QuickSight para visualização contínua de métricas de segurança.

---

## Erro 7: Uso de Conta Root para Operações Cotidianas

### Descrição do Problema

A conta root da AWS (criada com email utilizado para abrir a conta) possui acesso irrestrito a todos os recursos e não pode ter suas permissões limitadas. Algumas organizações utilizam a conta root para operações cotidianas ao invés de criar usuários IAM com permissões apropriadas.

### Por Que É Perigoso

A conta root tem poderes absolutos e não pode ser restringida por políticas IAM, SCPs ou qualquer outro mecanismo de controle. Pode realizar operações que nenhum outro usuário pode, como fechar a conta AWS, modificar plano de suporte, e alterar configurações de billing. Comprometimento da conta root significa comprometimento total da conta AWS.

O uso cotidiano da conta root aumenta significativamente a superfície de ataque. Cada login, cada operação, cada acesso é uma oportunidade para comprometimento. Credenciais root utilizadas frequentemente têm maior probabilidade de serem expostas através de phishing, keyloggers, ou shoulder surfing.

Atividades realizadas com conta root são mais difíceis de auditar e atribuir. Enquanto usuários IAM têm identidades distintas, todas as ações da conta root aparecem simplesmente como "root", sem diferenciação entre diferentes pessoas que possam ter acesso.

### Como Identificar

Revise logs do CloudTrail buscando por eventos onde `userIdentity.type` é `Root`. Presença frequente de eventos root indica uso cotidiano. Comando AWS CLI: `aws cloudtrail lookup-events --lookup-attributes AttributeKey=Username,AttributeValue=root`.

O **AWS Security Hub** e **CIS AWS Foundations Benchmark** incluem controles que verificam se a conta root foi utilizada recentemente. Ferramentas como **Prowler** automatizam esta verificação.

Entreviste equipes técnicas para identificar se credenciais root são conhecidas e utilizadas por múltiplas pessoas. Verifique se há processo documentado de gestão de credenciais root.

### Como Corrigir

Pare imediatamente de utilizar conta root para operações cotidianas. Crie usuários IAM individuais com permissões apropriadas para todas as atividades operacionais. Mesmo administradores devem usar usuários IAM com políticas administrativas, não a conta root.

Habilite MFA na conta root utilizando hardware MFA device (não aplicativo de smartphone) para segurança máxima. Armazene o dispositivo MFA em local físico seguro (cofre) com acesso restrito.

Remova access keys da conta root se existirem. Conta root não deve ter access keys programáticas. Verifique com comando `aws iam get-account-summary` e procure por `AccountAccessKeysPresent`.

Estabeleça processo de gestão de credenciais root: senha complexa e única armazenada em cofre de senhas corporativo (como 1Password, LastPass Enterprise), acesso ao cofre restrito a duas ou três pessoas específicas (CEO, CTO, CISO), procedimento documentado de quando e como acessar conta root (apenas para operações que absolutamente requerem root), e auditoria de todos os acessos root com justificativa documentada.

Configure alertas no CloudWatch para qualquer uso da conta root, gerando notificação imediata para equipe de segurança. Realize revisões trimestrais de necessidade de acesso root e considere rotação de senha root periodicamente.

---

## Erro 8: Políticas Inline ao Invés de Managed Policies

### Descrição do Problema

Políticas inline são criadas e anexadas diretamente a usuários, grupos ou roles individuais, ao invés de utilizar managed policies (AWS managed ou customer managed) que podem ser reutilizadas e gerenciadas centralmente.

### Por Que É Perigoso

Políticas inline criam fragmentação e dificultam gestão centralizada de permissões. Cada política inline é independente, então uma mudança de segurança que precisa ser aplicada a múltiplos usuários requer modificação de múltiplas políticas inline. Isto é propenso a erros e resulta em inconsistências.

Auditoria torna-se extremamente complexa. Para entender quais permissões existem na organização, é necessário examinar cada usuário, grupo e role individualmente, consolidando suas políticas inline. Não há visão centralizada de políticas em uso.

Reutilização de políticas é impossível. Se múltiplos grupos precisam das mesmas permissões, é necessário duplicar a política inline em cada grupo. Mudanças futuras requerem atualização de todas as cópias, processo manual e propenso a esquecimentos.

### Como Identificar

Utilize AWS CLI para listar políticas inline: `aws iam list-user-policies`, `aws iam list-group-policies`, `aws iam list-role-policies`. Presença de múltiplas políticas inline indica este problema.

Compare com managed policies: `aws iam list-policies --scope Local` lista customer managed policies. Ausência de customer managed policies enquanto há muitas inline policies indica gestão inadequada.

Ferramentas como **Prowler** verificam uso excessivo de políticas inline e recomendam migração para managed policies.

### Como Corrigir

Identifique políticas inline comuns ou similares. Consolide-as em customer managed policies que podem ser reutilizadas. Crie managed policy, anexe-a aos recursos apropriados, e remova políticas inline correspondentes.

Estabeleça padrão organizacional de utilizar apenas managed policies. Políticas inline devem ser exceção rara para casos específicos que realmente não podem ser generalizados.

Utilize versionamento de managed policies para rastrear mudanças ao longo do tempo. Managed policies suportam múltiplas versões, permitindo rollback se mudanças causarem problemas.

Documente cada managed policy claramente, especificando seu propósito, quais funções devem utilizá-la, e justificativa para permissões concedidas.

---

## Conclusão

Os erros documentados neste guia são extremamente comuns mas completamente evitáveis através de conhecimento adequado, processos estruturados e uso de ferramentas apropriadas. A compreensão profunda destes erros, suas implicações de segurança e métodos de correção é fundamental para qualquer profissional que trabalha com AWS. A implementação das correções recomendadas reduzirá significativamente riscos de segurança e alinhará a organização com best practices reconhecidas internacionalmente.

---

**Documento elaborado por:** Rafael Garcez  
**Escola da Nuvem:** Turma BRASAO 227
