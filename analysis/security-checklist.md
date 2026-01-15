# Checklist de Segurança AWS IAM
## Guia Prático de Implementação e Auditoria

---

## Sobre Este Checklist

Este checklist consolida as melhores práticas de segurança para **AWS Identity and Access Management (IAM)** em um formato prático e acionável. Pode ser utilizado para implementação inicial de controles de segurança, auditorias periódicas, ou avaliação de conformidade com frameworks reconhecidos como **CIS AWS Foundations Benchmark**, **AWS Well-Architected Framework** e **NIST Cybersecurity Framework**.

Cada item inclui indicação de prioridade (Crítica, Alta, Média) para auxiliar na priorização de esforços. Itens marcados como **Críticos** devem ser implementados imediatamente, pois representam vulnerabilidades severas. Itens de prioridade **Alta** devem ser endereçados em curto prazo, e itens de prioridade **Média** devem fazer parte de roadmap de melhorias contínuas.

---

## 1. Gestão de Conta Root

### ☐ Habilitar MFA na conta root
**Prioridade:** CRÍTICA  
**Descrição:** A conta root deve ter autenticação multi-fator habilitada utilizando hardware MFA device (YubiKey, AWS MFA device) para máxima segurança.  
**Como verificar:** Console AWS → IAM → Dashboard → Security Status  
**Como implementar:** Console AWS → IAM → Dashboard → Add MFA → Hardware MFA device  
**Referência:** CIS AWS Foundations Benchmark 1.5, 1.6

### ☐ Remover access keys da conta root
**Prioridade:** CRÍTICA  
**Descrição:** Conta root não deve possuir access keys programáticas. Se existirem, devem ser removidas imediatamente.  
**Como verificar:** `aws iam get-account-summary` → verificar `AccountAccessKeysPresent`  
**Como implementar:** Console AWS → IAM → My Security Credentials → Delete access keys  
**Referência:** CIS AWS Foundations Benchmark 1.12

### ☐ Não utilizar conta root para operações cotidianas
**Prioridade:** CRÍTICA  
**Descrição:** Conta root deve ser utilizada apenas para operações que absolutamente requerem root. Criar usuários IAM para todas as operações cotidianas.  
**Como verificar:** CloudTrail logs → buscar eventos com `userIdentity.type = Root`  
**Como implementar:** Criar usuários IAM com permissões apropriadas e cessar uso de root  
**Referência:** AWS IAM Best Practices, CIS AWS Foundations Benchmark 1.7

### ☐ Configurar alertas para uso da conta root
**Prioridade:** ALTA  
**Descrição:** Criar alarme no CloudWatch que notifica sempre que conta root é utilizada.  
**Como verificar:** CloudWatch → Alarms → verificar alarme para eventos root  
**Como implementar:** CloudWatch → Logs → Create metric filter → Create alarm  
**Referência:** CIS AWS Foundations Benchmark 3.3

### ☐ Armazenar credenciais root de forma segura
**Prioridade:** ALTA  
**Descrição:** Senha root deve ser complexa, única e armazenada em cofre de senhas corporativo com acesso restrito.  
**Como verificar:** Revisar processo de gestão de credenciais root  
**Como implementar:** Utilizar 1Password, LastPass Enterprise ou similar com acesso restrito  
**Referência:** AWS Security Best Practices

---

## 2. Autenticação Multi-Fator (MFA)

### ☐ Habilitar MFA para todos os usuários com acesso à console
**Prioridade:** CRÍTICA  
**Descrição:** Todos os usuários que acessam AWS Console devem ter MFA habilitado.  
**Como verificar:** IAM Credential Report → coluna `mfa_active`  
**Como implementar:** Política organizacional + processo de onboarding  
**Referência:** CIS AWS Foundations Benchmark 1.2

### ☐ Exigir MFA para operações sensíveis via políticas IAM
**Prioridade:** ALTA  
**Descrição:** Políticas IAM devem incluir condição `aws:MultiFactorAuthPresent: true` para operações críticas.  
**Como verificar:** Revisar políticas IAM buscando condição MFA  
**Como implementar:** Adicionar condição em todas as políticas com permissões sensíveis  
**Referência:** AWS IAM Best Practices

### ☐ Implementar MFA para acesso programático (API)
**Prioridade:** ALTA  
**Descrição:** Usuários que acessam AWS via CLI/SDK devem utilizar tokens de sessão temporários obtidos com MFA.  
**Como verificar:** Revisar processo de acesso programático  
**Como implementar:** `aws sts get-session-token --serial-number <mfa> --token-code <code>`  
**Referência:** AWS Security Best Practices

### ☐ Utilizar hardware MFA para administradores
**Prioridade:** MÉDIA  
**Descrição:** Administradores e usuários com acesso a produção devem utilizar hardware MFA (YubiKey) ao invés de aplicativos.  
**Como verificar:** IAM Credential Report → tipo de MFA device  
**Como implementar:** Adquirir e distribuir hardware MFA devices  
**Referência:** AWS Security Best Practices

---

## 3. Gestão de Usuários e Credenciais

### ☐ Criar usuários IAM individuais (não compartilhar credenciais)
**Prioridade:** CRÍTICA  
**Descrição:** Cada pessoa deve ter usuário IAM individual. Eliminar contas compartilhadas.  
**Como verificar:** Revisar nomes de usuários buscando padrões como "shared", "team"  
**Como implementar:** Criar usuários individuais e eliminar contas compartilhadas  
**Referência:** AWS IAM Best Practices, CIS AWS Foundations Benchmark 1.1

### ☐ Rotacionar access keys a cada 90 dias
**Prioridade:** ALTA  
**Descrição:** Access keys devem ser rotacionadas regularmente. Keys com mais de 90 dias devem ser substituídas.  
**Como verificar:** IAM Credential Report → colunas `access_key_*_last_rotated`  
**Como implementar:** Processo de rotação: criar nova key → atualizar aplicação → desabilitar antiga → excluir  
**Referência:** CIS AWS Foundations Benchmark 1.3, 1.4

### ☐ Remover credenciais não utilizadas
**Prioridade:** ALTA  
**Descrição:** Usuários inativos por mais de 90 dias devem ser desabilitados. Credenciais não utilizadas devem ser removidas.  
**Como verificar:** IAM Credential Report → coluna `password_last_used`, `access_key_*_last_used_date`  
**Como implementar:** Desabilitar usuários inativos e remover access keys não utilizadas  
**Referência:** CIS AWS Foundations Benchmark 1.3

### ☐ Implementar política de senhas forte
**Prioridade:** ALTA  
**Descrição:** Política de senhas deve exigir: mínimo 14 caracteres, letras maiúsculas e minúsculas, números, símbolos, e prevenir reutilização.  
**Como verificar:** Console IAM → Account Settings → Password Policy  
**Como implementar:** Configurar password policy com requisitos mínimos  
**Referência:** CIS AWS Foundations Benchmark 1.8, 1.9, 1.10, 1.11

### ☐ Preferir IAM roles ao invés de access keys
**Prioridade:** MÉDIA  
**Descrição:** Para aplicações em AWS, utilizar roles anexadas a recursos ao invés de access keys embarcadas.  
**Como verificar:** Revisar aplicações buscando access keys hardcoded  
**Como implementar:** Anexar roles a EC2, ECS, Lambda; remover access keys  
**Referência:** AWS IAM Best Practices

---

## 4. Políticas e Permissões

### ☐ Aplicar princípio do menor privilégio
**Prioridade:** CRÍTICA  
**Descrição:** Usuários devem ter apenas permissões estritamente necessárias para suas funções. Eliminar uso de wildcards (*).  
**Como verificar:** Revisar políticas IAM buscando `Action: *` e `Resource: *`  
**Como implementar:** Criar políticas granulares com ações e recursos específicos  
**Referência:** AWS IAM Best Practices, CIS AWS Foundations Benchmark 1.16

### ☐ Anexar políticas a grupos, não a usuários individuais
**Prioridade:** ALTA  
**Descrição:** Políticas devem ser anexadas a grupos IAM. Usuários são adicionados aos grupos apropriados.  
**Como verificar:** Verificar se usuários têm políticas anexadas diretamente  
**Como implementar:** Criar grupos por função, anexar políticas aos grupos, mover usuários  
**Referência:** AWS IAM Best Practices, CIS AWS Foundations Benchmark 1.16

### ☐ Utilizar managed policies ao invés de inline policies
**Prioridade:** MÉDIA  
**Descrição:** Preferir customer managed policies reutilizáveis ao invés de políticas inline.  
**Como verificar:** `aws iam list-user-policies`, `list-group-policies`, `list-role-policies`  
**Como implementar:** Consolidar políticas inline em managed policies  
**Referência:** AWS IAM Best Practices

### ☐ Implementar segregação de ambientes
**Prioridade:** ALTA  
**Descrição:** Usuários de desenvolvimento não devem ter acesso a produção. Utilizar tags e condições IAM.  
**Como verificar:** Revisar políticas buscando condições baseadas em tags de ambiente  
**Como implementar:** Taguear recursos, adicionar condições `aws:ResourceTag/Environment`  
**Referência:** AWS Well-Architected Security Pillar

### ☐ Utilizar negações explícitas para proteção adicional
**Prioridade:** MÉDIA  
**Descrição:** Adicionar statements Deny para prevenir ações críticas (modificação de IAM, desabilitação de CloudTrail).  
**Como verificar:** Revisar políticas buscando statements com `Effect: Deny`  
**Como implementar:** Adicionar negações explícitas em políticas  
**Referência:** AWS IAM Best Practices

### ☐ Revisar permissões trimestralmente
**Prioridade:** MÉDIA  
**Descrição:** Estabelecer processo de revisão trimestral onde gestores aprovam formalmente permissões de suas equipes.  
**Como verificar:** Verificar existência de processo documentado  
**Como implementar:** Criar processo de revisão com aprovações formais  
**Referência:** AWS IAM Best Practices

---

## 5. Monitoramento e Auditoria

### ☐ Habilitar CloudTrail em todas as regiões
**Prioridade:** CRÍTICA  
**Descrição:** CloudTrail deve estar habilitado em todas as regiões para logar eventos de gestão.  
**Como verificar:** Console CloudTrail → Trails → verificar multi-region trail  
**Como implementar:** Criar trail multi-region com log file validation habilitado  
**Referência:** CIS AWS Foundations Benchmark 2.1, 2.2

### ☐ Configurar log file validation no CloudTrail
**Prioridade:** ALTA  
**Descrição:** Habilitar validação de integridade de logs para prevenir adulteração.  
**Como verificar:** Console CloudTrail → Trail settings → Log file validation  
**Como implementar:** Habilitar log file validation no trail  
**Referência:** CIS AWS Foundations Benchmark 2.2

### ☐ Integrar CloudTrail com CloudWatch Logs
**Prioridade:** ALTA  
**Descrição:** Enviar logs do CloudTrail para CloudWatch Logs para análise e alertas em tempo real.  
**Como verificar:** Console CloudTrail → Trail settings → CloudWatch Logs  
**Como implementar:** Configurar integração CloudTrail → CloudWatch Logs  
**Referência:** CIS AWS Foundations Benchmark 2.4

### ☐ Habilitar AWS GuardDuty
**Prioridade:** CRÍTICA  
**Descrição:** GuardDuty fornece detecção de ameaças baseada em machine learning.  
**Como verificar:** Console GuardDuty → verificar se está habilitado  
**Como implementar:** Habilitar GuardDuty em todas as regiões  
**Referência:** AWS Security Best Practices

### ☐ Habilitar AWS Security Hub
**Prioridade:** ALTA  
**Descrição:** Security Hub centraliza descobertas de segurança de múltiplos serviços.  
**Como verificar:** Console Security Hub → verificar se está habilitado  
**Como implementar:** Habilitar Security Hub e padrões de conformidade (CIS, AWS Foundational)  
**Referência:** AWS Security Best Practices

### ☐ Configurar alarmes para eventos críticos de IAM
**Prioridade:** ALTA  
**Descrição:** Criar alarmes no CloudWatch para: criação de usuários, modificação de políticas, criação de access keys, tentativas de login falhadas.  
**Como verificar:** CloudWatch → Alarms → verificar alarmes IAM  
**Como implementar:** Criar metric filters e alarmes para eventos críticos  
**Referência:** CIS AWS Foundations Benchmark 3.x

### ☐ Gerar e revisar IAM Credential Report mensalmente
**Prioridade:** MÉDIA  
**Descrição:** Revisar mensalmente relatório de credenciais para identificar anomalias.  
**Como verificar:** `aws iam generate-credential-report && aws iam get-credential-report`  
**Como implementar:** Agendar revisão mensal do relatório  
**Referência:** AWS IAM Best Practices

### ☐ Utilizar IAM Access Analyzer
**Prioridade:** MÉDIA  
**Descrição:** Access Analyzer identifica recursos compartilhados externamente e permissões excessivas.  
**Como verificar:** Console IAM → Access Analyzer  
**Como implementar:** Habilitar Access Analyzer e revisar findings regularmente  
**Referência:** AWS IAM Best Practices

---

## 6. Governança e Compliance

### ☐ Implementar AWS Organizations
**Prioridade:** MÉDIA  
**Descrição:** Para múltiplas contas AWS, utilizar Organizations para gestão centralizada.  
**Como verificar:** Console AWS Organizations  
**Como implementar:** Criar organization e adicionar contas  
**Referência:** AWS Multi-Account Best Practices

### ☐ Utilizar Service Control Policies (SCPs)
**Prioridade:** MÉDIA  
**Descrição:** SCPs forçam controles de segurança em nível organizacional que não podem ser contornados.  
**Como verificar:** Console AWS Organizations → Policies → Service control policies  
**Como implementar:** Criar SCPs para prevenir ações críticas  
**Referência:** AWS Organizations Best Practices

### ☐ Implementar AWS IAM Identity Center (AWS SSO)
**Prioridade:** MÉDIA  
**Descrição:** Autenticação federada com provedor de identidade corporativo elimina necessidade de credenciais AWS individuais.  
**Como verificar:** Console IAM Identity Center  
**Como implementar:** Configurar integração com IdP corporativo (AD, Okta, Azure AD)  
**Referência:** AWS IAM Best Practices

### ☐ Documentar todas as políticas e justificativas
**Prioridade:** MÉDIA  
**Descrição:** Cada política deve ter documentação clara de propósito e justificativa de permissões.  
**Como verificar:** Revisar existência de documentação de políticas  
**Como implementar:** Criar documentação estruturada de políticas  
**Referência:** Compliance Best Practices

### ☐ Estabelecer processo de solicitação de acesso
**Prioridade:** MÉDIA  
**Descrição:** Implementar workflow formal para solicitação, aprovação e concessão de acessos.  
**Como verificar:** Verificar existência de processo documentado  
**Como implementar:** Criar processo com aprovações de gestor e segurança  
**Referência:** Governance Best Practices

### ☐ Implementar processo de offboarding
**Prioridade:** ALTA  
**Descrição:** Desabilitação imediata de credenciais quando colaboradores deixam a organização.  
**Como verificar:** Revisar processo de offboarding  
**Como implementar:** Integrar RH com gestão de identidades para automação  
**Referência:** Security Best Practices

---

## 7. Proteção de Credenciais

### ☐ Não armazenar credenciais em código-fonte
**Prioridade:** CRÍTICA  
**Descrição:** Credenciais nunca devem ser commitadas em repositórios de código.  
**Como verificar:** Utilizar ferramentas como git-secrets, truffleHog  
**Como implementar:** Implementar scanning automatizado de repositórios  
**Referência:** AWS Security Best Practices

### ☐ Utilizar AWS Secrets Manager para armazenamento de credenciais
**Prioridade:** ALTA  
**Descrição:** Secrets devem ser armazenados em Secrets Manager ou Parameter Store, não em arquivos de configuração.  
**Como verificar:** Revisar aplicações buscando credenciais hardcoded  
**Como implementar:** Migrar credenciais para Secrets Manager  
**Referência:** AWS Security Best Practices

### ☐ Implementar rotação automática de secrets
**Prioridade:** MÉDIA  
**Descrição:** Secrets Manager suporta rotação automática de credenciais de banco de dados e outras.  
**Como verificar:** Console Secrets Manager → verificar rotação configurada  
**Como implementar:** Configurar rotação automática para secrets suportados  
**Referência:** AWS Secrets Manager Best Practices

### ☐ Criptografar logs e backups contendo credenciais
**Prioridade:** ALTA  
**Descrição:** Logs de aplicações e backups que possam conter credenciais devem ser criptografados.  
**Como verificar:** Revisar configurações de S3, CloudWatch Logs  
**Como implementar:** Habilitar criptografia em repouso (KMS)  
**Referência:** AWS Security Best Practices

---

## 8. Resposta a Incidentes

### ☐ Criar runbooks de resposta a incidentes IAM
**Prioridade:** ALTA  
**Descrição:** Documentar procedimentos para resposta a comprometimento de credenciais, escalação de privilégios, etc.  
**Como verificar:** Verificar existência de runbooks documentados  
**Como implementar:** Criar runbooks para cenários comuns  
**Referência:** AWS Security Incident Response Guide

### ☐ Estabelecer processo de rotação emergencial de credenciais
**Prioridade:** ALTA  
**Descrição:** Procedimento para rotação rápida de todas as credenciais em caso de comprometimento.  
**Como verificar:** Verificar existência de processo documentado  
**Como implementar:** Documentar e testar processo de rotação emergencial  
**Referência:** Incident Response Best Practices

### ☐ Configurar notificações para equipe de segurança
**Prioridade:** ALTA  
**Descrição:** Alertas críticos devem notificar equipe de segurança via múltiplos canais (email, Slack, PagerDuty).  
**Como verificar:** Testar sistema de notificações  
**Como implementar:** Configurar SNS com múltiplos endpoints  
**Referência:** AWS Security Best Practices

### ☐ Realizar simulações de resposta a incidentes
**Prioridade:** MÉDIA  
**Descrição:** Testar periodicamente runbooks através de simulações (tabletop exercises, red team).  
**Como verificar:** Verificar histórico de simulações  
**Como implementar:** Agendar simulações trimestrais  
**Referência:** Incident Response Best Practices

---

## 9. Treinamento e Conscientização

### ☐ Implementar programa de security awareness
**Prioridade:** MÉDIA  
**Descrição:** Treinamento regular sobre segurança, phishing, gestão de credenciais para todos os colaboradores.  
**Como verificar:** Verificar existência de programa de treinamento  
**Como implementar:** Criar programa de treinamento anual obrigatório  
**Referência:** Security Awareness Best Practices

### ☐ Documentar políticas de segurança IAM
**Prioridade:** MÉDIA  
**Descrição:** Políticas organizacionais de segurança devem ser documentadas e comunicadas claramente.  
**Como verificar:** Verificar existência de documentação de políticas  
**Como implementar:** Criar e publicar políticas de segurança  
**Referência:** Governance Best Practices

### ☐ Realizar onboarding de segurança para novos colaboradores
**Prioridade:** MÉDIA  
**Descrição:** Novos colaboradores devem receber treinamento de segurança antes de receber acessos.  
**Como verificar:** Verificar processo de onboarding  
**Como implementar:** Incluir treinamento de segurança no onboarding  
**Referência:** Security Awareness Best Practices

---

## Frequência de Revisão Recomendada

| Categoria | Frequência |
|-----------|-----------|
| Itens Críticos | Mensal |
| Itens de Alta Prioridade | Trimestral |
| Itens de Média Prioridade | Semestral |
| Checklist Completo | Anual |

---

## Ferramentas Recomendadas

- **AWS IAM Access Analyzer**: Análise de permissões e recursos compartilhados
- **AWS Security Hub**: Centralização de descobertas de segurança
- **AWS GuardDuty**: Detecção de ameaças
- **Prowler**: Auditoria de segurança open-source
- **ScoutSuite**: Auditoria multi-cloud open-source
- **git-secrets**: Prevenção de commit de credenciais
- **truffleHog**: Scanning de credenciais em repositórios

---

## Conclusão

Este checklist deve ser utilizado como guia vivo, revisado e atualizado regularmente conforme novas best practices emergem e a infraestrutura evolui. A implementação completa destes controles estabelece base sólida de segurança IAM alinhada com frameworks reconhecidos internacionalmente.

---

**Documento elaborado por:** Rafael Garcez  
**Escola da Nuvem:** Turma BRASAO 227
