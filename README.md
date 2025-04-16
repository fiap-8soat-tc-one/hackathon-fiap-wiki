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
[Clique aqui para ser redirecionado ao desenho da arquitetura do projeto](https://viewer.diagrams.net/?tags=%7B%7D&lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=hackaton-arquitetura.drawio&dark=auto#R%3Cmxfile%20scale%3D%221%22%20border%3D%220%22%3E%3Cdiagram%20name%3D%22Draft%22%20id%3D%22jiXIkiBsy346j2COqR-b%22%3E7V3rd5u40%2F5rck73PcccJHH96Nhxm3fbbbbZbttPe7BRHBoMLpdc%2Btf%2FJEA2CIGJg4G0pG1qi4ux5plHM6PR6AzNNo9vA2t7%2B8G3sXsGZfvxDM3PIFRl1ST%2F0ZantAWqipG2rAPHTtvkfcO18xOnjYC1xo6Nw6wtbYp8342cbbFx5XseXkWFNisI%2FIfiaTe%2BaxcattYaFx6DNlyvLBeXTvvi2NFt2mpAfd%2F%2BDjvrW%2FbJQMu%2B8cZiJ2c3Dm8t23%2FINaGLMzQLfD9KX20eZ9ilvVfsl0XF0d2DBdiLmlyApx%2Fe%2FX19bi4f4%2FfvvJn505O%2FT5Tse9xbbpx94%2BxpoyfWBYEfezamdwFn6NwKVpmUiIjQuW2Ft8kx%2BubGcd2Z7%2FpBciFaqPQPaQ%2BjwL%2Fb9Z9GWtauFYbsKt%2BLclelP7urcke05IdeHVi2g%2FdXeb6HsxtdZ88N2Pv0WSG9rNxjWSfe4yDCj7mmrAffYn%2BDo%2BCJnJId1TNhZnBWs%2B572EMDKjpKG29zuEC6nnWslQFyvbv3XmbkRSa2Z4hQfY4EaX8%2F3DoRvt5aK3r0gegtabuNNi7rtaIQb1T65ygx7bABhBjoQFwTBZgFgU2QrElIKUvNMMtCAwY6kczgYZkRutjSl84mYai8hOj3dwhFTV1n7ZG2yN%2FmWt9bS%2Bxe%2BaETOT49uvSjyN%2BQE1x64Nxa3a0TPORlnPyQU5IPm4bblEkpWCz25sZ5pLI8z55nfhtFlIKntCPgYmV7iuQQEr5xCNICaUU%2BES5sK7LIf7Q9pP3kxJOlSx5gEvorx3JpNyw0olGLtRPdxssJgIa09db9oYUhY4cWqIvQomhlsLC21rGCSlh560Tv4iVpm66ojMMSdsj3jYqQsTKorEjv4ECAoY1j2%2FRyITsU%2BYPveV4yZdm1IRukcpoMTbEmy4qAfuUTCUd5viIfq6c2vrFiN6rUf7GmFpRZrLv35AQ%2FCDG%2By9T2YTshGhtRYcFFvHV9y6bqC2VIsLiQiRwWKQQnGQAn7%2F21P%2Fk3uY8U3q8n%2FWqxrHNI0QwRUgDSBZzPGluHilaCyuyS3GhGXslXzha7DjFifg9NBrx8kFCTAexQkxsYwlvf8aLkc9Vz8leWELG9ZnLyTyUnzJJG06ANEv2SufbknaQD7mSNfOyMa0zPNFVBq7ZrJH8bUEl4h6PVbSbtgxZEiUFKWEt6AAcX9zjtiAQkGb9tHtfUB5RWTrjygSnFYXIJZ07KsirrZbMxs%2BB7IQyTtxGBJiALRRWQBWtsHY3M%2Fc3B8RNeO6R%2FiEElX1IeJ0wtez75NSNdYhHyCMjr7KSnPoikljhOQfSqDIuC0zRUFpwpsAeAdipvjHFWgUe0ZNimXUBer%2Bnrz2F8NkNnUxA4fkgHAPrufLp0ia0csgvIA%2BSvYc22c8835U978wUT23DxwV86RB7ESNxu%2FzhwR2qq52%2F662PHYEjZ6bxZHn9UgX3AqKF94Kilfsf2GrOe8YPolhhZnuVe7Fu5ntyf896nhJ5053ccRU9Z71lx5BcFSXowePqaXZ%2B8%2BUbfSCp7O3%2FMH5w%2F5d9d4cAhX54CIGkUSM71H6YeMTuzgSmVrxVEUxqaowCicSBnxZoXjrsLQxxL%2FKEfB6usB7%2F%2BFajWpz9nf4Gn71%2BW779efENL5lCRz1vj7FLz43%2FgDmwff%2Fz5NIku%2F7rCl58mkwwRVAq1kAqwS77ffTFgKMJHdukVHUT3UFRkVUK6yX6MIjAVhbth%2BtjZPfaYIx1qPeVOy2yVmk8tGmF6xoiLivMNs%2FZ88iJ9gr0C7HroBWESo5JMw63lMcL6hH%2FETugkhKonRKpiSqrv%2Fvnn6s11nvzyVw2C5vgIZs5Y2vt7nTCiyntNmtwzIypNnOu8gcvb5wLjHIosdlGbXm4E5dPKxnv6CXyjqE0vN4IKZwCKfAFRm66Wn5i%2FGgiuBtzViXfhxxH1SWe7mRVBsJ78WVCR8mb%2BxVwHslyCeT7WWJg5OOjJVHgsvILm49gHnNuSt8M7MtZDqEgBTseTyxV9nnPyNn1VPAvfhadyl4kzYkhGMS4JZYG7bICybho1o9HLdLM86zD9ck0aLv6kv2duHFKB9BvOOORNtuA6cqEMIhhTUgU%2BCJAlgf%2BonywsKRo4OVkU9C8nEYEaIGnl%2BrFdVvOFaqioZkau8Bn9RAM5Q18TzNiZgjgTYBNC7cvG7MPQb2abV7kDwDSOcQgOyRc%2FOtHXvb9B3n1jD0Je7z%2BJvmEf1Mjur5sozdv9tXPipzb8J1ox7gT50HNLpv4E6sXhw4Dq6W13tRzBEtnuc7x1ffptLr2bwMIEpnEU0yDXaLM3iYCxWUdmtLMsgy6MdqFzLQp%2F9RnW2EcyvhV461WHNWojj3l%2BE4oIdENvfIgNcMkU6RcqsZsgUlGM8iqgGU0eQWTi7lKGMV4XsmlegMD9oKs3H3VbjRN2GtoTnqedRAWeHYNTiiEXNdOYtsblOu2vI2ih439qd13odlhb57%2B1FeEH2gu88zE3p7oCqp2PXvwNJBtF1pNFc4kih0NpIfGgTikOGGLTq0ty0tusswdpfgV%2BxPiuLtu0WaIi76GKSbcu07Rl4EAgGUXs7MI3eexoAmuujZyVOp58HWThWpulbf13E3ur9BOE0QplYIRh8inFsiB%2BBEVz2Lwd1JrUm826zPy150Q0AWEakwcdGaNrxjAVKLE8BhbdQlrPjAGajTZvcZLI8g%2FpPNIm%2F%2F%2BXf0YAdQ4gFRbHG8Us2yoICNBjthC3%2FqjB%2BMGau%2BvPkyX%2Ban6eXd7%2BnLDxpflylAOLGXLyM%2BUGfZ9flpKTdYcLTDiNhjQFXS0PCYqKJF2wZgGa6s6MaF881cotSivK67G13U5uV9FkG6wmq3BTqe21eUrpqTkwaD9iunzqfLnLlp6sUpFNqRsYWF7IpEFuIueOubTLJrYV3L0J1ss3dPqUtLL%2F%2Fkj%2Fp0cg7Xz6Jv%2Fijz%2FSp8o%2Bnj3n5HbbOHJZnn6syNUs2FIJ3MMi3I%2B0rzgug4auXsASZ%2BVnbEv5r7KspmmpiJvbziae%2BcxYYaswh9asuLGeXE2ON5hghwZ3bjKrzZuOd%2FESBx4mvZqsX6E9sG9KujSTwzbIZn6pkHsyFoGs6aTX9z8cWWhQYDwiwWoVgEiftbBiRUwTosSNMiWIqOPq43yX%2FrgM%2BDPLLdUJjaPyvU7l26nZ1rd788kgefKclnGT%2FIo2CC1rsPBzxPmI8%2BNxjhRB%2FKl7nDcIQo04H3H%2BApwL%2FN4ecC5agVa54uOZrpDjESBbrtBr4VyziY23h92zuvUiozYOVhsPuDZ2mo0jUtNSlK1%2FvQV6OUW9B71tFi4XeTshXgU4Gh2e31old8qXoqGvMdLQTclQKnVNNdWdCuXUTYUSm%2FDOK5x6qgAxEmUV%2FlYBYqCq3GIApGiSYNKwlwgxEqWXVJOhKELs%2BVFzM2QkvddJegMOsRqU02pCrMoQjA4kmigbQ6yj8r0WlxzoAEqGuf8pJrJDxUCihVTdK5poynOE%2Bgj19qCOVOGawe6h3mC99Qj1EeovYXVyeBBQF82ddRlrfZaTM8ZaX6VCvq5Y6yHVBYYxDNUVTQceFW4dHZtfTeMGEkoFmg7qYqkaUCS5%2F1gqW4XU98JGYSGCbJEie51bbVi5RvFoyZ56JaxJqBNxGNBlIEGuxETTFbH0fqYCgK4puqwaXMUwcphDTHsLZMU4Qr8WjlpdPD5YUALdkMuohMejMrlhjQHRNSy16mT1%2FPj%2BcUuXovCl42xaOROHq8BJKti72EmrUqQVV9NSR%2FMnz9rQt%2FPGtsO4eKW9hRKQkGD%2Bp4g2QxCmh6ohKYLlLG1s8yCEoPkKmPE5tXdarhbaXcGM2sVGBwuBshjwye0EU2AngBcwMlIUCeQSaNTCvREydGm%2FCKzlqkM65MwSRVdrn5a%2FACkmOvD1VLn2ihcXUBAr9UDqgTRXarmZUoNfQalZgshglNrgSwQqBlc0vrE6K7ohccb%2BaXUYyprOPXyG%2FqpnLF1xWItVUH%2FFibT4%2BDDSVUzLtFNjENN9EHxmLnp%2B5NzQAztjcop2B8N4hcOQvvVjeiVREXqI%2FtoGPj1mbdKbvSg4dVoDs96gfOZOYX0WXdA1hVcjWDIX6RS0oKYqkGs44GXmoinAJCdgFgW7cfFjxrOEie0y5RZy3ApeQoXUqvdxa1FEB9k2Hw%2BTy53P2l5qaQFT0nlrSOPcgMa8LCvG4bud2NvdofIIQvuEV3iJS3w2ktrrIjWgclukIaSXZ2w6ZzUgN0ihGWmtDVoDOlDKTMRPxTXmNUNBh%2B92al4TbqjDoafZDrWAQ0mlrp5kc1r6GFdWROiKMg2x12WjG1pQYdG%2BBqYuKBJksMI9BVLQecfiGFYQul%2BvqgzhKit1VWKZ6VwzjPNqoQ%2Biohhg8aO8vAGU2AJybvtDST2RyEWKLKhDmMTWd8XFhmAh%2FFZxdUFVsU7rEAqhM5CQW1sleE9XADc772D0q6NJRgSLfnZSYekYW4QPorGK9u1bIcLuqk4XFBS1%2B%2FzpPfl9FSTOkjmZEhB4lj3QkvZHcBlf5L4VJqvY9Li74ne6rAny205W%2Fk4Is%2BoKAHnE%2FEvwYOeqJ9amj1ams%2B4gi72Vk8C2uCXlS5JSBw%2Fh1zMc8zDd7U%2FdG0gbbC1UyIh81jZitXa0KLWRT4ZkN2rkJ%2BxAx6Nx50DUuQoipB7cUZuc7tC90edI7Gwo0jbw7XgVZRuO7VIaC%2BfYSeqJvRSCsw0jEBWnmUxRpQVV5D%2FwYc%2FWYNcgNj4ch%2FHXKEWts8hdJ6WoxXZqs12ErgIcEhEROWfml2ZtqOC8Zbg9G0qg%2Bbcat0xlt1yA7Tql9u1FgoGkY7W9k8tRya1tZHvQO59kc5c6n7WnzV3Ku7eoXN4CHyttnM6tFGlX1ZtNHT43mwPw%2B80AU699sNIFbW9QIxZzM2f7PA6THAzHu%2FGDTUUGb%2FwMn%2Bawp9TU5xq9o65GGQB1IHGTnrBvBwk0Sy2aPxETfr4cbZPubRN%2BRgwogpWN3Zom%2BjBMk1b2lYPNM8o73QquLpxxcmsBFCPYx1sLsMh3Otvys2VrAXEpckDOzJTTDv7Niimywd%2B2bD8Z7GnK0X0y1M9tnM8%2Fssc5xP6DlqZZDh91OybDBnUDhxM%2BClEZEheyChVULcZeIkZQ4UhClQVOfreBwt0I8QsMpY3H0WGsWW1tUGbqevL5aW5n9ONHZcCPlvyy%2FbaGZUZc7IMUTTv9sMw%2BtFHuzjUah9zOh1yIuHlCzRAUte%2FUp6nZLSyPis9b17dskRE3gqjrcAq%2FZTsU7HTTseHWoCjdgAy3H2EZFIPcp9rk96lWWaJ1j5Zbs8hvNsr8fT0yROcMUYKNZvY9q8f2B3rN9n6bk3Knm4BjXDwY891UeBIzuLzQpvY74uouEDrknqYl%2B12BvMdsdDCnVrO3UCGNYrey%2BYKtBNyXsIwTO42mWOyLWo42W%2B%2BMrJgqByjBco%2BObTbj12FkLp6yO1bFyMcU4DhVFkXtNmOHSVztiMQBT%2BLmkQuxFdk8cKeWSNw0i0EYCFAXJN6kPMCYefvKM28BBy3TFCRhduuXCTddEiU0eGNGQ19uWbHYIgKa0bNbJtwJamSrX4utgC4jiVviNwTCajZdMRJWj5FmU%2BeAMwTOalCpZAw2HyFtxK0oR7t0qB5JQjSzMEabB8USpQqfqmir7m45osn6x0xdnY21xk0YoWL1836ldQVl8FtRJB84Za0yayGvb6NoGyb70yzI3ztsW1J4S145mzX57fprmkeXtE%2FobgrpfjbSlpogPXEGMLgC4rpghkoRbD%2BitLD1iFj0z6jPN4r%2BBaI3DUUCvPShItqAuFMAKKNPc9a7T4NXQTswIxZAcXSZGGZ57QhgjmzBKlHAycoksW1ZGtklF7NPo13SuV0yQWoxZj3RFUGNrW5LJDXLtppeXZKT3loRfqAh7xE6Xefp8bu3q2WnF7IISh442qlm65RmWTcfx31SXiXg9in5fW2N8i7%2BGC%2Ffgmh97a6gfP5T3epfmtR%2FHLwpvXECX9pg24k30oo8AVzc016htSioCKc3Drl%2BCpKiZwv5%2F%2BT54yf736ev%2FyJ4%2FuXRW2cmdoVABWKvHo8AZCFPNiAhGUmaYCWsbEosVb1g0NDNoE4l67JoB5MOcMibEa3EOFRiIf223LT5wcl6PP3w7u%2Frc3P5GL9%2F583Mn578ndWdy8%2FVV2tT6zPzRtE8NlCz0r%2FVs%2BmVM%2FcTCIvu3r7u74EkgCMm14U9OJDSIC%2FC6DGbvjwzv%2BW02K7xh1oH98QExWT3CZSbFZU8Bt98agr5sGZJLm3huzrYnStVkVUWFJX6%2FxEnx9KNAWjrKrECke2s81l%2Fw6l7URG5KhlXLxt4Nd4RRIKlrsAUjbgaR3ANhltqXPg09XKPDBoX%2BeDbdHvpi%2F8B%3C%2Fdiagram%3E%3C%2Fmxfile%3E)

[Clique aqui para ser redirecionado para a wiki de infra com os detalhamentos e comparação entre as técnologias](https://github.com/fiap-8soat-tc-one/hackathon-fiap-iac)

