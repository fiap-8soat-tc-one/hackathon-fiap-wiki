# Hack SOAT 8 - Desafio de Arquitetura de Software

## 🏢 Contexto

A empresa **FIAP X** apresentou a investidores um protótipo simples que:
- Processa um vídeo enviado.
- Retorna as imagens extraídas dentro de um arquivo `.zip`.

Os investidores gostaram da proposta e agora desejam uma versão mais robusta, onde:
- Usuários possam **enviar vídeos**.
- Possam **baixar o resultado (arquivo zip)** via uma aplicação funcional.

📎 Projeto utilizado na apresentação:
[Link para o vídeo](https://drive.google.com/file/d/1aYCnARmf1KMvRs_HUishp8LUYL_yPlMA/view?usp=sharing)

---

## 🎯 Desafio Principal

Refatorar e evoluir o protótipo utilizando boas práticas de arquitetura de software aprendidas no curso, como:

- Desenho de Arquitetura
- Microsserviços
- Qualidade de Software (testes)
- Mensageria

---

## ✅ Requisitos Funcionais

- O sistema deve processar **mais de um vídeo ao mesmo tempo**.
- Em caso de **picos**, o sistema **não deve perder requisições**.
- O sistema deve ser protegido por **usuário e senha**.
- Deve haver **listagem de status** dos vídeos de um usuário.
- Em caso de erro, o usuário deve ser **notificado** (e-mail ou outro meio).

---

## 🛠️ Requisitos Técnicos

- O sistema deve **persistir os dados**.
- Deve possuir arquitetura **escalável**.
- O projeto deve ser **versionado no GitHub**.
- Deve conter **testes automatizados** para garantir qualidade.
- Deve possuir **pipeline de CI/CD**.

---

## 📦 Entregáveis

- 📄 Documentação da arquitetura proposta.
- 🛠️ Script de criação do banco de dados ou recursos necessários.
- 🔗 Link para o repositório GitHub com o projeto.
- 🎥 Vídeo de no máximo 10 minutos apresentando:
  - Documentação
  - Arquitetura escolhida
  - Projeto em funcionamento

## 🔧 Infraestrutura

Este repositório define a infraestrutura como código (IaC) para o projeto do Hackathon FIAP, utilizando serviços da AWS com foco em escalabilidade, segurança e alta disponibilidade. Abaixo, os principais componentes descritos no README:

---

### ☸️ Amazon EKS (Elastic Kubernetes Service)
- Serviço gerenciado de Kubernetes.
- Elimina a gestão manual do control plane.
- Suporta workloads em EC2, Fargate e Graviton.
- Integração com IAM para controle de acesso.
- Observabilidade com CloudWatch.
- **Justificativa**: alta disponibilidade, escalabilidade e segurança nativa.

---

### 🔐 Amazon Cognito
- Gerenciamento de autenticação e usuários.
- Suporte a login com e-mail/senha ou redes sociais.
- MFA (autenticação multifator) e triggers customizáveis.
- Integra-se diretamente com API Gateway e IAM.
- **Justificativa**: autenticação serverless, escalável e integrada com serviços AWS.

---

### 🌐 Amazon API Gateway
- Serviço para expor APIs REST, HTTP e WebSocket.
- Suporte nativo a autenticação via Cognito, IAM e JWT.
- Recursos como caching, versionamento, CORS e throttling.
- **Justificativa**: criação rápida de APIs com segurança e escalabilidade, totalmente gerenciado.

---

### 🧠 Amazon SageMaker
- Plataforma para treinar, testar e hospedar modelos de Machine Learning.
- Suporte a notebooks Jupyter e integração com CI/CD de ML.
- **Justificativa**: simplifica o deployment de modelos, integrando com os demais serviços AWS.

---

### 📦 Amazon S3 (Simple Storage Service)
- Armazenamento de objetos escalável e durável.
- Utilizado para guardar vídeos, imagens extraídas e arquivos `.zip`.
- Integração com CloudFront, Lambda, eventos e controle de acesso por políticas.
- **Justificativa**: armazenamento seguro e escalável para dados não estruturados.

---

### 🐙 GitHub Actions
- Utilizado para CI/CD da aplicação.
- Automatiza testes, build, deploy e integração com a AWS.
- **Justificativa**: pipeline ágil e versionado, totalmente integrado ao GitHub.

---

### ⚙️ Terraform
- Ferramenta de IaC usada para provisionar todos os recursos.
- Facilita a reprodutibilidade e versionamento da infraestrutura.
- **Justificativa**: automatização e consistência no provisionamento de ambientes AWS.

[Clique aqui para ser redirecionado para a wiki de infra com os detalhamentos e comparação entre as técnologias](https://github.com/fiap-8soat-tc-one/hackathon-fiap-iac)

