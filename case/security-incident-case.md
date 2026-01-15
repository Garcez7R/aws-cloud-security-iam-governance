# Estudo de Caso: Incidente de Segurança IAM
## TechNova Solutions - Análise de Incidente e Resposta

---

## Resumo Executivo

Este documento apresenta análise detalhada de um incidente de segurança ocorrido na **TechNova Solutions** em dezembro de 2025, causado por configurações inadequadas de **AWS Identity and Access Management (IAM)**. O incidente resultou em acesso não autorizado a dados sensíveis de clientes, exposição de propriedade intelectual, e custos financeiros significativos. A análise inclui cronologia do evento, investigação de causa raiz, impactos identificados, medidas corretivas implementadas e lições aprendidas.

Este caso serve como exemplo educacional de como falhas aparentemente simples em configurações de IAM podem resultar em consequências severas, e demonstra a importância crítica de implementação rigorosa de princípios de segurança em ambientes cloud.

---

## 1. Contexto da Organização

### Perfil da Empresa

**TechNova Solutions** é uma empresa brasileira de e-commerce em crescimento acelerado, fundada em 2020, com foco em produtos eletrônicos e tecnologia. A empresa opera plataforma web que processa aproximadamente 50.000 transações mensais, gerando receita anual de R$ 15 milhões. A equipe técnica é composta por 25 profissionais, incluindo desenvolvedores, engenheiros de DevOps, analistas de dados e administradores de sistemas.

### Infraestrutura AWS

A infraestrutura da TechNova Solutions é inteiramente hospedada na AWS, utilizando os seguintes serviços principais:

**Amazon S3** para armazenamento de assets estáticos (imagens de produtos, documentos), backups de banco de dados, e logs de aplicações. Três buckets principais: `technova-prod-assets` (público para imagens de produtos), `technova-prod-backups` (privado com dados sensíveis), e `technova-prod-logs` (logs de auditoria).

**Amazon EC2** para servidores de aplicação web, com 12 instâncias distribuídas em ambiente de produção e 6 instâncias em desenvolvimento. Instâncias de produção executam aplicação e-commerce principal, APIs de integração com gateways de pagamento, e serviços de processamento de pedidos.

**Amazon RDS** (PostgreSQL) para banco de dados relacional contendo dados de clientes, histórico de transações, inventário de produtos e informações financeiras. Instância principal de produção com 500GB de dados e réplica de leitura para relatórios.

**AWS Lambda** para processamento assíncrono de tarefas como envio de emails, geração de relatórios, e integração com sistemas de terceiros.

**Amazon CloudWatch** para monitoramento de infraestrutura e logs de aplicações.

### Postura de Segurança Pré-Incidente

Antes do incidente, a postura de segurança da TechNova Solutions apresentava vulnerabilidades significativas que não haviam sido endereçadas adequadamente:

Políticas IAM com permissões administrativas completas (`Action: *`, `Resource: *`) eram amplamente utilizadas, anexadas a 15 usuários diferentes incluindo desenvolvedores, estagiários e contas de serviço. Esta configuração havia sido implementada durante fase inicial de desenvolvimento por conveniência e nunca foi revisada.

Autenticação multi-fator não era obrigatória, com apenas 8 de 20 usuários tendo MFA habilitado voluntariamente. Não havia políticas IAM que exigissem MFA para operações sensíveis.

CloudTrail estava habilitado mas logs não eram analisados proativamente. Não havia alarmes configurados no CloudWatch para eventos críticos de IAM. GuardDuty e Security Hub não estavam habilitados.

Não havia segregação clara entre ambientes de desenvolvimento e produção nas políticas IAM. Desenvolvedores possuíam acesso irrestrito a recursos de produção, sem controles baseados em tags ou ARNs específicos.

Access keys com mais de 180 dias de idade eram comuns, e não havia processo de rotação regular. Algumas access keys eram compartilhadas informalmente entre membros da equipe via Slack.

---

## 2. Cronologia do Incidente

### Dia 1 - Sexta-feira, 13 de dezembro de 2025

**09:30 BRT**: Um desenvolvedor júnior recebe email de phishing sofisticado aparentemente originado do provedor de email corporativo, solicitando "verificação urgente de credenciais AWS devido a atividade suspeita". O email inclui link para página de login falsificada visualmente idêntica ao console AWS.

**09:45 BRT**: O desenvolvedor, preocupado com possível comprometimento de segurança, clica no link e insere suas credenciais AWS (usuário: `dev-junior-01`, senha). A página falsificada captura as credenciais e as transmite para servidor controlado pelos atacantes.

**10:15 BRT**: Atacantes testam as credenciais comprometidas e confirmam acesso válido. Descobrem que o usuário possui política IAM com permissões administrativas completas e não tem MFA habilitado.

**10:30 BRT**: Atacantes iniciam reconnaissance da infraestrutura, listando buckets S3, instâncias EC2, bancos de dados RDS e usuários IAM. Todas estas operações são logadas no CloudTrail mas não geram alertas.

**11:00 BRT**: Atacantes identificam bucket `technova-prod-backups` contendo backups de banco de dados com dados sensíveis de clientes. Iniciam download de múltiplos arquivos de backup totalizando 50GB.

**14:00 BRT**: Atacantes criam novo usuário IAM (`system-backup-user`) com permissões administrativas como mecanismo de persistência. Este usuário é criado sem MFA e com access keys programáticas.

**15:30 BRT**: Atacantes modificam política de bucket S3 `technova-prod-backups` para permitir acesso público, facilitando exfiltração contínua de dados.

**18:00 BRT**: Equipe técnica encerra expediente de sexta-feira sem ter detectado qualquer atividade anômala. Atacantes continuam operando durante o fim de semana.

### Dia 2-3 - Final de Semana (14-15 de dezembro)

Durante o fim de semana, sem monitoramento ativo, atacantes expandem suas atividades. Acessam múltiplos buckets S3 e exfiltram aproximadamente 200GB de dados incluindo backups de banco de dados, código-fonte de aplicações proprietárias, documentos internos e logs contendo informações sensíveis.

Criam duas access keys adicionais para o usuário comprometido original como backup. Exploram instâncias EC2 de produção, obtendo acesso a variáveis de ambiente contendo credenciais de banco de dados e chaves de API de serviços de terceiros.

Modificam configurações de logging para reduzir visibilidade de suas atividades, desabilitando logs de acesso em alguns buckets S3.

### Dia 4 - Segunda-feira, 16 de dezembro de 2025

**09:00 BRT**: Equipe técnica retorna ao trabalho. Nenhuma anomalia é imediatamente aparente pois sistemas continuam operando normalmente.

**11:30 BRT**: Analista de custos nota aumento significativo em custos de transferência de dados S3 no dashboard de billing. Custos de data transfer out aumentaram de média de R$ 500/dia para R$ 3.500 no fim de semana.

**12:00 BRT**: Analista de custos reporta anomalia ao time de infraestrutura. Engenheiro de DevOps inicia investigação revisando logs do CloudWatch.

**13:30 BRT**: Investigação de logs do CloudTrail revela atividades suspeitas: criação de usuário IAM não autorizado, modificações em políticas de bucket S3, e volume massivo de operações `GetObject` em buckets de backup.

**14:00 BRT**: Incidente é escalado para CTO e equipe de segurança. Processo de resposta a incidentes é ativado.

**14:30 BRT**: Equipe confirma comprometimento de credenciais e acesso não autorizado a dados sensíveis. Decisão é tomada de iniciar contenção imediata.

---

## 3. Investigação e Análise de Causa Raiz

### Vetor de Ataque Inicial

A análise forense determinou que o vetor de ataque inicial foi **phishing direcionado** (spear phishing) contra desenvolvedor júnior. O email de phishing demonstrava conhecimento da organização e utilizava engenharia social sofisticada para criar senso de urgência. A página de login falsificada era tecnicamente bem executada, utilizando certificado SSL válido e domínio similar ao legítimo (`aws-console-signin.com` ao invés de `aws.amazon.com`).

O sucesso do ataque de phishing foi facilitado por ausência de treinamento adequado de security awareness. O desenvolvedor não havia recebido orientação sobre como identificar tentativas de phishing e não sabia que AWS nunca solicita credenciais via email.

### Vulnerabilidades Exploradas

**Permissões Administrativas Excessivas**: A vulnerabilidade crítica que permitiu o impacto severo foi a política IAM com permissões administrativas completas anexada ao usuário comprometido. Esta política, exemplificada no arquivo `insecure-admin.json` deste repositório, concedia `Action: *` e `Resource: *`, permitindo aos atacantes executar qualquer operação em qualquer recurso.

Se o usuário tivesse apenas permissões granulares necessárias para suas funções de desenvolvimento (acesso a recursos de desenvolvimento, sem acesso a produção), o impacto do comprometimento teria sido drasticamente limitado. Atacantes não teriam conseguido acessar buckets de produção, criar novos usuários, ou modificar configurações de segurança.

**Ausência de MFA**: A falta de autenticação multi-fator significou que o comprometimento da senha foi suficiente para acesso completo. Se MFA estivesse habilitado, atacantes precisariam também comprometer o dispositivo MFA do usuário, barreira significativamente mais difícil de superar.

**Ausência de Segregação de Ambientes**: Não havia controles que impedissem usuários de desenvolvimento de acessar recursos de produção. Políticas IAM não utilizavam condições baseadas em tags de ambiente ou ARNs específicos. Esta falta de segregação permitiu que atacantes, usando credenciais de desenvolvedor, acessassem dados críticos de produção.

**Ausência de Monitoramento Proativo**: CloudTrail estava habilitado mas logs não eram analisados em tempo real. Não havia alarmes configurados para eventos críticos como criação de usuários IAM, modificações em políticas de bucket S3, ou volume anômalo de downloads de S3. Esta ausência de monitoramento permitiu que atacantes operassem por 4 dias sem detecção.

**Ausência de Controles de Acesso em Buckets S3**: Bucket `technova-prod-backups` contendo dados sensíveis não tinha controles adicionais além de políticas IAM. Não havia criptografia adicional, versionamento habilitado, ou MFA Delete configurado. Isto facilitou exfiltração massiva de dados.

### Análise de Logs

Análise forense de logs do CloudTrail revelou cronologia detalhada das atividades dos atacantes:

Mais de 5.000 operações `s3:GetObject` foram executadas contra bucket `technova-prod-backups` durante o período de comprometimento, totalizando 200GB de dados exfiltrados. Operações `iam:CreateUser`, `iam:CreateAccessKey`, e `iam:AttachUserPolicy` indicaram criação de mecanismos de persistência. Operações `s3:PutBucketPolicy` mostraram modificações em políticas de bucket para facilitar acesso.

Análise de endereços IP de origem revelou que ataques foram originados de múltiplas localizações geográficas (Europa Oriental e Sudeste Asiático), indicando uso de VPNs ou proxies para ofuscar localização real dos atacantes.

---

## 4. Impactos Identificados

### Impacto em Dados

**Exposição de Dados de Clientes**: Aproximadamente 150.000 registros de clientes foram expostos, incluindo nomes completos, endereços, números de telefone, emails, e histórico de compras. Embora dados de cartões de crédito não estivessem armazenados (processamento via gateway de pagamento terceirizado), informações de perfil de consumo e preferências foram comprometidas.

**Exposição de Propriedade Intelectual**: Código-fonte de aplicações proprietárias foi exfiltrado, incluindo algoritmos de recomendação de produtos, lógica de precificação dinâmica, e integrações customizadas. Esta propriedade intelectual representa vantagem competitiva significativa da empresa.

**Exposição de Credenciais**: Logs e backups continham credenciais de banco de dados, chaves de API de serviços de terceiros (gateways de pagamento, serviços de email, analytics), e tokens de acesso. Todas estas credenciais precisaram ser rotacionadas emergencialmente.

### Impacto Financeiro

**Custos Diretos**: Custos de transferência de dados AWS durante exfiltração: R$ 8.500. Custos de consultoria de segurança externa para investigação forense e remediação: R$ 45.000. Custos de serviços de monitoramento de crédito oferecidos a clientes afetados: R$ 120.000 (estimado para 12 meses).

**Custos Indiretos**: Perda de produtividade durante investigação e remediação: aproximadamente 500 horas-homem da equipe técnica. Desenvolvimento de novos recursos atrasado em 6 semanas. Custos de relações públicas e gestão de crise: R$ 25.000.

**Custos Potenciais**: Multas regulatórias potenciais por violação da LGPD (Lei Geral de Proteção de Dados): sob investigação da ANPD, podendo chegar a 2% do faturamento anual. Processos judiciais de clientes afetados: risco não quantificado.

### Impacto Reputacional

Notícia do incidente foi divulgada em mídia especializada em tecnologia após notificação obrigatória à ANPD. Avaliações negativas em redes sociais e sites de reclamação aumentaram 300% na semana seguinte ao incidente. Taxa de conversão de visitantes em compradores caiu 25% no mês seguinte, indicando perda de confiança de clientes potenciais. Duas parcerias comerciais em negociação foram suspensas até conclusão de auditoria de segurança independente.

### Impacto Operacional

Sistemas foram parcialmente desligados durante investigação e remediação, causando indisponibilidade de 6 horas. Todas as credenciais (senhas de usuários IAM, access keys, credenciais de banco de dados, chaves de API) precisaram ser rotacionadas emergencialmente, causando interrupções em integrações e processos automatizados. Equipe técnica trabalhou em regime de urgência por 2 semanas, causando burnout e impactando moral.

---

## 5. Medidas de Contenção Imediata

### Fase 1: Contenção (Primeiras 2 horas)

**14:30 BRT**: Desabilitação imediata do usuário IAM comprometido (`dev-junior-01`) e do usuário criado pelos atacantes (`system-backup-user`). Revogação de todas as access keys associadas.

**15:00 BRT**: Rotação de senha da conta root AWS e confirmação de que MFA estava habilitado (estava, felizmente). Revisão de todos os usuários IAM buscando por criações não autorizadas. Identificação e desabilitação de 3 usuários suspeitos adicionais.

**15:30 BRT**: Reversão de modificações em políticas de bucket S3, removendo permissões públicas não autorizadas. Habilitação de MFA Delete em buckets críticos para prevenir exclusões não autorizadas.

**16:00 BRT**: Isolamento de instâncias EC2 potencialmente comprometidas através de modificação de security groups, bloqueando todo tráfego de entrada e saída exceto para IPs administrativos específicos.

### Fase 2: Erradicação (Dias 1-3)

Rotação emergencial de todas as credenciais de banco de dados RDS. Atualização de aplicações com novas credenciais. Rotação de todas as chaves de API de serviços de terceiros (gateways de pagamento, serviços de email, analytics, etc.). Coordenação com fornecedores para revogação de tokens antigos.

Análise forense completa de instâncias EC2 para identificar possíveis backdoors ou malware. Decisão de terminar e reconstruir instâncias potencialmente comprometidas ao invés de tentar limpeza. Reconstrução de instâncias EC2 de produção a partir de AMIs limpas conhecidas, com configurações de segurança reforçadas.

Revisão completa de todos os usuários IAM, políticas e roles. Remoção de permissões administrativas desnecessárias. Implementação de políticas granulares baseadas no princípio do menor privilégio.

### Fase 3: Recuperação (Semana 1-2)

Restauração gradual de serviços após confirmação de erradicação de acesso não autorizado. Monitoramento intensivo de todas as atividades durante período de recuperação. Implementação de controles de segurança adicionais antes de restauração completa de operações.

Comunicação transparente com clientes afetados, conforme exigido pela LGPD. Oferta de serviços de monitoramento de crédito. Notificação formal à ANPD (Autoridade Nacional de Proteção de Dados) dentro do prazo legal.

---

## 6. Medidas Corretivas Implementadas

### Remediação de Vulnerabilidades Identificadas

**Eliminação de Permissões Administrativas Excessivas**: Todas as políticas com `Action: *` e `Resource: *` foram removidas. Políticas granulares baseadas no princípio do menor privilégio foram criadas para cada função organizacional, conforme exemplificado em `developer-least-privilege.json` neste repositório. Desenvolvedores agora possuem acesso apenas a recursos de desenvolvimento, com negação explícita de acesso a produção.

**MFA Obrigatório**: Autenticação multi-fator foi implementada obrigatoriamente para todos os usuários. Política IAM foi criada que nega todas as operações exceto habilitação de MFA para usuários que não o configuraram. Condições `aws:MultiFactorAuthPresent: true` foram adicionadas a todas as políticas com permissões sensíveis. Hardware MFA tokens foram adquiridos para administradores e usuários com acesso a produção.

**Segregação de Ambientes**: Todos os recursos AWS foram tagueados obrigatoriamente com tag `Environment` (production, staging, development). Políticas IAM foram modificadas para incluir condições baseadas em tags, impedindo que usuários de desenvolvimento acessem recursos de produção. Grupos IAM distintos foram criados para cada ambiente: `Developers-Dev`, `Developers-Staging`, `Engineers-Production`.

**Implementação de Monitoramento Proativo**: AWS GuardDuty foi habilitado em todas as regiões para detecção de ameaças baseada em machine learning. AWS Security Hub foi configurado para centralização de descobertas de segurança. Alarmes no CloudWatch foram criados para eventos críticos de IAM: criação de usuários, modificação de políticas, criação de access keys, tentativas de login falhadas, uso de credenciais root, e volume anômalo de operações em S3.

**Reforço de Segurança em S3**: Criptografia em repouso (SSE-KMS) foi habilitada em todos os buckets contendo dados sensíveis. Versionamento foi habilitado em buckets críticos para permitir recuperação de objetos deletados ou modificados. MFA Delete foi configurado em buckets de produção, exigindo MFA para exclusão de objetos ou desabilitação de versionamento. Políticas de bucket foram revisadas e restritas, removendo permissões públicas desnecessárias.

### Melhorias de Processo

**Gestão de Identidades**: Eliminação de todas as contas compartilhadas, com criação de usuários individuais para cada pessoa. Implementação de política de rotação obrigatória de access keys a cada 90 dias. Migração para uso de IAM roles ao invés de access keys para aplicações rodando em AWS. Implementação de processo formal de solicitação de acesso com aprovação de gestor e revisão de segurança.

**Treinamento de Segurança**: Programa de security awareness foi implementado, incluindo treinamento obrigatório anual sobre phishing, gestão de credenciais, e boas práticas de segurança. Simulações de phishing são realizadas trimestralmente para avaliar efetividade do treinamento. Desenvolvedores receberam treinamento específico sobre segurança em cloud e princípios de IAM.

**Resposta a Incidentes**: Runbooks de resposta a incidentes foram criados para cenários comuns (comprometimento de credenciais, acesso não autorizado, exfiltração de dados). Equipe de resposta a incidentes foi formalmente designada com papéis e responsabilidades claros. Simulações de resposta a incidentes (tabletop exercises) são realizadas semestralmente.

**Auditoria e Compliance**: Processo de revisão trimestral de permissões foi estabelecido, onde gestores devem aprovar formalmente os acessos de suas equipes. Auditorias de segurança são realizadas semestralmente por consultoria externa independente. Conformidade com LGPD e frameworks de segurança (CIS AWS Foundations Benchmark) é verificada continuamente através do Security Hub.

---

## 7. Lições Aprendidas

### Princípios de Segurança Fundamentais

**O Princípio do Menor Privilégio Não É Opcional**: A lição mais crítica deste incidente é que permissões administrativas excessivas transformam comprometimentos simples em desastres. Se o usuário comprometido tivesse apenas as permissões necessárias para suas funções de desenvolvimento, o impacto teria sido mínimo. O princípio do menor privilégio não é uma "boa prática opcional", mas um requisito fundamental de segurança.

**MFA É Barreira Crítica**: Autenticação multi-fator representa camada adicional de segurança que teria prevenido este incidente completamente, mesmo com senha comprometida. MFA deve ser obrigatório, não opcional, especialmente para usuários com permissões elevadas.

**Monitoramento Sem Análise É Inútil**: Ter CloudTrail habilitado não forneceu proteção porque logs não eram analisados proativamente. Monitoramento efetivo requer não apenas coleta de logs, mas análise automatizada e alertas em tempo real para atividades suspeitas.

**Segregação de Ambientes É Essencial**: Desenvolvedores não devem ter acesso a produção. Esta segregação deve ser forçada tecnicamente através de políticas IAM, não apenas através de políticas organizacionais que dependem de comportamento humano.

### Aspectos Organizacionais

**Segurança Requer Investimento Contínuo**: A TechNova Solutions havia priorizado velocidade de desenvolvimento sobre segurança durante fase de crescimento rápido. Configurações inseguras implementadas por conveniência nunca foram revisadas. Segurança não pode ser "deixada para depois" - deve ser integrada desde o início.

**Treinamento É Investimento, Não Custo**: A ausência de treinamento de security awareness foi fator contribuinte crítico. Desenvolvedores não sabiam como identificar phishing ou gerenciar credenciais adequadamente. Investimento em treinamento teria prevenido o vetor de ataque inicial.

**Resposta Rápida Limita Impacto**: Embora o incidente tenha causado danos significativos, a resposta estruturada e rápida após detecção limitou impactos adicionais. Ter processo de resposta a incidentes documentado e equipe treinada foi crucial.

**Transparência Constrói Confiança**: A decisão de comunicar transparentemente com clientes afetados, embora difícil, foi importante para manutenção de confiança a longo prazo. Tentativas de ocultar violações frequentemente resultam em danos reputacionais maiores quando inevitavelmente descobertas.

### Aspectos Técnicos

**Defesa em Profundidade É Necessária**: Nenhum controle único é suficiente. Este incidente explorou múltiplas vulnerabilidades simultaneamente. Estratégia de defesa em profundidade com múltiplas camadas de controles (MFA, menor privilégio, segregação, monitoramento) torna ataques significativamente mais difíceis.

**Automação de Segurança É Essencial**: Processos manuais de revisão de permissões e análise de logs são propensos a falhas. Automação através de ferramentas como GuardDuty, Security Hub e alarmes do CloudWatch fornece detecção contínua e confiável.

**Assume Breach Mentality**: Organizações devem operar assumindo que comprometimentos ocorrerão. Controles devem ser implementados não apenas para prevenir comprometimentos, mas para limitar impacto quando ocorrerem e detectá-los rapidamente.

---

## 8. Recomendações para Outras Organizações

Organizações que operam em AWS devem aprender com este incidente e implementar proativamente controles de segurança adequados antes de sofrerem incidentes similares.

**Auditoria Imediata de Políticas IAM**: Revisar todas as políticas IAM buscando por uso de wildcards (`Action: *`, `Resource: *`). Substituir políticas genéricas por políticas granulares baseadas no princípio do menor privilégio. Utilizar ferramentas como IAM Access Analyzer para identificar permissões excessivas.

**Implementação de MFA Obrigatório**: Habilitar MFA para todos os usuários, especialmente aqueles com permissões elevadas. Implementar políticas IAM que exijam MFA para operações sensíveis. Considerar hardware MFA para administradores.

**Habilitação de Serviços de Segurança AWS**: Ativar GuardDuty para detecção de ameaças. Configurar Security Hub para centralização de descobertas. Criar alarmes no CloudWatch para eventos críticos. Estas ferramentas fornecem detecção automatizada de atividades maliciosas.

**Segregação de Ambientes**: Implementar separação rigorosa entre desenvolvimento, staging e produção através de tags e condições IAM. Considerar uso de contas AWS separadas para cada ambiente, gerenciadas via AWS Organizations.

**Treinamento de Equipe**: Implementar programa de security awareness com foco em phishing, gestão de credenciais e boas práticas de segurança. Realizar simulações periódicas para avaliar efetividade.

**Estabelecimento de Processos**: Criar processos formais de solicitação de acesso, revisão de permissões, resposta a incidentes e auditoria de segurança. Documentar runbooks e realizar simulações.

---

## 9. Conclusão

O incidente de segurança da TechNova Solutions demonstra como configurações inadequadas de IAM, aparentemente simples, podem resultar em consequências severas. Permissões administrativas excessivas, ausência de MFA, falta de segregação de ambientes e monitoramento inadequado criaram condições perfeitas para que um ataque de phishing relativamente simples resultasse em comprometimento massivo de dados.

As medidas corretivas implementadas - eliminação de permissões excessivas, MFA obrigatório, segregação de ambientes, monitoramento proativo e treinamento de equipe - representam implementação de princípios fundamentais de segurança que deveriam ter sido estabelecidos desde o início. O custo financeiro, reputacional e operacional do incidente foi significativamente maior do que teria sido o investimento em implementação proativa destes controles.

Este caso serve como alerta para organizações que operam em cloud: segurança não pode ser tratada como prioridade secundária ou "deixada para depois". Os princípios de menor privilégio, defesa em profundidade, e monitoramento contínuo não são luxos ou "boas práticas opcionais", mas requisitos fundamentais para operação segura em ambientes cloud modernos.

A boa notícia é que as vulnerabilidades que permitiram este incidente são completamente evitáveis através de conhecimento adequado, processos estruturados e uso apropriado de ferramentas disponíveis. Organizações que aprendem com incidentes como este e implementam proativamente controles adequados podem operar com confiança em cloud, aproveitando seus benefícios enquanto gerenciam riscos efetivamente.

---

## Anexos

### Cronologia Consolidada

| Data/Hora | Evento |
|-----------|--------|
| 13/12 09:30 | Desenvolvedor recebe email de phishing |
| 13/12 09:45 | Credenciais comprometidas |
| 13/12 10:15 | Atacantes confirmam acesso |
| 13/12 10:30 | Início de reconnaissance |
| 13/12 11:00 | Início de exfiltração de dados |
| 13/12 14:00 | Criação de usuário backdoor |
| 13/12 15:30 | Modificação de políticas S3 |
| 14-15/12 | Exfiltração contínua durante fim de semana |
| 16/12 11:30 | Anomalia de custos detectada |
| 16/12 13:30 | Confirmação de comprometimento |
| 16/12 14:00 | Ativação de resposta a incidentes |
| 16/12 14:30 | Início de contenção |

### Métricas de Impacto

- **Dados Expostos**: 200GB (150.000 registros de clientes)
- **Duração do Comprometimento**: 4 dias
- **Tempo de Detecção**: 76 horas
- **Tempo de Contenção**: 2 horas após detecção
- **Custo Direto**: R$ 173.500
- **Horas de Trabalho de Remediação**: 500 horas-homem
- **Indisponibilidade de Sistemas**: 6 horas

---

**Documento elaborado por:** Rafael Garcez  
**Escola da Nuvem:** Turma BRASAO 227  
**Data:** Janeiro 2026

**Nota**: Este é um caso fictício criado para fins educacionais. Qualquer semelhança com eventos reais é coincidência.
