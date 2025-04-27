# **Criando um Azure Data Factory com Azure DevOps e Shell Script**

## **Índice**

- [**Criando um Azure Data Factory com Azure DevOps e Shell Script**](#criando-um-azure-data-factory-com-azure-devops-e-shell-script)
  - [**Índice**](#índice)
  - [**Introdução**](#introdução)
  - [**Configurando o Repositório no Azure DevOps**](#configurando-o-repositório-no-azure-devops)
    - [**1. Criar um Projeto no Azure DevOps**](#1-criar-um-projeto-no-azure-devops)
    - [**2. Conectar o Repositório ao Azure Data Factory**](#2-conectar-o-repositório-ao-azure-data-factory)
  - [**Automatizando a Criação do Azure Data Factory com Shell Script**](#automatizando-a-criação-do-azure-data-factory-com-shell-script)
    - [**Pré-requisitos**](#pré-requisitos)
    - [**Script de Criação**](#script-de-criação)
  - [**Resumo**](#resumo)
  - [**Insights e Boas Práticas**](#insights-e-boas-práticas)
  - [**Referências**](#referências)


## **Introdução**

O Azure Data Factory (ADF) é um serviço de integração de dados baseado na nuvem que permite criar fluxos de dados escaláveis e orquestrar pipelines de ETL/ELT. Combinado ao Azure DevOps, é possível automatizar completamente seu ciclo de vida — desde o versionamento até a implantação — promovendo governança e agilidade.

Este artigo apresenta um guia prático para configurar um repositório no Azure DevOps vinculado ao ADF, além de automatizar sua criação via Shell Script, ideal para pipelines CI/CD.


## **Configurando o Repositório no Azure DevOps**

Antes de realizar a automação, é fundamental vincular o Azure Data Factory a um repositório Git. Siga os passos abaixo:

### **1. Criar um Projeto no Azure DevOps**

- Acesse o portal do [Azure DevOps](https://dev.azure.com)
- Crie um novo projeto e configure um repositório Git (pode ser público ou privado)
- Defina a estrutura de branches (ex: `main`, `dev`)

### **2. Conectar o Repositório ao Azure Data Factory**

No portal do Azure:

- Acesse seu Data Factory  
- Clique em **"Set up code repository"**
- Escolha **Azure DevOps Git**
- Autentique e selecione o projeto, repositório e branch  
- Clique em **Apply**

A partir desse momento, as alterações feitas no ADF serão versionadas no Git, permitindo colaboração e rastreabilidade.


## **Automatizando a Criação do Azure Data Factory com Shell Script**

A automação é essencial para ambientes controlados. Com o Azure CLI via Shell Script, é possível criar e configurar um ADF automaticamente.

### **Pré-requisitos**

- Azure CLI instalado e autenticado (`az login`)
- Subscription ID e grupo de recursos criados

### **Script de Criação**

```bash
#!/bin/bash

# Variáveis
RESOURCE_GROUP="rg-datafactory-dev"
LOCATION="eastus"
ADF_NAME="adf-devops-demo"
REPO_NAME="meu-repo-adf"
PROJECT_NAME="meu-projeto"
ORG_NAME="https://dev.azure.com/minha-organizacao"
ADF_BRANCH="main"

# Criar grupo de recursos (se necessário)
az group create --name $RESOURCE_GROUP --location $LOCATION

# Criar Data Factory
az datafactory create \
  --resource-group $RESOURCE_GROUP \
  --factory-name $ADF_NAME \
  --location $LOCATION

# Configurar repositório Git no ADF
az datafactory configure-factory-repo \
  --factory-name $ADF_NAME \
  --resource-group $RESOURCE_GROUP \
  --repository-name $REPO_NAME \
  --collaboration-branch $ADF_BRANCH \
  --git-url "$ORG_NAME/$PROJECT_NAME/_git/$REPO_NAME" \
  --factory-git-configuration "{accountName:'minha-organizacao',projectName:'$PROJECT_NAME',repositoryName:'$REPO_NAME',collaborationBranch:'$ADF_BRANCH',rootFolder:'/',tenantId:'<TENANT_ID>'}"
```

> **Nota:** Substitua `<TENANT_ID>` pelo seu tenant ID do Azure Active Directory, que pode ser obtido via `az account show`.


## **Resumo**

Este artigo demonstrou como integrar o Azure Data Factory ao Azure DevOps para versionamento e controle de código, além de automatizar sua criação usando um shell script. Isso permite:

- Maior controle sobre versões e mudanças
- Automatização do provisionamento de ambientes
- Implantação contínua com segurança e rastreabilidade


## **Insights e Boas Práticas**

- **Infra como código (IaC):** Integre o script com ferramentas como Terraform ou Bicep para controle mais avançado.
- **Branches separados:** Utilize branches `dev`, `qa` e `main` para gerenciar ambientes distintos.
- **Validar antes de publicar:** Utilize `Validate all` no portal do ADF antes de `Publish` para evitar que erros sejam propagados.
- **CI/CD pipelines:** Crie pipelines no Azure DevOps que usem o script para ambientes de testes, staging e produção.


## **Referências**

- [Documentação Oficial Azure Data Factory](https://learn.microsoft.com/azure/data-factory/)
- [Documentação Azure CLI para ADF](https://learn.microsoft.com/cli/azure/datafactory)
- [Integração do ADF com Git](https://learn.microsoft.com/azure/data-factory/source-control)
