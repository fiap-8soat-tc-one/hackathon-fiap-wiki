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
[Clique aqui para ser redirecionado ao desenho da arquitetura do projeto](https://viewer.diagrams.net/?tags=%7B%7D&lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=hackaton-arc-v2.drawio&dark=auto#R%3Cmxfile%20scale%3D%221%22%20border%3D%220%22%3E%3Cdiagram%20name%3D%22Draft%22%20id%3D%22jiXIkiBsy346j2COqR-b%22%3E7V1bd5u6Ev41Wat9MAuJ%2B6Njx23Obnezk3a3ferCoNjsYEMB59JffyRuBklg4nBxWtI2tcXFWPPNp5nRaHQmzTaP7wLTX3%2F0bOSeQdF%2BPJPmZxAaui7h%2F0jLU9IiqZqctKwCx07axH3DjfMLJY0ga905NgrTtqQp8jw3cvxyo%2BVtt8iKSm1mEHgP5dNuPdcuNfjmCpUegzTcWKaLmNO%2BOna0Tlp1qO3b3yNntc4%2BGahGcmRjZienNw7Xpu09FJqkizNpFnhelLzaPM6QS3qv3C%2BLiqP5gwVoGzW5AE0%2Fvv%2Fn5txYPu4%2BvN%2FOjF9b8b%2BJnH6Pe9Pdpd84fdroKeuCwNttbUTuAs6kczOwUilhEUnnthmu42Pkza3jujPP9YL4QmmhkD%2B4PYwC7y7vPxW3rFwzDLOrvG1UuCr5ya8qHFHjH3J1YNoO2l%2B19bYovdFN%2Btwge588KySXsT2WduI9CiL0WGhKe%2FAd8jYoCp7wKelRLRVmCmc9BfPDHhpQ1lLMrwu4kDRZSUGZAnKV33svM%2FwiFdszRGgwAkM2hnD61guitbfytqZ7sW8934uUCGB%2FzgfP89PO%2Bw9F0VPae%2BYu8nDTOtq46VFWokd3fujtAgvVfMG048i3qhVRgFwzcu7Lisvr7vTSK8%2FBj5KLFkigJNsJbhEooUVmsEJReiElt%2FxJjhel8hxlJJJ7WDsRuvHNuP8eMAWXpUTp461C%2Fhylcbmatyz85po3kYFRlo8kKqz26QarfCAbhVrXPXhYYJj2ffLS2cQjTVE85Ms7eKiZus5qi9sionx56wdzidwrL3QixyNHl14UeRt8gksOnJvW3SoGQ1HA8Q8%2BJf6waegnIyJBipm9uXUeiSDP0%2BeZr6OIDKVT0hFwYdlbWXDwYHrrYJgFgoU%2FES5sMzLxf6Q9JP3k7CZLFz%2FAJPQsx3RJNyxUzIyLlROtd8sJgLrgb1fDQUWSKajgW9NQkVUWKVlb60CRGKC8c6L3uyVum1pEwCEDHPxlozJezBQnFu4aFHAAtHFsO%2BZ3Hi%2BUmYPudlosrODaEIykUDoMDY4OizJnABU7koz8fBU%2BVkNtdGvu3KhS8%2Fk6WlJjvtbe4xO8IEToLlXYB3%2BCdTUikoKLne96pk0UF4oQA3EhYiEsEvxNUvRNPmADYPJvfB8hvF9NhtVfUaNgomoMTICkcag%2Ba2wdJyqDk9klvtEMvxKvHB%2B5DrZB%2F0wdVjk6DGCPOtzAifGJzRZ%2FrnKO%2F4qChB96Jsb%2FFHzCLG40dNIgKKDUHr8TNECdTDA5oxqTMw2F06rmjfhvAxIJ71BkrVNRH7QaGO5ggBb3AAou7lHSETFCUmbbPK6I%2Fy5YTmh5wBB2YXwJZT%2BKoiJqrJ2Yel%2BDUIVBG4VAZV0ykPleJabozCHLQhcFOF6jlYP7BxtR4iVhcMzR4tbDv2a4S0zMHAF%2BnZ70NASL1LJGFxRv0MY8x5UGBscQACrsSm48a16Nx2vSA%2Fj1irz%2BEu7OZtLZFASOFxLyJ%2B%2FOp0sXm8dhdgF%2BgOI1WbPt3NNNxdPefEXYIlx89JYOFgc2DX3%2F7YE7Euu8eNPfHzq6KlEqb7DDj8KxDTJmaB84CtPvPYRgcA8GT9%2FS6%2BM338kbQcnezh%2BLB%2BdPxXdXKHDwlycAiBs5knO9h%2BkW25vpuJTI1wyiKYmqEgCREJ5jZc0Lx83DDm2Egr79HSjm9V%2Bzv8HTf1%2BXH75dfJeWmRuVhGHSEeDTD3AH%2FMeffz1Nosu%2Fr9Dl9SSLiXUdMpJFRZA0I%2FvRy8CUZeqGFdEj3KHmU%2BG01FSp%2BdRyoEpLGXFRcb5u1J6PXyRP0GooC%2BqVZBr65jYjrGv0c%2BeETkyoWkykCiKk%2Bv7z56s3N0XyK151EjRHB58LttLe0euFERXaXVLFgRlRbuJVF%2B1b2jzn2OaQZ7Dz2jS2EbCnsbZ78gl0I69NYxtBhS8Aea4Ar01T2CemrwacqwF1dexceLuI%2BKOzfFKMM8%2BC%2FyyISGkr%2F2KuAVFkYF4ML5YmfQ46MhUOC62gxbj1AceWcXZoP8Z8CGUhQMl4cmmR5znHb5NX5bPQXdiVq4x9EV3Qy6FIKEqsoasDVjf1mtHoZbrJzjJMv97ghou%2FyO%2BZuwuJQIYNZRxyJlvwHBVAC8YQ2EAGMIAocNxHrbN4JG%2FgpGRR0r%2FirByrBpJgud7OZtV8oeiKVDOZWvqMQXx7tWxNcax8gxNkAtkMUPuSOZWZVo5lXuUMAEM%2Fxh3ofDK3aMHXJiZ0bcJP1HIgAtIB5JaM9gmUynjWU3x3aoUrbCiKZ4XPke965Ntcbm8DE2HI7aIdiVaN1neDUJakytkIkRngit6fAc51lHmhrCFDFPuoxPcSC73qEEVtFLHIcFwRgX4Ijg6XASoXIvlCDL9xog6wPB6DZkR5BJXxu0s%2BjdG3lAnzAgSiRydKVENT0rffC4f2mkHelBSjrZhfr2E67nlqJyrw7HiaXA6fKKnGtDUy12l%2FHUFznfiuXW%2BuC2H6zo%2BVGaEH0gu0IzE3ppoMqh2JQXwHSYGCXhYrkCEzKANRj2MsjBMBWnDv6lTjgEE2vbrEJ71Lu%2FwkzbDAizLWq0v9bZZqSPucfOqtS%2FttGT4QCLpedgyygEwRPirHpmsjA6WOLV8HZbjmZmmbP253Wyv5BG78QT4x2jCgQZnyisiZl4a8eWnaHmpN7s1mUmbeautEJKdgusMPOnJG35xhiGXCUCVOzlqvhAGaDTbvUJyY8hn3HG4T%2F%2Ff184ie3tGjwDJ6ZIPlHQlw0GO0YKl8UuHuwZy7qy%2BTJfpmfJldrn9NsuGl%2BdKgA6sRCvIzxAZ9X1wiVJB1j4t9slSjPDYINUFh5whkRRI0zroDaCi5FdG%2BeKqVm5cnVNRj0%2Fcnayua%2BIE1scJNpbbXJh4lpxbAoP7ckaVs58s873liJSKbEl8wMLdhJg18E7FwzCVdNrHN4O5NsFq%2BIfOhuDX7723yPzkCSeeTN8UXb98mT5V%2BfPack7XfOIDJzidW5F6WTKkY7mEZ7keaVxSXQV1TLiDDWcUpWCafVRSVJM1Uoiar05lkOtOV28rNiTUqbqzFVwuG0WDGHOrUufE0NW053u2WKNgi3KvxGhTSA%2FumuEtTOfhBOpVLhDyQrQhEVcO9vv%2BhyEKFnNlEibPoBEi4z1pYeMKnCV4mBksJPOq4%2BjTP8xmXAX0m21KdoTgq3%2BtUvlzNfM8eziXDT17QMmrWXuYsPhhAyxqs3BxxPuL8eJxLMif81D%2FOG8SgRpyPOH8Bzjl%2B7wA4560oq1zC8UxXyNliIJsu12uhXLOJjfzD7lndApBRG09WGw%2B4NnaSlMNTUybKNrzeAo3NOR9Ab5vFynneToisAEWjw%2FNHq2SufAkahhojdc0QdLlS1xRDyVWooG4KFLIpiKLCKV0FiCVecuEfFSAGikJl90uyKnDK0gwSIc7qGTUkQ16EeOtFzc2QkfReJ%2BmdcIhVJ5xWE2KVT8HokHgTZWOIdVS%2B1%2BKSAw1AQTf2P%2BWVClDWJd7KqP4VjTflOUJ9hHp7UJcU7iLA%2FqHeYAH1CPUR6i9hdXz4JKDOmzvrM9b6LCdnjLW%2BSoV8XbHWQ6oLdP00VJc3HXhUuHV0bH43jTuRUCpQNVAXS1WBLIjDx1KztUlDr27k1hZIVypmrwtLDisXKh4t2a6XwxqYOqlC71ATgQCpqhFNl8WS%2BxkyAJoqa6KiUyXA8GEKMe2tkuXjSPq9cNTqCvKTBSXQdJFFJTwelfENawyIvmGpVierF8f3Tz5ZikLXgrNJKUwUWoETV6F3kZMUp0gqqCa1i%2BZPW3ND3s4b2w7j4pX2FkpATILFnzLadE6YHiq6IHOWs7SxVQMXgsYrYMY9G%2FZcCqDPqhm1i40OVvbMYsCd2wkGx04AL2BkSZYFUEigKS8zlSRdE%2FaLwFouPqRByiyRNaX2aekLJNmQDny9zDSuuOLFVRT4Sn0iRUGaK7XYTKnB76DUWYLIySi1Ttf8k3WqCHxjdZY1XaCM%2FW51GIqqRj18iv6qZ2SuOKzFCqi%2FoiMtPj6MdLUjddeJMYjIvgZeZi5uvci5JQdyY3Iq5QfDnYXCkLz1duRKrCLkEPnlBx45Zm6Sm70oONWtgVlvUD5zq68hay5oqkyrEWTMRTIFzSmSCsQaDniZuWhwMEkJOIuC3broMeVZzMQ2S7mlHLeSl1AhteqN2FoU0UG2LcbDRLbzs7aXWlrAEDTaGlIpN6AxL4uyfvhuHXu7OSqPILRrZKElYvhsJLXXRWpAoXZJkiSNnbHpndWA2CCFZqS1NmgNaEBmmYieimvMa7osHb5b17zG3SGHQk%2Bz3YIBhZJKXe1ko2DyGFdmhOmKMA2210W9H1pQIFWTztDYuSegZ4V7SqSg0Y7FMazAdb9eVS1CK61zxbDMdK7q%2Bnm10IdZ7ShDWm2BInIm7zWVV2XeaCEayhU5T5E5ZQjj2HpeWewULIQ%2FKq6u6zkq8rU%2FHL7oqqgYFzknEnFrqwxvd0Vw0%2FMOBr96mmOUYNnNjgssHWOK0DE0japh3p4Rwu2u6mxBTk27L9cf8O%2BrIPaVjMkUg2Br2ida2P4IKqNL3bdCZBW7F%2FdX%2B04TVc4I2Vn1Oy7MqgsAFBHzL8aDXSieWJs9WpnNmkMWbS0nhm15i8mX5KSePIRfz2hMwxRwCnz2C9IGWwWVEiKftS1YrRnNy2ykcyGzGzVyE3LQ0WjM%2FYc6T4GH1IO7Y%2BPTHbLJ%2BVzi%2Bxqy4AeevbOidAOxPKOxdI4dZ57YSy442%2FAgRImyAQ2VYwMqHO8B0HHP1oDXIDh%2BOh7jKy1FTe1srWWhu14KUfMt1Wa7CV0FKMQiwnJODTDV3BDBbZehf3YqkeY%2FauTSDY3iEB2yOxT160eCE8nHans%2Fl6OyW9tI9yB37mSLlzqvdaAtXtg9XBQqcYEOljbO55bLeUyK1mzu8LnpHIDedQYYWu2DMRe0vU0NX8zN3O3zXRgnYTjbWy%2FYVKTw7p7h1Rz2lZp6XaN%2F1NcoA6AGBGrWEw7tIoFmuUXzJ2zEz5ejbXICtgkAvFqn%2FRon2mkYJ63sLwebJ5X3uiVcXUijc3sBlKPYx9sLkHLTNNiJvSBRWXJATA2Vbof%2FZvUUs%2BHfNm0vHu5J1tF9PNjPbVRMQbLHacThA5eGwa7H6XdUhg1KB55OACmUWEhciAqUpWoxDhIzgjJFEorIGUn7DRXmI8RvMJQ2HkdPY9lqa4Nypq6dz1FrZfwePyoDerSkV%2B63NSxnxJVvMqq2u607Xx6w0bCcpO%2FcSOOQ2%2FuQCyVqrlDVOXXte%2FVpajYMK6Lii%2B96ps0z4kYQ9R1Qobduh5zNbno23BrUpTshw%2B1nyILiJPerpjcPBUqWaz2g5dYs9puOMv%2FcjAzRO0MwsFGNoUNn2RZBr9neb3NarrspuIyLT8Z8N2SaxHQqN7Sp%2FS5RpRcwHVJP05L9LkPaY9Z7mFWr2V6olEiRL26%2ByBYD7qtY7mI7jSRZ7Otajjbb4IwsG%2FRqjyxAPJzNpv8%2BjEzFU%2FJjVYx8TA2OrvIoancaO0ziSk8kDmgSpxchNSVxWTQO3KklEjeMchAGAqkPEm9SIWDMvn3l2beAgpZhcNIw%2B%2FXLuPsu8VIatmNOw1BuWbneogTUodMtuZtBjWz1e7EV0ERJoJb5nQJhNZuuGAlrwEizoVHAOQXOalCsZAw2HyFtSS1bNVKeDjUgSfBmFsZo80mxBFPkU%2BHt1t0vRzRZA5mqq7MxV6gJI1SsgN6vtq6gDHo3ivgDp1mrmLXg1%2Bso8sN4i5oF%2FnuHbFMI1%2FiVs1nh3663Inl0cfuEbKiQbGkj%2BMQEGYgzgE7VENc4M1QyZwcSuYXdR%2Fiif0aJvlH0LxC9ocsCoKUPZd4exL0CQB59mrPBfRpkBS3BTC5DbEKWk%2FB2OeJVzJPzU9tHWTPvJTFMLmbXo2HSv2FCJThPgDS0XSI3y7aaXl3ik96ZEXogIe8ROX3n6dEbuCus05tXYSsCR%2B1qtk5ulnXzadwq5VUCbp%2BSP9TuKKHx6%2FOV7ev6e%2FvL14v3X8V%2F7I9NSkBmJjRj6hZgUd7cbs4DSqFPsQOxN4tT89dZ4t%2Fmr12AyDfCd0NYNjEa4OIGbe13gWP%2FmFoWRlAUCuH9qiUe0LNhPp9DxCYFZ2M4oMmCzttnUWuhOidXNKz5cXPx95yMGteX81FVOxwbdIUqvpinJfVhVnDB8JsVzWglr6KO0DrfuU2k1n9CkRJ98wrP1ApPPAhQLk1bO4no9MZG2SdV7iRy6IoXZ1hwZchGhC%2B2986hPT5GQuyMEAFV1B6jnY0BtpXcht8GHkls3EOKRB0%2BejbZv%2Fni%2Fw%3D%3D%3C%2Fdiagram%3E%3C%2Fmxfile%3E)

[Clique aqui para ser redirecionado para a wiki de infra com os detalhamentos e comparação entre as técnologias](https://github.com/fiap-8soat-tc-one/hackathon-fiap-iac)

