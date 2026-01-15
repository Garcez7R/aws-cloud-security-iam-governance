# AWS Cloud Security – IAM & Governança

![AWS](https://img.shields.io/badge/AWS-Cloud%20Security-orange?style=flat-square&logo=amazon-aws)
![IAM](https://img.shields.io/badge/IAM-Identity%20%26%20Access%20Management-blue?style=flat-square)
![Security](https://img.shields.io/badge/Security-Best%20Practices-green?style=flat-square)
![Status](https://img.shields.io/badge/status-educational-blue)


## 📋 Sobre o Projeto

Este projeto demonstra conhecimentos fundamentais de **segurança em cloud computing** através da análise, implementação e documentação de políticas de **AWS Identity and Access Management (IAM)** e práticas de governança corporativa.

O repositório simula um cenário corporativo real, comum em times de segurança, auditoria e governança, onde é necessário identificar riscos, aplicar o princípio do menor privilégio e documentar decisões técnicas de forma clara e profissional.

**Autor:** Rafael Garcez  
**LinkedIn:** [linkedin.com/in/rgarcez7](https://www.linkedin.com/in/rgarcez7/)  
**Escola da Nuvem:** Turma BRASAO 227  
**Disciplina:** Competências Profissionais

---

## 🎯 Objetivos de Aprendizagem

Este projeto demonstra competências em:

- **Análise de Políticas IAM**: Identificação de permissões excessivas e vulnerabilidades de segurança
- **Princípio do Menor Privilégio**: Implementação de controles de acesso baseados em necessidade real
- **Governança em Cloud**: Aplicação de frameworks e boas práticas de segurança corporativa
- **Documentação Técnica**: Comunicação clara de decisões de segurança e análise de riscos
- **Pensamento Crítico**: Análise de incidentes e proposição de medidas corretivas

---

## 🏢 Cenário Simulado

**Empresa:** TechNova Solutions  
**Segmento:** E-commerce em crescimento  
**Infraestrutura:** AWS Cloud (S3, EC2, RDS, CloudWatch)  
**Desafio:** Revisar e corrigir políticas IAM após auditoria de segurança

A TechNova Solutions identificou vulnerabilidades em suas políticas de acesso após um incidente de segurança. Este projeto documenta a análise realizada, os riscos identificados e as correções implementadas seguindo as melhores práticas da AWS.

---

## 📁 Estrutura do Repositório

```
aws-cloud-security-iam-governance/
├── README.md                          # Documento principal do projeto
├── PROJECT_GUIDE.md                   # Guia detalhado de implementação
├── policies/                          # Políticas IAM em formato JSON
│   ├── insecure-admin.json           # Exemplo de política insegura
│   ├── developer-least-privilege.json # Política corrigida (least privilege)
│   └── read-only.json                # Política de acesso somente leitura
├── analysis/                          # Análises de segurança
│   ├── risk-analysis.md              # Análise de riscos IAM
│   ├── least-privilege.md            # Explicação do princípio
│   ├── common-iam-mistakes.md        # Erros comuns em IAM
│   └── security-checklist.md         # Checklist de boas práticas
├── case/                              # Caso prático
│   └── security-incident-case.md     # Estudo de caso de incidente
└── diagrams/                          # Diagramas de arquitetura
    └── iam-architecture.png          # Arquitetura IAM visual
```

---

## 🔐 Tecnologias e Conceitos

### Tecnologias AWS
- **IAM (Identity and Access Management)**: Gerenciamento de identidades e acessos
- **S3 (Simple Storage Service)**: Armazenamento de objetos
- **EC2 (Elastic Compute Cloud)**: Servidores virtuais
- **RDS (Relational Database Service)**: Banco de dados gerenciado
- **CloudWatch**: Monitoramento e logs

### Conceitos de Segurança
- **Least Privilege**: Princípio do menor privilégio
- **Zero Trust**: Modelo de segurança baseado em verificação contínua
- **Defense in Depth**: Defesa em camadas
- **Separation of Duties**: Segregação de funções
- **IAM Policies**: Políticas baseadas em JSON
- **Resource-Based Permissions**: Permissões baseadas em recursos

---

## 🚀 Navegação do Repositório

Este repositório está organizado para facilitar a compreensão progressiva do projeto, desde o contexto do problema até as decisões técnicas de segurança adotadas.

- **README.md**: visão geral do projeto, objetivos e contexto
- **case/security-incident-case.md**: estudo de caso que simula um incidente real envolvendo falhas de IAM
- **policies/**: exemplos práticos de políticas IAM, incluindo cenários inseguros e versões corrigidas
- **analysis/**: análises de risco, boas práticas e erros comuns relacionados a IAM
- **diagrams/**: representação visual da arquitetura e dos controles de acesso
- **PROJECT_GUIDE.md**: documentação detalhada do processo e da metodologia utilizada

---

## 📊 Principais Entregas

### 1. Políticas IAM Documentadas
Três exemplos práticos de políticas IAM:
- Política insegura com permissões excessivas (anti-pattern)
- Política corrigida seguindo least privilege
- Política read-only para auditoria e monitoramento

### 2. Análise de Riscos
Identificação e classificação de riscos associados a configurações inadequadas de IAM, incluindo:
- Impacto potencial
- Probabilidade de ocorrência
- Medidas de mitigação

### 3. Caso de Incidente Real
Simulação de incidente de segurança causado por falhas de IAM, incluindo:
- Contexto e cronologia
- Análise de causa raiz
- Medidas corretivas implementadas
- Lições aprendidas

### 4. Documentação de Boas Práticas
Compilação de erros comuns, recomendações e checklist de segurança para implementação em ambientes reais.

---

## 🎓 Aprendizados e Competências

Este projeto reflete atividades comuns em funções de:
- **Cloud Security Engineer**
- **DevSecOps Engineer**
- **Cloud Governance Analyst**
- **Security Auditor**
- **IAM Specialist**

Competências demonstradas:
- ✅ Leitura e interpretação de políticas IAM em JSON
- ✅ Identificação de vulnerabilidades de segurança
- ✅ Aplicação de frameworks de segurança (AWS Well-Architected)
- ✅ Documentação técnica clara e estruturada
- ✅ Pensamento analítico e resolução de problemas
- ✅ Comunicação de conceitos técnicos complexos

---

## 📚 Referências e Recursos

- [AWS IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [AWS Well-Architected Framework - Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html)
- [Princípio do Menor Privilégio (NIST)](https://csrc.nist.gov/glossary/term/least_privilege)
- [AWS Security Documentation](https://docs.aws.amazon.com/security/)
- [CIS AWS Foundations Benchmark](https://www.cisecurity.org/benchmark/amazon_web_services)

---

## 📝 Observações

Este projeto **não requer acesso ativo à AWS Console**. Ele foi desenvolvido como exercício de análise, documentação e demonstração de conhecimentos teóricos e práticos em segurança cloud, perfeitamente alinhado com o nível **iniciante-júnior** e com o conteúdo da Escola da Nuvem – AWS.

Todos os exemplos são baseados em cenários reais de mercado, mas utilizam dados fictícios e não representam configurações de ambientes produtivos reais.

---

## 📞 Contato

**Rafael Garcez**  
LinkedIn: [linkedin.com/in/rgarcez7](https://www.linkedin.com/in/rgarcez7/)  
Escola da Nuvem - Turma BRASAO 227

---

## 📄 Licença

Este projeto foi desenvolvido para fins educacionais como parte do programa da Escola da Nuvem – AWS.

---

**⭐ Se este projeto foi útil para você, considere deixar uma estrela no repositório!**
