# Criando Processos de Redundância de Arquivos na Azure

## 📁 Parte 1: Solução Automatizada do Desafio

### 💾 Redundância de Arquivos na Azure com Data Factory e Databricks

### 🎯 Objetivo
Implementar uma solução de redundância de arquivos utilizando os serviços da Azure, copiando arquivos de uma fonte on-premise para um Data Lake, utilizando Data Factory, integração com SQL Database e organização em camadas (bronze, prata, ouro).


### 🧱 Recursos Criados

1. **Azure Data Factory**
2. **SQL Database**
3. **SQL Server**
4. **Storage Account (Data Lake Gen2 habilitado)**


### 🛠️ Etapas e Scripts

#### 1. 🔧 Criação dos Recursos via CLI

```bash
# Variáveis
RG="rg-redundancia"
LOCATION="eastus"
STORAGE_NAME="redundanciastorage$RANDOM"
DATAFACTORY_NAME="redundanciaADF$RANDOM"
SQL_SERVER_NAME="redundanciasqlsrv$RANDOM"
SQL_DB_NAME="redundanciadb"

# Criar grupo de recursos
az group create --name $RG --location $LOCATION

# Criar Storage Account com Data Lake Gen2
az storage account create \
  --name $STORAGE_NAME \
  --resource-group $RG \
  --location $LOCATION \
  --sku Standard_LRS \
  --kind StorageV2 \
  --hierarchical-namespace true

# Criar Azure SQL Server
az sql server create \
  --name $SQL_SERVER_NAME \
  --resource-group $RG \
  --location $LOCATION \
  --admin-user sqladmin \
  --admin-password "YourP@ssword123"

# Criar SQL Database
az sql db create \
  --resource-group $RG \
  --server $SQL_SERVER_NAME \
  --name $SQL_DB_NAME \
  --service-objective S0

# Criar Azure Data Factory
az datafactory create \
  --resource-group $RG \
  --factory-name $DATAFACTORY_NAME \
  --location $LOCATION
```


#### 2. 📁 Organização do Data Lake

```bash
# Criar containers e diretórios bronze, prata e ouro
az storage container create --name raw --account-name $STORAGE_NAME
az storage fs directory create -n bronze --file-system raw --account-name $STORAGE_NAME
az storage fs directory create -n prata --file-system raw --account-name $STORAGE_NAME
az storage fs directory create -n ouro --file-system raw --account-name $STORAGE_NAME
```


#### 3. 🔌 Configuração no Data Factory

- Criar **Linked Services**:
  - Azure SQL Database (On-Prem via Integration Runtime)
  - Azure Data Lake Storage Gen2

- Criar um **Integration Runtime Self-hosted** para acesso on-premise:
  - Instalar e configurar localmente (https://learn.microsoft.com/en-us/azure/data-factory/create-self-hosted-integration-runtime)


#### 4. 🧪 Pipeline de Cópia

- Criar pipeline no Data Factory:
  - Fonte: Tabela `produtos` no SQL Database on-premise
  - Destino: Data Lake – diretório `bronze/produtos.txt`
  - Usar atividade **Copy Data**
  - Configurar transformação opcional (ex: renomear colunas ou filtrar)


### 🧠 Boas Práticas

- Separação por camadas (bronze, prata, ouro) permite versionamento e governança de dados.
- Use triggers no Data Factory para automatizar execuções.
- Versione seu pipeline com JSON exportado via Azure DevOps.


### 📚 Referências Importantes

- [Documentação Azure Data Factory](https://learn.microsoft.com/azure/data-factory/introduction)
- [Integration Runtime Self-hosted](https://learn.microsoft.com/en-us/azure/data-factory/create-self-hosted-integration-runtime)
- [Camadas de dados: bronze, prata, ouro](https://learn.microsoft.com/en-us/azure/architecture/example-scenario/data/data-lake)
- [CLI de Azure](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)


### 🚀 Conclusão

Este projeto configura uma pipeline completa de redundância e ingestão de dados. Os dados saem de uma base on-premise e são organizados em um Data Lake moderno, prontos para processamento com Databricks, Synapse ou Power BI.


## 📝 Parte 2: Artigo – Insights do Desafio: Redundância de Arquivos na Azure


### Desafio Microsoft AI for Tech: Redundância de Arquivos na Azure

#### 📌 Contexto

Durante o bootcamp **Microsoft AI for Tech**, realizamos um desafio com foco em engenharia de dados utilizando a Azure. A missão? Criar um fluxo de redundância de dados a partir de uma base on-premise, organizando tudo em um Data Lake e utilizando o Azure Data Factory como motor de orquestração.


#### 🛠️ A Arquitetura

Os serviços principais que compõem a arquitetura são:

- **Azure Data Factory**: responsável pela orquestração dos dados.
- **SQL Server (on-premise)**: a fonte original dos dados.
- **Storage Account com Data Lake Gen2**: destino dos arquivos.
- **Integration Runtime Self-hosted**: ponte segura entre o ambiente local e a nuvem.


#### 🔄 Processo de Cópia

1. **Criação dos Recursos** – Utilizamos a CLI da Azure para criar todos os recursos, o que traz agilidade e padronização.
2. **Conexão com Fonte On-Premise** – Foi necessário instalar e configurar um Integration Runtime local. Essa parte foi um pouco mais trabalhosa, mas essencial para o processo.
3. **Pipeline de Cópia** – Criamos um pipeline no Data Factory que extrai os dados da tabela `produtos`, transforma para `.txt` e armazena na camada bronze do Data Lake.


#### 💡 Insights e Aprendizados

- **Modularização é tudo**: trabalhar com camadas (bronze, prata e ouro) permite maior controle da qualidade dos dados.
- **A automação vale a pena**: scripts com Azure CLI economizam tempo e reduzem erros.
- **Segurança importa**: o uso do self-hosted IR traz robustez à comunicação com bases locais.
- **Escalabilidade**: essa estrutura está pronta para ser escalada para milhões de registros e múltiplas tabelas.


#### 🔮 Próximos Passos

- Integrar com Databricks para transformar os dados da camada bronze para prata e ouro.
- Criar visualizações com Power BI conectando diretamente ao Lakehouse.
- Automatizar as execuções com triggers baseadas em tempo ou eventos.


#### 📘 Conclusão

Esse desafio mostrou como é possível unir diferentes serviços da Azure para criar uma arquitetura robusta e escalável de ingestão de dados. Com poucos cliques (ou comandos), você sai de uma base local para uma estrutura moderna de Data Lake, pronta para análise e inteligência artificial.

