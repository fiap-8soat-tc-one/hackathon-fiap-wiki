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

Este repositório define a infraestrutura como código (IaC) para o projeto do Hackathon FIAP, utilizando serviços da AWS com foco em escalabilidade, segurança, resiliência e alta disponibilidade. Abaixo, os principais componentes da arquitetura:

---

### 🌐 Amazon API Gateway
- Serviço para criar, publicar, manter, monitorar e proteger APIs em qualquer escala (REST, HTTP, WebSocket).
- Atua como "porta de entrada" para as requisições dos usuários.
- Suporte nativo a autenticação via Cognito, IAM e JWT.
- Recursos como caching, versionamento, CORS e throttling (controle de picos).
- **Justificativa**: Criação rápida de APIs seguras e escaláveis, totalmente gerenciado, integração nativa com Lambda e Cognito.

---

### 🔐 Amazon Cognito
- Serviço de identidade para adicionar autenticação, autorização e gerenciamento de usuários a aplicações web e móveis.
- Responsável pela **gestão dos usuários** (cadastro, login com e-mail/senha ou redes sociais) e pela **validação/gestão dos tokens** de autenticação/autorização.
- Suporte a MFA (autenticação multifator) e fluxos customizáveis com triggers Lambda.
- Integra-se diretamente com API Gateway e IAM para controle de acesso fino.
- **Justificativa**: Solução serverless robusta para identidade, escalável, segura e integrada com outros serviços AWS, atendendo ao requisito de proteção por usuário e senha.

---

### λ AWS Lambda
- Serviço de computação serverless que executa código em resposta a eventos.
- Utilizado para funções específicas e pontuais, como:
    - **Geração de URLs pré-assinadas (Presigned URLs)** para permitir que os usuários façam upload dos vídeos diretamente para o S3 de forma segura e temporária.
    - **Geração de tokens específicos** (se necessário complementar a lógica do Cognito para acessos temporários a recursos, por exemplo).
- Pode ser acionado por API Gateway, S3 Events, SQS, etc.
- **Justificativa**: Execução sob demanda, escalabilidade automática, sem gerenciamento de servidores, ideal para tarefas pontuais e integrações, otimizando custos.

---

### 📦 Amazon S3 (Simple Storage Service)
- Serviço de armazenamento de objetos altamente escalável, durável e seguro.
- Utilizado para:
    - Armazenar os **vídeos originais** enviados pelos usuários (via Presigned URL gerada pelo Lambda).
    - Armazenar as **imagens extraídas** durante o processamento.
    - Armazenar os **arquivos `.zip`** resultantes para download pelo usuário.
- Integração com CloudFront (CDN), Lambda (eventos de criação de objeto), e controle de acesso fino por políticas IAM e de bucket.
- **Justificativa**: Armazenamento virtualmente infinito, seguro, de baixo custo e escalável para dados não estruturados, central no fluxo de dados do projeto.

---

### 📨 Amazon SQS (Simple Queue Service)
- Serviço de fila de mensagens totalmente gerenciado para desacoplar e escalar microsserviços, aplicações distribuídas e sistemas serverless.
- Utilizado para a **troca de mensagens entre microsserviços**, garantindo o desacoplamento e a resiliência:
    - Ex: O serviço que recebe o upload (via API Gateway/Lambda) envia uma mensagem para uma fila SQS contendo informações sobre o vídeo a ser processado.
    - O(s) microsserviço(s) de processamento (rodando no EKS) consomem mensagens dessa fila para iniciar o trabalho.
- **Justificativa**: Essencial para atender aos requisitos de processamento concorrente e resiliência a picos. Garante que nenhuma requisição seja perdida, desacopla o front-end do back-end de processamento, e facilita o escalonamento baseado em eventos (com KEDA).

---

### 💾 Amazon DynamoDB
- Banco de dados NoSQL chave-valor e de documentos, totalmente gerenciado, que oferece performance de milissegundos de um dígito em qualquer escala.
- Utilizado para **persistir e consultar o estado e metadados do processamento dos vídeos**:
    - Armazena informações como ID do vídeo, ID do usuário, status (ex: `UPLOAD_CONCLUÍDO`, `PROCESSANDO`, `CONCLUÍDO`, `ERRO`), timestamp, link para o resultado no S3, etc.
    - Permite a implementação do requisito de **listagem de status** dos vídeos por usuário.
- **Justificativa**: Altíssima escalabilidade, performance consistente, modelo serverless (pay-as-you-go), ideal para controle de estado, metadados e catálogos que exigem baixa latência e alta concorrência.

---

### ☸️ Amazon EKS (Elastic Kubernetes Service)
- Serviço gerenciado de Kubernetes que facilita a execução, gerenciamento e escalonamento de aplicações containerizadas na AWS.
- Orquestra os contêineres dos microsserviços responsáveis pelo **processamento pesado dos vídeos**.
- Elimina a necessidade de gerenciar o control plane do Kubernetes.
- Suporta workloads em EC2, Fargate e Graviton.
- Integração nativa com IAM para controle de acesso, CloudWatch para observabilidade, e ECR para imagens.
- **Justificativa**: Plataforma robusta, escalável e padrão de mercado para orquestração de contêineres, garantindo alta disponibilidade e facilitando a gestão de microsserviços complexos.

---

### ☸️ Amazon ECR (Elastic Container Registry)
- Registro de imagens de contêiner Docker totalmente gerenciado, seguro e escalável.
- Utilizado para **armazenar as imagens Docker** dos microsserviços da aplicação (ex: serviço de processamento de vídeo).
- Integra-se perfeitamente com EKS, Fargate, Lambda (imagens de contêiner) e pipelines de CI/CD.
- **Justificativa**: Essencial para o fluxo de trabalho com contêineres, fornece um local seguro e gerenciado para as imagens, integrado ao ecossistema AWS e ao controle de acesso IAM.

---

### ☸️ KEDA (Kubernetes Event-driven Autoscaling)
- Componente de escalonamento automático baseado em eventos para Kubernetes (instalado no cluster EKS).
- Permite **escalar os pods dos microsserviços de processamento no EKS com base em eventos externos**, como o número de mensagens na fila SQS.
- Pode escalar os pods de 0 (quando não há trabalho) até N (conforme a demanda), otimizando custos e garantindo performance.
- **Justificativa**: Implementa o requisito de arquitetura escalável de forma eficiente e orientada a eventos. Garante que os recursos de processamento (pods no EKS) sejam provisionados apenas quando necessário, respondendo dinamicamente aos picos de upload de vídeos refletidos na fila SQS.

---

### ✉️ SendGrid (ou alternativa como AWS SES)
- Plataforma de comunicação e entrega de e-mail na nuvem.
- Utilizada para **notificar os usuários por e-mail** sobre o status do processamento do vídeo, especialmente em caso de erro, conforme requisito funcional.
- Pode ser acionada por um serviço (ex: Lambda ou um microsserviço no EKS) ao final do processamento ou ao detectar um erro.
- **Justificativa**: Serviço especializado e confiável para entrega de e-mails transacionais, com APIs fáceis de integrar, garantindo que as notificações cheguem à caixa de entrada do usuário. (AWS Simple Email Service - SES - é uma alternativa nativa da AWS).

---

### 🐙 GitHub Actions
- Plataforma de CI/CD integrada ao GitHub.
- Utilizada para **automatizar o pipeline de integração e entrega contínua (CI/CD)** da aplicação:
    - Execução de testes automatizados.
    - Build das imagens Docker e push para o ECR.
    - Deploy das aplicações no EKS.
    - Provisionamento/atualização da infraestrutura via Terraform.
- **Justificativa**: Pipeline ágil, versionado junto ao código, totalmente integrado ao GitHub, facilitando a automação e a qualidade do processo de desenvolvimento e deploy.

---

### ⚙️ Terraform
- Ferramenta de Infraestrutura como Código (IaC) open-source.
- Utilizada para **definir, provisionar e gerenciar toda a infraestrutura AWS** descrita acima de forma declarativa e versionada.
- Garante a consistência, reprodutibilidade e automação da criação e atualização dos ambientes (desenvolvimento, produção, etc.).
- **Justificativa**: Padrão de mercado para IaC multi-cloud, permite gerenciar a complexidade da infraestrutura de forma eficiente, segura e versionada no Git.

---

## Arquitetura

[Clique aqui para ser redirecionado ao desenho da arquitetura do projeto](https://viewer.diagrams.net/?tags=%7B%7D&lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=hackaton-arc-v2.drawio&dark=auto#R%3Cmxfile%20scale%3D%221%22%20border%3D%220%22%3E%3Cdiagram%20name%3D%22Draft%22%20id%3D%22jiXIkiBsy346j2COqR-b%22%3E7V1rd5u40%2F80OWf3hTlI4vrSsZM2z%2FaSbdp%2Fu6%2F2YKPYbDC4gHPpp38kbgZJYOJwcVLSNgUhMNbM%2FDQzGs2codnm8V1gbdcffRu7Z1C2H8%2FQ%2FAxC3dAU8h9teUpaoKnoScsqcOykTd433Di%2FcNIIstadY%2BMwbUuaIt93I2dbblz6noeXUanNCgL%2Fodzt1nftUsPWWuHSa9CGm6XlYq7bd8eO1kmrAfV9%2B3vsrNbZJwPNTK5srKxz%2BuBwbdn%2BQ6EJXZyhWeD7UXK0eZxhl45eeVwuK67mLxZgL2pyA55%2BfP%2F3zbm5eNx9eO%2FNzF%2Be%2FN8ko8a95e7Sb5y%2BbfSUDUHg7zwb06eAM3RuBcuUSoRE6Ny2wnV8jZ7cOq47810%2FiG9Elyr9Q9rDKPDv8vHTSMvKtcIwu8v3osJdyU9%2BV%2BGKFv%2FQuwPLdvD%2BLs%2F3cPqgm%2FS9QXaevCukt%2FEjlg7iPQ4i%2FFhoSkfwHfY3OAqeSJeMo1NipuxspOz9sGcNqOgoaVwX%2BALpipoyZcqQq%2FzZe5qRg5RszyChyREM24SF01M%2FiNb%2Byvcs92Lfer4nKSXAvs8H39%2Bmg%2FcfjqKndPSsXeSTpnW0cdOrPEWPHvzQ3wVLXPMF04Gj36qWRAF2rci5LwuuaLjTW699h7xKTlqAQIm2E9IiMUSLrGCFo%2FRGhm75mxxPSvU5wkgp97B2InyzteLxeyAQXKYSI4%2B3Kv1zlMTlYt4y8ZtL3kQBZpk%2BSFZ56TNMXviAgTqSPXiYYAT2t%2FTQ2cQzTZE89Ms7ZKqZus7KI20RFb689YO1wO61HzqR49OrCz%2BK%2FA3p4NIL59bybhUzQ5HA8Q%2FpEn%2FYNNwmMyLlFCs7uXUeKSHP0%2FeZr6OITqVTOhDwcml7iuSQyfTWIWwWSEvyifDStiKL%2FEfbQzpOzm6ycMkLTEJ%2F6VguHYZLjSDj5cqJ1rvFBEBD2nqr4VgFKQyrkEezrKJoPKdkba0zCuIY5Z0Tvd8tSNt0SQkccoxDvmxU5hcr5ZMlGRocCBho49h2jO8iXCgjBzvsLFl4wrVBGKQyMgxNgQzLimAClTuijPJ8ET5WQm18a%2B3cqFLyxTJaEmOx1N6TDn4QYnyXCuzDdkJkNaKUgpe7retbNhVcKEPCiJcyIcJlwn%2BTlPsmH4gCMPlf%2FBwpvF9NhpVfWWfYRNM5NgFIF0B91tg6n2gcn8yuyINm5Ei%2BdrbYdYgO%2BnvKsCaQYQB7lOEGRsyW6mzx56rn5K8sIfLSMzn%2Bp5IOs7jRNGiDpIJSe3wm6YDpTHlyxjQmPU1V0KrljeRvAxAJ73C0XKekPqg1cNjBMVo8Aji4uMfJQMQckiLb5nFF7Xdp6YRLH5jSLoxvYfRHWVZlndcTU%2BtrEKgwWaUQaLxJBjLbq4QUnRlkmeuiwI5f8Moh40OUKPmKIjjBaNnzya8ZGRKLIEdAjtNOT0OgSC1qdAHxJqvMC0xpYAoUAaDBrugm0ua1eL6mI0COV%2FT4W7g7m6GzKQgcP6TgT8%2FOpwuXqMdhdgN5geI9WbPt3LNNxW5%2FfMdEI7z86C8cQg6iGm63fx54ItXOiw99%2B6xjaIgReZOfflSBbpAhQ%2FuMo3Lj3oMLhoxg8PQjvT8%2B%2BYeeSGp2On8sXpw%2FFc%2BuceCQL08ZIG4UUM71H6Ye0TfTeSmhrxVEU%2BpVpQxEXXjOMmu%2BdNzc7dCGK%2BjHp0C1vvw1%2BwSe%2Fvu%2B%2BPDj4h%2B0yMyoxA2TzgCf%2FwV3YPv486%2BnSXT16RpffZlkPrGuXUaKrEpIN7Mfo8yYisI8sMJ7RAbUeip0S1WVmk8tO6r0FBEvK%2FobZm1%2FcpC8QauuLGhUgmm4tbwMsL7gnzsndGJA1WMgVTEF1fdfv17%2FcVMEv%2BJdJwFzrPO5oCvtDb1eEFFlzSVNHhgRlSZWdVG%2FZdVzgW4ORQq7qE3nGwHfjdfdk09gG0VtOt8IKmwBKDIFRG26yr8xezcQ3A2Yu2Pjwt9F1B6d5YtignUW8ueSkpTV8i%2FmOpBljs2L7sXSos9BQ6bCYGEFtOi3PmDYcsYOa8dYD6EiBTiZT66W9H3OyWlyVO6F78KuTGViixiSUXZFQhnxiq4BeNk0amajl8kmv8ow%2FX5DGi7%2Bor9n7i6kBBnWlXHImGxBjTQ5wpgS78gAJpAlgfmod%2BaPFE2cDC1K8ldclePFAElL19%2FZvJhfqoaKahZTS58xiG2vlbUpgZZvCpxMIFsBap8yp7LSKtDMq4wBYBrHmAOdL%2BYWNfjawISuVfiJVnZEQNaB3JLSPoGozM9Gyt%2BdauEq74oSaeFzvHV9%2Bm2uvNvAwoTldtGOeqtG7buBKwtpSjZDZAq4avSngAsNZZEra0gXxd4r8U8JhV61i6LWi1hEOCGJQD8Ax7rLABMLkXwhDt8EXgdYno9BM6A8AsrEw6WcxuxbioR5AQfiRydKRENX09N%2FCpf2kkFPSoLRls%2BvVzedsJ%2FWiQg825%2BmlN0naioxbc3MddJfB9BCI75r01toQlhb59%2BVFeEHOgqsITE3p7oCqg2JQWwHpELJKJMVKJCblIFsxD4WzogALZh3daJxQCGbXl%2BRTu%2FSIT9JNSzwowz16kJ%2Fm4UasjanGHrrwn5bZh8IJMMoGwaZQ6bIPppAp2sjAqUOLV8HZLjWZmFb%2F97uvGXyCUL%2Fg3JisGFCk1HlVVmwLg1F69KsPtQa3ZutpMz8ledENKZguiMvOmJG35hhymXA0JAgZq1XwADNJpt3OA5M%2BUpGjrTJ%2F%2Ff968g9vXOPCsvco5g87iAg4B6zBU3lswZ3D9bcXX2bLPAP89vsav1rkk0vzbcGHdiNUKCfKTcY%2B%2BIWoQKte9zsk4Ua5b5BqEsqv0agqEjSBfsOoKnmWkT75KkWblGcUFGOre12sl5Gk22wnCzDTaW01wYeJV0LzKD93NGtbOeLPO55skxINqW2YGB5YUYN8hC5cM2lQzaxreDuj2C1%2BIOuh5LW7L8%2Fk%2F%2FpFUgHn54UD%2F78M3mr9OOz95yst40dmPx6YkXsZUmVitk9LLP7keoVg2XQ0NULyGFWcQmWi2eVZTUJM0XMYnW6ksxGugpbhTGxZsWD9fhuyTQbrJhDg%2BkbL1OzmuPdboEDD5NRjfeg0BHYN8VDmtJhG6RLuZTIA%2BmKQNZ0Mur7HwYsNChYTUSCTScAkTFrYeOJGCZEkRg8JIig4%2FrzPI9nXARsT76lOkJxFL7XKXy5mG19eziTjLx5QcrKsWxQEWw%2BGEDKGuzcHPl85PPj%2BRwpAvdT%2F3zewAc18vnI5y%2Fgc4HdOwCfi3aUVW7heKYp5HiEkS1XaLUwptnExtvD5lndBpBRGk9WGg%2BYNnYSlCMSU87LNrzcAp2POR9Abpv5ykXWToiXAY5Gg%2Be3Fslc%2BBJuGGqONHRTMpRKWVNNNRehgripUMqWIIoCp3blIEai4MLfykEMVJUJIkeKJgnS0gziIc7yGTUEQ5GH2POjWA2p86WOoPfKQe%2BEXawGxbQaF6tyCkoHEi2UjS7WUfhei0kOdAAlw9z%2FlHcqQMVAop1R%2FQuaaMlzZPWR1dtjdaQKNwH2z%2BoNNlCPrD6y%2BktQnVw%2BCVYXrZ316WvNjJzR1%2Fp2BfJ1%2BVoPiS4wjNMQXdFy4FHu1tGweWsSdyKuVKDpoM6XqgFFkof3pWZ7k4be3SjMLZDuVMyOC1sOKzcqHk3ZrrfDmgQ6mUTvUJeBBJmsEU23xdLnmQoAuqbosmowKcDIZYZj2tslK%2BYj9Lb4qNUd5CfLlEA3ZJ4r4fFcGT%2BwRoHomy216mD14vz%2BeUu3orC54GyaChOHy8CJs9C72EmSUyQZVJPcRfMnz9rQ03lj3WHcvNLeRglIQLD4U%2BY2Q%2BCmh6ohKYLtLG2UahCyoPkKkHGPhj2nAugza0btZqODmT0zH3DneoIp0BPACxAZKYoECgE05W2mCBm6tN8E1nLyIR0yaomiq7Vvy96AFBMd%2BHqZalxxx4uzKIiF%2BkSSgjQXarmZUIO3INRZgMjJCLWhsrsHDCYJfGNxVnRDYpT9bmUYyprOvHzK%2FVXvyN1xWIpVUH9HR1J8vBvpekfzrlNlENO6Bn6mLnp%2B5NzSC7kyOUX5xXC3xGFIT%2F0dvZOICL1Ef20Dn16zNsnDXuSc6lbBrFcon1nqa8icC7qmsGIEOXWRLkELkqQCuQYDXqYumgKeZAicecFuXfyY4ixBYpuH3FKMW8lKqKBadSG2Fkl0EG2L%2FjCZH%2Fys7aWaFjAlndWGNMYMaIzLsmIcflrH1m7OlUcA2he8xAvM4dkIaq8L1IDKVElCSOdXbHpHNSA3CKEZYa0NWAM6UHgkYpfiGuOaoaDDT%2Bsa14QVchjuaVYtGDBcUimrnRQKpq9xbUUErijSEH1dNvqBBU0tkxDkvrLiOq6RJe4pgYLOGhbHoILQ%2FHpVuQiXaZ4rDmWmc80wzquJPsxuRwWyYgtUWbB4r2uiLPNmC95QIclFgixIQxj71vPMYqegIfxWfnXDyLki3%2FvDr1V3llRMyDkn4nFrKw1vd0lw034HnV89rTEiWDaz4wRLx6girA9NZ3KYt6eECIerOlpQkNPu25cP5Pd1ENtK5mRKmMCz7BNNbH8ElLGp7lsBsorqxf3lvtNlTTBDdpb9Tshm1QkAihzzP8IPdiF5Ym30aGU0a86y2Fs6MduWS0y%2BJCb15Fn49czGLJsCQYLPfpm0QamgUkDks8qC1arRoshGNhYye1AjMyFnOpYbc%2FuhzlIQcerB6tiku0OLnM%2BR2NZQpG3g27tllBYQyyMaS33sOPLEXgiZsw0LQkaMDmhqAh1QFVgPgPV7tsZ4DZzjp2MxvtJU1Exlaz1z3fWSiFqsqTarJnQd4JCQiNA5VcA0a0MJ5y3C7dmpeJp%2Fq5nLMHUGQwzIVyjq144EJxKP1XY9l6OiW9sI96BP7qTES53VOlCJF76Gi8oELrDO0sbx3Eo5jknVm60dPjecA7BVZ4Cp174Yd0PbZWrEZG5mbp%2FvwjgIw%2FFu%2FWBTEcK7e4ZVc9hWamp1jfZRX7MMgDqQmFVPOLSJBJrFFs2fiBI%2FX4y6yQnoJgCIcp32q5zop6GctFJfDjYPKu%2B1JFydS6NzfQGUvdjH6wuQMdN02Im%2BgJgoOSCnikq303%2BzfIrZ9G9bth9P9zTq6D6e7Oc2LoYg2eMy4vCOS9Pk9%2BP0OyvDBqkDT8eBFCKeJS5kFSqomoyD%2BIygwoCEKgtm0n5dhfkM8Qam0sbz6GlsW21tUs7EtfM1ar3Mv8fPyoCdLdmd%2B21Nyxlw5UVGtXbLuovpARtNy0n4zg0ap9zep1yImLVCzRDkte%2FVpqkpGFbkim9b17dskRI3MlHfDhW2dDsUFLvpWXFrkJfuhBS3nyHPFCdZr5otHgrULNZ6QM2tme83nWX%2BvhkRoneE4NhGM4d2nWUlgl6zvt%2Fmslx3S3AZFp%2BM%2Bm4qLIgZTGxoU%2F0dMakXCBwyb9OS%2Fq5A1mI2elhVqykvVAqkyDc3X2SbAfdZLHexnkaDLPZ5LUedbXBEVkx2t0fmIB5OZzPeDiIz%2FpT8WhUiH5ODo6s4itpKY4dBXO0JxAEL4uwmpKYgrsjmgSe1BOKmWXbCQID6APEmGQLG6NtXHn0LGNYyTUEYZr92mbDukiikwRtjGoYyy8r5FhHQhg63FBaDGtHqbaEV0GUkMdv8TgGwmi1XjIA1oKfZ1BnGOQXMapCsZHQ2H0FtpJW1GpSHQw0IEqKVhdHbfFIowSX5VEXVuvvFiCZ7IFNxdTbWCjdBhIod0Pvd1hWQwVajiD9wmrXKWQs5XkfRNoxL1FySv3fYtqRwTY6czYr8dv0VjaOL2ye0oEJS0kbaUhVkIMwABpNDXBesUCmCCiRKC9VHxKR%2FRoq%2BkfQvIL1pKBJgqQ8VUQ3iXhlAGW2as8FtGrwMWmIzpcxiE7qdRFTlSJQxT8m7ts9lzayXRDG5mH0ZFZP%2BFRMmwHkC0NB6idIs2mp6fUU6vbMi%2FEBd3iPn9B2nxxZwV3mjN8%2FCVmQcravVOqVZ1M3nsVTKq2S4fUj%2BUNVRQvPX12t7axjv7W%2FfL95%2Fl%2F%2B2PzZJAZmp0JyqW2CLcnG7uYhRCmNKDIi9Wpyqv86C%2FLZ%2B7QJMvxF5Gia0ibkBXt5gz34XOPa%2F0%2BWScFAUSuH9qiUcMLJpPl9DJCqFoDAc0BXJENVZ1FvIzikkDa9%2B3Fx8mtNZ48vVfBTVDucGg8nWug9L6kOtEDLDG0uacch6LcR9mGelWDx4dkQsXiVTFMM06vCRCb4oh23UVDduPUoDyMy%2BUygzLNc8szSzs5RMPowp1VYFkzxfdf5JoL4M0cE7XhzZISQ274m%2B8O6dQ7VFRiDuDIgBE1i6X5voJaju5qd69%2Bkp0ryN8X6t%2Fbj5evd3BghDA%2FEAMXUHMTuPuSunQoJnz426q8DcbsPsYCd4zdVEBGwFzuYVThgDAuiom4pT7N5GCMyW9zaKBItXd%2Be%2B91C%2FJ62Q4KewfY0axL%2BcLT226HFsaXjFzENjYqCm%2BM2BtUAMa6ILmH1sJuINKyF8A%2FR8%2FCangU8D4vdcSb3VH30b0x7%2FDw%3D%3D%3C%2Fdiagram%3E%3C%2Fmxfile%3E)

[Clique aqui para ser redirecionado para a wiki de infra com os detalhamentos e comparação entre as técnologias](https://github.com/fiap-8soat-tc-one/hackathon-fiap-iac)