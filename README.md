# 🚀 CI/CD Workflow Templates (`descubra-muriae`)

Este repositório centraliza os **Workflows Reutilizáveis (Reusable Workflows)** do GitHub Actions para a organização **Descubra Muriaé**.

O objetivo é padronizar e otimizar nossas esteiras de Deploy Contínuo (CD) em todos os microsserviços, utilizando boas práticas de segurança, cache de alta performance e integração automatizada com o **Coolify**.

---

## 🛠️ O que esta pipeline unificada faz?

O template central (`cd-template.yml`) executa os seguintes passos automaticamente:

1. **Checkout** do código fonte do projeto.
2. **Login no GHCR** (GitHub Container Registry) usando o token nativo de execução.
3. **Setup do Docker Buildx** (motor BuildKit para alta performance).
4. **Build e Push da Imagem Docker** com tag `:latest` e `:${{ github.sha }}`.
* Utiliza cache nativo do GitHub Actions (`type=gha`) para builds ultrarrápidos.


5. **Disparo de Webhook para o Coolify**, enviando o token de autorização e o bypass do Cloudflare WAF (quando configurado).

---

## 📋 Pré-requisitos (Configuração por Repositório)

Devido às limitações do plano *GitHub Free for Organizations*, as **Secrets** de deploys em repositórios privados precisam ser cadastradas diretamente na aba de configurações de cada repositório.

Antes de utilizar a pipeline no seu projeto, acesse **Settings > Secrets and variables > Actions > New repository secret** do repositório da sua aplicação e cadastre:

| Secret | Descrição | Obrigatório? |
| --- | --- | --- |
| `COOLIFY_WEBHOOK` | URL do Webhook do app correspondente dentro do Coolify. | **Sim** |
| `COOLIFY_TOKEN` | Bearer Token de autenticação da API do Coolify. | **Sim** |
| `CF_BYPASS_HEADER_TOKEN` | Token do header `x-cf-bypass-token` para o WAF da Cloudflare. | Opcional |

---

## 📖 Como Usar no seu Projeto (Guia Passo a Passo)

Para integrar qualquer microsserviço ou API da organização a esta esteira unificada, siga o procedimento abaixo:

### Passo 1: Garantir que o projeto possui um Dockerfile

Certifique-se de que o repositório da sua aplicação possui um `Dockerfile` válido localizado na **raiz do projeto**.

### Passo 2: Criar o arquivo de Workflow

Dentro do seu repositório de projeto (ex: `descubra-muriae-backend`), crie o arquivo no caminho:
`.github/workflows/cd.yml`

### Passo 3: Colar o código de integração

Cole o conteúdo abaixo dentro do arquivo criado:

```yaml
name: CD

on:
  push:
    branches: [ main ]
  workflow_dispatch:

jobs:
  deploy:
    uses: descubra-muriae/workflow-templates/.github/workflows/cd-template.yml@main
    secrets: inherit

```

> 💡 **Como funciona o `secrets: inherit`?**
> Ele passa automaticamente as secrets salvas no seu repositório privado (`COOLIFY_WEBHOOK`, `COOLIFY_TOKEN`, etc.) para o template reutilizável, validando os campos obrigatórios e garantindo a segurança sem expor valores.

---

## 🔒 Segurança e Boas Práticas

* **Soberania do Código:** O workflow reutilizável roda exclusivamente no contexto e no ambiente do seu repositório chamador. As secrets do seu projeto nunca são expostas para terceiros.
* **Versionamento do Template:** Por padrão, utilizamos `@main` para herdar melhorias do template de forma transparente. Se precisar travar a versão da pipeline para um projeto crítico, você pode apontar para uma tag específica (ex: `@v1.0.0`).
* **Acionamento Manual:** O gatilho `workflow_dispatch` permite que qualquer membro autorizado da equipe force um deploy manual através da aba *Actions* do GitHub.
