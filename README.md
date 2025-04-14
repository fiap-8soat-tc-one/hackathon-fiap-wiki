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


## Arquitetura
[Clique aqui para ser redirecionado ao desenho da arquitetura do projeto](https://viewer.diagrams.net/?tags=%7B%7D&lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=hackaton-arquitetura.drawio&dark=auto#R%3Cmxfile%20scale%3D%221%22%20border%3D%220%22%3E%3Cdiagram%20name%3D%22Draft%22%20id%3D%22jiXIkiBsy346j2COqR-b%22%3E7V1rd5s4E%2F41Oaf9YA6SuH507KTNu%2B0227Tb9tMebBSbDTYUcC7761%2BJm0ESmDhcnBanTcwgMNbMPJoZjUZnaLZ5fBdY%2FvqjZ2P3DMr24xman0EIVFWWyV9KekpIGkApZRU4dkIrEG6c%2F3B6bUbdOTYOU1pCijzPjRy%2FTFx62y1eRiWaFQTeQ7nZrefaJYJvrXDpMSjhZmm5mGv2zbGjdUI1oL6nv8fOap19MtDM5MzGyhqnNw7Xlu09FEjo4gzNAs%2BLknebxxl2afeV%2B%2BWy4mz%2BYAHeRk0uwNOP7%2F%2B6OTcXj7sP77cz87%2Bt%2FO9ESb%2FHveXu0m%2BcPm30lHVB4O22NqZ3AWfo3AqWKZcIi9C5bYXr%2BBw9uHVcd%2Ba5XhBfiC5V%2BkPoYRR4d3n%2FaYSycq0wzK7ytlHhquSVX1U4o8UvenVg2Q7eX7X1tji90U363CA7Tp4V0sv4Hks78R4HEX4skNIefIe9DY6CJ9IkPaunzEzF2VCSw4e9aEBFRwlxXZALpCtqKpSpQK7ye%2B95Rt6kbHsGC02OYdgmIpweekG09lbe1nIv9tTzPUspA%2FZtPnien3bevziKntLes3aRR0jraOOmZ3mOHt35obcLlrjmC6YdR79VLYsC7FqRc19WXFF3p5deew55lJy1AIESbyeEIjFMi6xghaP0QoZv%2BZMcz0r1OcpIOfewdiJ841tx%2Fz0QDC5zidHHW5X%2BHKVxuZq3zPzmmjdRgFnmD5JVXvsMk1c%2BYKCOdA8eZhiBfZ%2B%2BdTbxSFNkD%2F3yDhlqpq6z2hJaRJUvp36wFti99kIncjx6duFFkbchDVx64txa3q1iYSgyOH6RJvGHTUM%2FGRGppFjZwa3zSBl5nj7PfB1FdCid0o6Al0t7q0gOGUxvHSJmgbQknwgvbSuyyB9KD2k%2FObvJwiUPMAm9pWO5tBsuNYKMlysnWu8WEwANyd%2BuhhMVpDCiQm7Nioqi8ZKS0VoXFMQJyjsner9bENp0SRkccoJDvmxUlhcrlZMl6RocCARo49h2jO8iXCgjB9vtLFt4xrXBGKQyOgxNgQ7LimAAlTvijPJ8FT5WQ218a%2B3cqFLzxTpaUmOx1t6TBl4QYnyXKuyDPyG6GlFOwcud73qWTRUXypAI4qVMmHCZyN8klb7JB2IATP6O7yOF96vJsPor64yYaDonJgDpAqjPiK3LicbJyeyK3GhG3snXjo9dh9igv6cOawIdBrBHHW7gxPjUZos%2FVz0n%2F2QJkYeeyfF%2FlTSYxUTToARJBSV6fCTpgGlMZXLGEJOWpiqgajmR%2FGsAIuEdjpbrlNUHrQYOOzhBi3sABxf3OOmIWEJSZNs8rqgDLy2dcOkBU9qF8SWM%2FSjLqqzzdmLqfQ0CFSZrFAKNd8lA5nuVkKIzhywLXRTE8TNeOaR%2FiBElX1EEJxgtbz3ya0a6xCLIEZD3aaOnIVCkFjW6gHiTNeYFrjQwBYYA0GBXfBNZ81o8XtMeIO9X9P3XcHc2Q2dTEDheSMGfHp1PFy4xj8PsAvIAxWsysu3cs6RiszffMLEILz96C4ewg5iGvv%2F2wB2pdV686a8vOoaGGJU3%2BeFHFdgGGTK0Lzgq1%2B89hGBIDwZP39Pr44Mf9EBSs8P5Y%2FHk%2FKl4dI0Dh3x5KgAxUcA513uYbom9mY5LCX%2BtIJrSqCoVIBrCc5YZ%2BdJx87BDG6Gg738GqvX5j9mf4Onfb4sP3y9%2BoEXmRiVhmHQE%2BPQPuAP%2B488%2FnibR1Z%2FX%2BOrzJIuJdR0yUmRVQrqZvYyyYCoKc8OK6BHpUOup0Cw1VWo%2BtRyo0lNEvKxob5i17cmb5AlaDWVBoxJMQ9%2FaZoD1Gf%2FcOaETA6oeA6mKKai%2B%2F%2FLl%2Bs1NEfyKV50EzLHB54KttHf0ekFElXWXNHlgRFSaeNVF%2B5Y1zwW2ORQZ7CKazhMB34y33ZNPYIkims4TQYUvAEWugIimq%2FwTs1cDwdWAuTp2LrxdRP3RWT4pJphnIT%2BXlKWslX8x14Esc2JeDC%2BWJn0OOjIVDguroMW49QHHlnN2WD%2FGeggVKcDJeHK1pM9zTg6Td%2BVW%2BC7sylUmvoghGeVQJJQRb%2BgagNdNo2Y0eplu8rMM0283hHDxB%2F09c3chZciwoYxDzmQLnqMKWMaYEh%2FIACaQJYH7qHcWjxQNnAwvSvpXnJXj1QBJS9fb2byaX6qGimomU0ufMYhvr5WtKYGVbwqCTCCbAWqfM6cy0yqwzKucAWAax7gDnU%2FmFi342sSErk34iVYOREA2gNyS0T6BqCzPRirfnVrhKh%2BKElnhc%2By7Hv02V9vbwMJE5HbRjkarRuu7QSgLaUo2QmQGuGr0Z4ALHWVRKGvIEMU%2BKvGjhEKvOkRRG0UsIpyQRaAfgGPDZYDJhUi%2BEIdvgqgDLI%2FHoBlQHgFl4u5STmP0LWXCvEAC8aMTJaqhq%2Bnhj8KpvWbQg5JitBXz6zVMJ2yndaICz46nKeXwiZpqTFsjc5321wG00Inv2vUWuhCW7%2FyzsiL8QHuBdSTm5lRXQLUjMYjvgFQoGWW2AgVygzKQjTjGwjkRoAX3rk41Dhhk0%2Bsr0uhd2uUnaYYFXpShXl3qb7NUQ9bnFENvXdpvy%2BIDgWQYZccgC8gUxUcT2HRtZKDUoeXrgAzX2ixs65%2Fb3XaZfIIw%2FqCcGGyY0GRMeVUWzEtD0bw0aw%2B1xvdmMykzb7V1IppTMN2RBx0xo2%2FMMOUyYGhIkLPWK2CAZoPNOxwnpnwhPUdo8v%2B%2BfRmlp3fpUWFZehSTxx0EBNJjtmCpfNLg7sGau6uvkwX%2Bbn6dXa3%2Fm2TDS%2FOlQQdWIxT4Z8oN%2Br64RKjA6x4X%2B2SpRnlsEOqSys8RKCqSdMG6A2iquRXRPnuqlVuUJ1TUY8v3J%2BtlNPGD5WQZbiq1vTbxKGlaEAbt544uZTtf5HnPk2XCsin1BQNrG2bcIDeRC%2Bdc2mUT2wru3gSrxRs6H0qo2Z%2B3yV96BtLOpwfFN2%2FfJk%2BVfnz2nJO13ziAyc8nVuRelkypWNzDsrgfaV4xWAYNXb2AHGYVp2C5fFZZVpM0U8RMVqczyWymq5AqzIk1K26sx1dLptlgxhwaTNt4mpq1HO92CxxsMenVeA0K7YE9Ke7SlA9%2BkE7lUiYPZCsCWdNJr%2B9fDFhoUDCbiASLTgAifdbCwhMxTIgyMXhIEEHH9ad5ns%2B4CNiWPKU6Q3FUvtepfLma%2BZ49nEtGnrygZcysvSJYfDCAljVYuTnK%2BSjnx8s5UgThp%2F7lvEEMapTzUc5fIOcCv3cAORetKKtcwvFMV8jZEkG2XKHXwrhmExv7h92zugUgozaerDYecG3sJClHpKZclG14vQU6n3M%2BgN42i5WLvJ0QLwMcjQ7Pb62SufIl0jDUGGnopmQolbqmmmquQgV1U6GUTUEUFU7tKkCMRMmFv1WAGKgqk92PFE0SlKUZJEKc1TNqCIaiCPHWi5qbISPovU7QO%2BEQq0ExrSbEqpyC0YFEE2VjiHVUvtfikgMdQMkw96%2FySgWoGEi0Mqp%2FRRNNeY6iPop6e6KOVOEiwP5FvcEC6lHUR1F%2FCaqT0ych6qK5sz5jrc9ycsZY66tUyNcVaz2kusAwTkN1RdOBR4VbR8fmV9O4EwmlAk0HdbFUDSiSPHwsNVubNPTqRmFtgXSlYva%2BsOSwcqHi0ZztejmsSaCTKfQOdRlIkKka0XRZLL2fqQCga4ouqwZTAoycZiSmvVWyYjlCv5YctbqC%2FGSFEuiGzEslPF4q4xvWGBB9i6VWnaxeHN8%2F%2BXQpClsLzqalMHG4DJy4Cr2LnaQ4RVJBNaldNH%2FaWht6OG9sO4yLV9pbKAEJCBZfZWkzBGF6qBqSIljO0sZWDUIRNF8BMu7RsOdSAH1WzahdbHSwsmcWA%2B7cTjAFdgJ4ASIjRZFAIYGmvMwUIUOX9ovAWi4%2BpEPGLFF0tfZp2QuQYqIDXy8zjSuueHEVBbFSn0hRkOZKLTdTavArKHWWIHIySm2wNf8UgykC31idFd2QGGO%2FWx2GsqYzD59Kf9Uzclcc1mIV1F%2FRkRYfH0a63tG669QYxHRfAy8zF7de5NzSE7kxOUX5yXC3xGFID70dvZKoCD1Ff%2FmBR89Zm%2BRmLwpOdWtg1huUz9zqa8iaC7qmsGoEOXORTkELiqQCuQYDXmYumgKZZBicRcFuXfyY4ixBYpuH3FKOW8lLqOBa9UZsLbLoINoW42Ey3%2FkZ7aWWFjAlnbWGNMYNaIzLsmIcvlvH3m4ulUcA2me8xAvM4dkIaq8L1IDK7JKEkM7P2PSOakBukEIzwlobsAZ0oPBIxE7FNcY1Q0GH79Y1rgl3yGGkp9luwYCRkkpd7WSjYPoY11ZE4IoiDbHXZaMfWFAhU5PO1Pm5J2BkhXtKoKCzjsUxqCB0v15VLcJlWueKQ5npXDOM82qmD7PaUYGs2gJVFkze65qoyrzZQjRUyHKRIgvKEMax9byy2ClYCL9VXN0wcqnI1%2F4I8KKromJCyTmRiFtbZXi7K4KbtjsY%2FOppjhHBspsdF1g6xhRhY2g6U8O8PSNE2F3V2YKCmnZfP38gv6%2BD2FcyJ1MiBFvLPtHC9kdAGVvqvhUgq9i9uL%2Fad7qsCUbIzqrfCcWsugBAUWL%2BJvJgF4on1maPVmaz5iKLt0snFtvyFpMvyUk9eRF%2BPaMxK6ZAUOCzXyFtsFVQKSHyWduC1ZrRosxGNhcyu1EjNyEXOlYac%2F%2BhzlMQSerB3bFJc4ducj5HYl9DkfzAs3fLKN1ALM9oLLWx48wTeyEUzjY8CBkxNqCpCWxAVeA9ADbu2ZrgNQiOn47H%2BEpLUTM7W%2BtZ6K6XQtRiS7XZbkLXAQ4JiwifUwNMszaUcdtF6J%2BdSqT5txq5DFNnMMSA%2FA5F%2FfqR4ETysdrez%2BWo7NY20j3onTvZ4qXOax1oixd%2BDxeVSVxgg6WN87mVch6TqjebO3xuOgdgd50Bpl77YNwFbW9TI2ZzM3f7fBfGSRjO9tYLNhUpvLtneDWHfaWmXtfoH%2FU1ygCoA4mZ9YRDu0igWW7R%2FIkY8fPFaJucgG0CgKjWab%2FGiX4axkkr%2B8vB5knlvW4JVxfS6NxeAOUo9vH2AmTcNB12Yi8gJksOyKmh0u3w36yeYjb825btxcM9zTq6jwf7uY2LKUj2OI04fODSNPn1OP2OyrBB6cDTCSCFiBeJC1mFCqpm4yAxI6gwIKHKgpG031BhPkL8AkNp43H0NJattjYoZ%2Bra%2BRy1Xpbf40dlwI6W7Mr9toblDLjyTUa1drd1F%2FMDNhqWk%2FSdGzQOub0PuRAxc4WaIahr36tPU7NhWFEqvvquZ9kiI24Uor4DKuzW7VCw2U3PhluDunQnZLj9DHmhOMn9qtnNQ4Ga5VoPaLk1i%2F2mo8xfNyNC9I4QnNho5tChs2yLoNds77c5LdfdFFyGxSdjvpsKC2IGkxva1H5HTOkFAofM07RkvyuQ9ZiNHmbVarYXKiVS5IubL7LFgPsqlrvYTqNJFvu6lqPNNjgiKya72iMLEA9nsxm%2FDiIz8ZT8XBUiH1ODo6s8itqdxg6DuNoTiAMWxNlFSE1BXJHNA3dqCcRNsxyEgQD1AeJNKgSM2bevPPsWMKJlmoI0zH79MuG%2BS6KUhu2Y0zCUW1aut4iANnS6pXAzqBGtfi20ArqMJGaZ3ykAVrPpihGwBow0mzojOKeAWQ2KlYzB5iO4jbSyVYPydKgBQUI0szBGm08KJbgin6pot%2B5%2BMaLJGshUXZ2NtcJNEKFiBfR%2BtXUFZLC7UcQfOM2ockYh79dR5IfxFjWX5N8dti0pXJN3zmZFfrveiubRxfQJ3VAh2dJG8qkJMhBmAIOpIa4LZqgUwQ4kSgu7j4hZ%2F4wSfSPrX8B601AkwHIfKqI9iHsVAGX0ac4G92nwMmhJzJSyiE3ochLRLkeiinlK3rR9KWvmvSSGycXs82iY9G%2BYMAnOE4CGtkuUZtlW0%2Bsr0uidFeEHGvIeJafvPD12A3eVd3rzKmxFwdG6mq1TmmXdfBq3SnmVArdPye9pdxRyGHg0PyA%2F944O3h89m26DePF%2F%3C%2Fdiagram%3E%3C%2Fmxfile%3E)

[Clique aqui para ser redirecionado para a wiki de infra com os detalhamentos e comparação entre as técnologias](https://github.com/fiap-8soat-tc-one/hackathon-fiap-iac)

