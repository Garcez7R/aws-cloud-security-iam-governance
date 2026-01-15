# Análise de Riscos em AWS IAM
## TechNova Solutions - Avaliação de Segurança

---

## Introdução

Este documento apresenta uma análise estruturada dos riscos de segurança identificados nas configurações de **AWS Identity and Access Management (IAM)** da TechNova Solutions. A análise foi conduzida seguindo metodologias reconhecidas de gestão de riscos, incluindo elementos dos frameworks **NIST 800-30** (Guide for Conducting Risk Assessments) e **ISO 27005** (Information Security Risk Management).

O objetivo desta análise é identificar vulnerabilidades nas políticas de controle de acesso, avaliar o impacto potencial de sua exploração e propor medidas de mitigação alinhadas com as melhores práticas de segurança em cloud computing.

---

## Metodologia de Avaliação

A avaliação de riscos foi realizada através da análise sistemática das políticas IAM existentes, considerando três dimensões principais:

**Identificação de Ameaças**: Reconhecimento de configurações que violam princípios de segurança estabelecidos, como o princípio do menor privilégio e a segregação de funções. Foram examinadas políticas anexadas a usuários, grupos e roles, identificando permissões excessivas e ausência de controles restritivos.

**Avaliação de Impacto**: Análise das consequências potenciais caso as vulnerabilidades identificadas sejam exploradas, seja por agentes maliciosos externos, insiders mal-intencionados ou erros operacionais. O impacto foi classificado considerando aspectos de confidencialidade, integridade e disponibilidade dos dados e sistemas.

**Determinação de Probabilidade**: Estimativa da likelihood de ocorrência de incidentes baseada em fatores como exposição da vulnerabilidade, complexidade de exploração, controles compensatórios existentes e histórico de incidentes similares no setor.

A classificação de riscos segue a matriz de severidade que combina impacto e probabilidade, resultando em níveis de risco: **Crítico**, **Alto**, **Médio** e **Baixo**.

---

## Riscos Identificados

### Risco 1: Permissões Administrativas Irrestritas

**Descrição do Risco**: Múltiplos usuários possuem políticas IAM com permissões administrativas completas (`Action: "*"` e `Resource: "*"`), incluindo desenvolvedores, estagiários e contas de serviço. Esta configuração viola fundamentalmente o princípio do menor privilégio e cria uma superfície de ataque extremamente ampla.

**Classificação de Severidade**: **CRÍTICO**

**Impacto Potencial**: O impacto desta vulnerabilidade é catastrófico. Um usuário com estas permissões pode executar qualquer ação em qualquer recurso da conta AWS, incluindo exclusão de buckets S3 com dados críticos, terminação de instâncias EC2 de produção, modificação de configurações de segurança, criação de novos usuários administrativos, desabilitação de logs de auditoria e exfiltração de dados sensíveis. Em um cenário de comprometimento de credenciais, um atacante teria controle total sobre toda a infraestrutura cloud da organização. O impacto financeiro pode incluir custos de recuperação de desastres, perda de receita devido a indisponibilidade de serviços, multas regulatórias por violação de dados e danos reputacionais significativos.

**Probabilidade de Ocorrência**: **ALTA**. A probabilidade é elevada devido a múltiplos fatores: o grande número de usuários com estas permissões aumenta a superfície de ataque; credenciais podem ser comprometidas através de phishing, malware ou exposição acidental em repositórios de código; a ausência de controles como MFA obrigatório facilita o acesso não autorizado; e erros humanos são estatisticamente inevitáveis quando muitos usuários possuem permissões destrutivas.

**Nível de Risco Global**: **CRÍTICO** (Impacto Crítico × Probabilidade Alta)

**Evidências Técnicas**: A política `insecure-admin.json` documentada neste repositório exemplifica esta configuração vulnerável. Análise de CloudTrail revelou que esta política estava anexada a 15 usuários diferentes, incluindo 8 desenvolvedores, 3 estagiários, 2 contas de serviço e 2 administradores de sistemas.

**Medidas de Mitigação Recomendadas**: A correção deste risco requer ações imediatas e estruturadas. Primeiramente, deve-se realizar auditoria completa de todos os usuários com permissões administrativas, documentando justificativas de negócio para cada caso. Em seguida, implementar políticas granulares baseadas no princípio do menor privilégio, como exemplificado em `developer-least-privilege.json`. Para os poucos casos legítimos que requerem acesso administrativo, implementar controles compensatórios rigorosos: MFA obrigatório, restrições por IP de origem, limitação de horário de acesso, sessões temporárias com duração máxima, e aprovação via workflow para operações sensíveis. Todas as políticas administrativas devem ser anexadas a roles com assume role policies restritivas, nunca diretamente a usuários. Implementar monitoramento contínuo com alertas em tempo real para uso de permissões administrativas. Estabelecer processo de revisão trimestral de permissões com aprovação formal de gestores.

**Indicadores de Comprometimento**: Monitorar CloudTrail para: tentativas de criação de novos usuários IAM fora do processo padrão; modificações em políticas de segurança; desabilitação de CloudTrail ou GuardDuty; acesso a recursos de múltiplas regiões simultaneamente; download massivo de dados de S3; tentativas de escalação de privilégios; e acessos originados de localizações geográficas incomuns.

---

### Risco 2: Ausência de Segregação entre Ambientes

**Descrição do Risco**: Usuários de desenvolvimento possuem acesso irrestrito a recursos de produção devido à ausência de controles baseados em tags ou ARNs específicos. Não há separação lógica entre ambientes de desenvolvimento, homologação e produção nas políticas IAM.

**Classificação de Severidade**: **ALTO**

**Impacto Potencial**: Esta falha de segregação pode resultar em modificações acidentais ou intencionais em sistemas de produção por usuários que deveriam operar apenas em desenvolvimento. Cenários comuns incluem: exclusão acidental de banco de dados de produção ao tentar limpar ambiente de desenvolvimento; deployment de código não testado diretamente em produção; modificação de configurações críticas de produção durante troubleshooting; e exposição de dados sensíveis de clientes a desenvolvedores que não possuem necessidade legítima de acesso. O impacto inclui indisponibilidade de serviços críticos, perda de dados de clientes, violação de regulamentações de proteção de dados (LGPD, GDPR), e comprometimento da integridade de sistemas produtivos.

**Probabilidade de Ocorrência**: **ALTA**. A probabilidade é elevada porque erros humanos são inevitáveis em ambientes onde não há barreiras técnicas. Desenvolvedores frequentemente trabalham sob pressão e podem inadvertidamente executar comandos em ambiente errado. A ausência de confirmações adicionais para operações em produção aumenta o risco.

**Nível de Risco Global**: **ALTO** (Impacto Alto × Probabilidade Alta)

**Evidências Técnicas**: Análise das políticas IAM revelou que nenhuma política utiliza condições baseadas em tags de ambiente (`ec2:ResourceTag/Environment`). Não há statements de negação explícita para recursos de produção em políticas de desenvolvimento.

**Medidas de Mitigação Recomendadas**: Implementar estratégia abrangente de segregação de ambientes. Todos os recursos AWS devem ser tagueados obrigatoriamente com tag `Environment` (valores: production, staging, development). Políticas IAM devem incluir condições que restrinjam acesso baseado nestas tags, como demonstrado em `developer-least-privilege.json`. Criar grupos IAM distintos para cada ambiente: `Developers-Dev`, `Developers-Staging`, `Engineers-Production`. Implementar AWS Organizations com Service Control Policies (SCPs) que forcem a segregação em nível de conta. Considerar uso de contas AWS separadas para cada ambiente, conectadas via AWS Organizations. Estabelecer processo de change management para produção com aprovações formais. Implementar ferramentas de Infrastructure as Code (Terraform, CloudFormation) com pipelines CI/CD que previnam deployments manuais em produção.

**Indicadores de Comprometimento**: Monitorar: acessos a recursos de produção por usuários de desenvolvimento; modificações em recursos de produção fora de janelas de manutenção aprovadas; e tentativas de acesso negadas a recursos de produção.

---

### Risco 3: Ausência de Autenticação Multi-Fator (MFA)

**Descrição do Risco**: A autenticação multi-fator não é obrigatória para usuários com acesso à console AWS ou para operações sensíveis via API. Usuários podem acessar recursos críticos utilizando apenas senha, que pode ser comprometida através de diversos vetores de ataque.

**Classificação de Severidade**: **ALTO**

**Impacto Potencial**: A ausência de MFA torna as contas vulneráveis a comprometimento através de phishing, credential stuffing, keyloggers, e vazamento de senhas em data breaches de terceiros. Uma vez que credenciais sejam obtidas, atacantes têm acesso irrestrito aos recursos permitidos pela política IAM do usuário. Em combinação com o Risco 1 (permissões administrativas), isto representa uma vulnerabilidade crítica. O impacto inclui acesso não autorizado a dados sensíveis, modificação ou exclusão de recursos, criação de recursos para mineração de criptomoedas (cryptojacking), e uso da infraestrutura comprometida como ponto de partida para ataques a terceiros.

**Probabilidade de Ocorrência**: **MÉDIA**. Embora ataques de phishing e comprometimento de credenciais sejam comuns, a probabilidade é moderada devido à necessidade de o atacante identificar e direcionar especificamente usuários da organização. No entanto, campanhas de phishing genéricas e vazamentos de credenciais em breaches de terceiros são ocorrências frequentes.

**Nível de Risco Global**: **ALTO** (Impacto Alto × Probabilidade Média)

**Evidências Técnicas**: Análise do IAM Credential Report revelou que 12 de 20 usuários não possuem MFA habilitado. Não há políticas IAM com condições que exijam MFA para operações sensíveis (`aws:MultiFactorAuthPresent`).

**Medidas de Mitigação Recomendadas**: Implementar MFA obrigatório para todos os usuários através de política organizacional. Criar política IAM que negue todas as operações exceto habilitação de MFA para usuários que ainda não o configuraram. Adicionar condições `aws:MultiFactorAuthPresent: true` em todas as políticas que concedem permissões sensíveis. Implementar MFA para acesso programático (API) através de tokens de sessão temporários obtidos via `aws sts get-session-token --serial-number <mfa-device> --token-code <code>`. Estabelecer processo de onboarding que inclua configuração obrigatória de MFA antes de concessão de acessos. Utilizar hardware MFA tokens (YubiKey) para administradores e usuários com acesso a produção. Implementar alertas para logins sem MFA e desabilitar automaticamente usuários que não configurarem MFA em 7 dias.

**Indicadores de Comprometimento**: Monitorar: logins de localizações geográficas incomuns; múltiplas tentativas de login falhadas seguidas de sucesso; acessos em horários atípicos; e mudanças de senha seguidas imediatamente de ações sensíveis.

---

### Risco 4: Compartilhamento de Credenciais

**Descrição do Risco**: Identificação de padrões de uso que sugerem compartilhamento de credenciais entre múltiplos usuários, incluindo uso de contas genéricas (como "developer-shared") e access keys de longa duração não rotacionadas.

**Classificação de Severidade**: **MÉDIO**

**Impacto Potencial**: O compartilhamento de credenciais impossibilita a atribuição precisa de ações a indivíduos específicos, comprometendo a auditoria e a não-repudiação. Em caso de incidente, torna-se impossível determinar quem realizou ações maliciosas ou erradas. Dificulta a revogação de acesso quando um membro da equipe deixa a organização, pois múltiplas pessoas conhecem as credenciais. Aumenta o risco de exposição acidental de credenciais, pois quanto mais pessoas as conhecem, maior a probabilidade de vazamento. Viola requisitos de compliance de diversos frameworks de segurança.

**Probabilidade de Ocorrência**: **MÉDIA**. A prática de compartilhamento de credenciais é comum em organizações com processos de segurança imaturos, especialmente em equipes de desenvolvimento sob pressão de prazos.

**Nível de Risco Global**: **MÉDIO** (Impacto Médio × Probabilidade Média)

**Evidências Técnicas**: CloudTrail logs mostram mesmas credenciais sendo utilizadas simultaneamente de endereços IP diferentes. IAM Credential Report indica access keys com mais de 180 dias sem rotação. Existência de usuários com nomes genéricos como "developer-shared" e "team-access".

**Medidas de Mitigação Recomendadas**: Eliminar todas as contas compartilhadas, criando usuários individuais para cada pessoa. Implementar política de rotação obrigatória de access keys a cada 90 dias. Preferir uso de roles IAM com credenciais temporárias ao invés de access keys de longa duração. Para aplicações, utilizar IAM roles anexadas a instâncias EC2 ou ECS tasks ao invés de access keys embarcadas. Implementar AWS IAM Identity Center (sucessor do AWS SSO) para autenticação federada com provedor de identidade corporativo. Estabelecer processo de offboarding que inclua desabilitação imediata de credenciais quando colaboradores deixam a organização. Implementar alertas para access keys antigas e uso simultâneo de credenciais.

**Indicadores de Comprometimento**: Monitorar: uso de mesmas credenciais de localizações geográficas distantes simultaneamente; mudanças bruscas em padrões de uso de credenciais; e access keys com mais de 90 dias sem rotação.

---

### Risco 5: Ausência de Monitoramento e Alertas

**Descrição do Risco**: Não há configuração adequada de monitoramento contínuo de ações IAM e alertas em tempo real para atividades suspeitas. CloudTrail está habilitado mas logs não são analisados proativamente. Ausência de integração com GuardDuty e Security Hub.

**Classificação de Severidade**: **MÉDIO**

**Impacto Potencial**: A ausência de monitoramento significa que incidentes de segurança podem passar despercebidos por longos períodos, aumentando significativamente o impacto. Atacantes têm tempo para estabelecer persistência, exfiltrar dados e cobrir rastros. Violações de compliance podem não ser detectadas até auditorias formais. A organização opera "às cegas" em relação à sua postura de segurança real.

**Probabilidade de Ocorrência**: **ALTA**. Sem monitoramento, a probabilidade de incidentes não detectados é alta, pois não há mecanismo de detecção ativa.

**Nível de Risco Global**: **MÉDIO** (Impacto Médio × Probabilidade Alta)

**Evidências Técnicas**: CloudTrail está habilitado mas logs são armazenados em S3 sem processamento. Não há alarmes configurados no CloudWatch. GuardDuty não está habilitado. Security Hub não está configurado. Não há integração com SIEM corporativo.

**Medidas de Mitigação Recomendadas**: Habilitar AWS GuardDuty em todas as regiões para detecção de ameaças baseada em machine learning. Configurar AWS Security Hub para centralização de descobertas de segurança. Criar alarmes no CloudWatch para eventos críticos: criação de usuários IAM, modificação de políticas, desabilitação de CloudTrail, tentativas de acesso negadas repetidas, e uso de credenciais root. Implementar análise automatizada de logs do CloudTrail usando AWS Lambda ou soluções de SIEM. Configurar notificações via SNS para alertas críticos. Estabelecer processo de resposta a incidentes com runbooks documentados. Realizar revisões semanais de descobertas do Security Hub. Implementar dashboards de segurança para visibilidade contínua.

**Indicadores de Comprometimento**: A ausência de monitoramento impede a identificação de IOCs, o que reforça a criticidade desta mitigação.

---

## Matriz de Riscos Consolidada

| ID | Risco | Severidade | Impacto | Probabilidade | Prioridade |
|----|-------|------------|---------|---------------|------------|
| 1 | Permissões Administrativas Irrestritas | CRÍTICO | Crítico | Alta | 1 |
| 2 | Ausência de Segregação de Ambientes | ALTO | Alto | Alta | 2 |
| 3 | Ausência de MFA | ALTO | Alto | Média | 3 |
| 4 | Compartilhamento de Credenciais | MÉDIO | Médio | Média | 4 |
| 5 | Ausência de Monitoramento | MÉDIO | Médio | Alta | 5 |

---

## Roadmap de Mitigação

**Fase 1 - Ações Imediatas (0-7 dias)**:
Revogar permissões administrativas desnecessárias, implementar políticas de menor privilégio para desenvolvedores, habilitar MFA obrigatório para todos os usuários, e ativar GuardDuty e Security Hub.

**Fase 2 - Curto Prazo (1-4 semanas)**:
Implementar segregação de ambientes via tags e políticas condicionais, eliminar contas compartilhadas, rotacionar todas as access keys, e configurar alarmes críticos no CloudWatch.

**Fase 3 - Médio Prazo (1-3 meses)**:
Migrar para IAM Identity Center, implementar roles para aplicações, estabelecer processo de revisão trimestral de permissões, e integrar com SIEM corporativo.

**Fase 4 - Longo Prazo (3-6 meses)**:
Considerar separação de contas AWS por ambiente, implementar Service Control Policies, estabelecer programa de security awareness, e obter certificações de compliance.

---

## Conclusão

A análise identificou vulnerabilidades significativas nas configurações de IAM da TechNova Solutions, com destaque para permissões administrativas excessivas e ausência de controles fundamentais de segurança. A implementação das medidas de mitigação recomendadas reduzirá substancialmente a superfície de ataque e alinhará a organização com as melhores práticas de segurança em cloud computing. A priorização baseada em risco garante que as vulnerabilidades mais críticas sejam endereçadas primeiro, proporcionando melhoria rápida da postura de segurança.

---

**Documento elaborado por:** Rafael Garcez  
**Data:** Janeiro 2026  
**Escola da Nuvem:** Turma BRASAO 227
