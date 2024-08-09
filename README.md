
# Setup do Ambiente de Lab com Docker Compose

Este projeto configura um ambiente de laboratório utilizando `Docker Compose`, que inclui vários serviços essenciais como PostgreSQL, Kafka, Kafka Connect, MinIO, MongoDB, e mais. 

## Índice

1. [Pré-requisitos](#pré-requisitos)
2. [Configuração de Pastas Locais](#configuração-de-pastas-locais)
3. [Como Executar](#como-executar)
4. [Serviços e Portas](#serviços-e-portas)
5. [Acesso e Credenciais](#acesso-e-credenciais)
6. [Dockerfile para Kafka Connect com Plugins Customizados](#dockerfile-para-kafka-connect-com-plugins-customizados)
7. [Considerações](#considerações)

## Pré-requisitos

Certifique-se de ter o seguinte instalado em sua máquina:

- [Docker](https://www.docker.com/get-started)
- [Docker Compose](https://docs.docker.com/compose/install/)

## Configuração de Pastas Locais

Antes de iniciar os serviços, é necessário criar as pastas locais que serão montadas nos volumes dos containers. As instruções a seguir são para sistemas operacionais diferentes:

### Linux / MacOS

Execute os seguintes comandos no terminal para criar as pastas:

```bash
mkdir -p ~/docker_files/volumes/lab/postgresql
mkdir -p ~/docker_files/volumes/lab/kafka
mkdir -p ~/docker_files/volumes/lab/minio/data
mkdir -p ~/docker_files/volumes/lab/mongodb
```

### Windows

Para criar as pastas no Windows, execute os seguintes comandos no PowerShell:

```powershell
New-Item -ItemType Directory -Path "$HOME\docker_filesolumes\lab\postgresql"
New-Item -ItemType Directory -Path "$HOME\docker_filesolumes\lab\kafka"
New-Item -ItemType Directory -Path "$HOME\docker_filesolumes\lab\minio\data"
New-Item -ItemType Directory -Path "$HOME\docker_filesolumes\lab\mongodb"
```

## Como Executar

1. Clone o repositório ou copie o arquivo `docker-compose.yml` para o seu diretório de trabalho.
2. Certifique-se de estar no diretório correto onde o arquivo `docker-compose.yml` está localizado.
3. Execute o seguinte comando para iniciar todos os serviços:

```bash
docker-compose up -d
```

Este comando iniciará os containers em modo *detached*, permitindo que continuem rodando em segundo plano.

## Serviços e Portas

Aqui está uma lista dos serviços configurados e as portas expostas para acesso:

| Serviço               | Descrição                       | Porta Host  | Porta Container |
|-----------------------|---------------------------------|-------------|-----------------|
| **PostgreSQL**        | Banco de dados relacional       | 5432        | 5432            |
| **Kafka Broker**      | Servidor Kafka                  | 9092, 9101  | 9092, 9101      |
| **Kafka Connect**     | Plataforma de integração Kafka  | 8083        | 8083            |
| **Redpanda Console**  | Interface web para Kafka        | 8080        | 8080            |
| **MinIO**             | Armazenamento S3-like           | 9050, 9051  | 9000, 9001      |
| **MongoDB**           | Banco de dados NoSQL            | 27017       | 27017           |
| **Mongo Express**     | Interface web para MongoDB      | 8091        | 8081            |

## Acesso e Credenciais

Aqui estão as credenciais de acesso para os serviços:

### PostgreSQL
- **Usuário**: `postgres`
- **Senha**: `postgres`
- **Banco de Dados**: `postgres`

### Kafka Connect
- **URL do Kafka Connect**: `http://localhost:8083`

### MinIO
- **Usuário**: `admin`
- **Senha**: `minioadmin`
- **Console**: `http://localhost:9051`

### MongoDB
- **Usuário**: `root`
- **Senha**: `password`
- **Mongo Express**: `http://localhost:8091`
  - **Usuário**: `admin`
  - **Senha**: `pass`

## Dockerfile para Kafka Connect com Plugins Customizados

Este `Dockerfile` é utilizado para criar uma imagem personalizada do Kafka Connect com diversos plugins adicionais, como conectores para S3, Azure Blob Storage, e MongoDB. A imagem base utilizada é a do Strimzi Kafka, que é uma solução popular para rodar Kafka no Kubernetes.

### Estrutura do Dockerfile

1. **ARG e ENV**:
    - `ARG STRIMZI_VERSION=latest-kafka-3.7.0`: Define a versão do Kafka Strimzi a ser usada como base.
    - `ARG DEBEZIUM_CONNECTOR_VERSION=2.7.0.Final`: Define a versão do conector Debezium para PostgreSQL.
    - `ENV KAFKA_CONNECT_PLUGIN_PATH=/tmp/connect-plugins/`: Define o caminho onde os plugins serão armazenados.
    - `ENV KAFKA_CONNECT_LIBS=/opt/kafka/libs`: Define o caminho das bibliotecas do Kafka.

2. **Layer Temporário para Download e Extração**:
    - O Dockerfile começa criando uma camada temporária (`unzip-layer`) usando uma imagem Debian para instalar ferramentas como `curl` e `unzip`.
    - Nesta camada, os arquivos dos conectores para S3, Azure Blob Storage e MongoDB são baixados e extraídos.

3. **Imagem Final**:
    - A imagem final é baseada na imagem do Kafka Strimzi.
    - O conector Debezium para PostgreSQL é baixado e descompactado diretamente dentro da imagem.
    - Os plugins baixados na camada temporária são copiados para a imagem final.

### Como Utilizar

1. **Construção da Imagem**:
    - Para construir a imagem personalizada do Kafka Connect com os plugins, navegue até o diretório onde o `Dockerfile` está localizado e execute o comando:

    ```bash
    docker build -t custom-kafka-connect:latest .
    ```

    Isso criará uma imagem chamada `custom-kafka-connect:latest` contendo todos os plugins necessários.

2. **Executar com Docker Compose**:
    - Após a construção da imagem, você pode referenciá-la em um serviço Kafka Connect no `docker-compose.yml`, substituindo a imagem padrão:

    ```yaml
    kafka-connect-lab:
      image: custom-kafka-connect:latest
      ...
    ```

    Isso garantirá que sua instância do Kafka Connect seja iniciada com os plugins customizados já incluídos.

### Plugins Incluídos

- **Kafka Connect para S3** (`kafka-connect-s3-10.5.14`)
- **Kafka Connect para Azure Blob Storage** (`kafka-connect-azure-data-lake-gen2-storage-1.6.23`)
- **Kafka Connect para MongoDB** (`mongo-kafka-connect-1.13.0`)
- **Debezium Connector para PostgreSQL** (`debezium-connector-postgres-2.7.0.Final`)

Este `Dockerfile` é uma ótima forma de configurar rapidamente uma instância do Kafka Connect com todos os plugins necessários, sem precisar configurá-los manualmente após o início do container.

## Considerações

- Certifique-se de que as portas mencionadas acima estejam livres em sua máquina antes de executar o `docker-compose`.
- O serviço `redpanda-console` está configurado para funcionar com o Kafka e pode ser acessado através do navegador na porta `8080`.
- **MinIO** é um serviço de armazenamento compatível com S3 e pode ser acessado na porta `9051` para o console e `9050` para a API.
- **Mongo Express** é uma interface web para visualizar e gerenciar o banco de dados MongoDB e está acessível na porta `8091`.

## Encerramento dos Serviços

Para parar e remover todos os containers, execute:

```bash
docker-compose down
```

Isso encerrará os containers e liberará as portas utilizadas.

---

**Nota**: A configuração dos serviços foi realizada para facilitar o uso em ambientes de desenvolvimento e testes. Para ambientes de produção, considere revisitar as configurações, especialmente as relacionadas a segurança e persistência de dados.

## Script de Inicialização Automática para Kafka Connect

Um script de inicialização está incluído para facilitar a configuração automática dos conectores Kafka quando o container Kafka Connect é iniciado. Este script está localizado na pasta `utils/scripts/kafka-connect`.

### Detalhes do Script

O script `entrypoint.sh` faz o seguinte:

1. **Espera o Kafka estar pronto**: O script primeiro espera que o Kafka Broker esteja acessível na porta 29092.
2. **Inicia o Kafka Connect**: Uma vez que o Kafka está pronto, o script inicia o Kafka Connect.
3. **Espera a API REST do Kafka Connect estar disponível**: O script verifica repetidamente se a API REST do Kafka Connect está acessível na porta 8083.
4. **Registra os conectores**: Para cada arquivo `.json` encontrado na pasta `/etc/kafka-connect/connectors/`, o script faz uma requisição `PUT` para registrar o conector na instância do Kafka Connect.

### Como Utilizar

1. **Arquivo de Script**:
   - Certifique-se de que o script `entrypoint.sh` está presente na pasta `utils/scripts/kafka-connect` no seu diretório de trabalho.

2. **Configuração no Docker Compose**:
   - O serviço Kafka Connect no `docker-compose.yml` já está configurado para utilizar este script como ponto de entrada.
   - Exemplo:

    ```yaml
    kafka-connect-lab:
      image: custom-kafka-connect:latest
      ...
      volumes:
         - ./utils/scripts/kafka-connect:/scripts  
      command: /scripts/entrypoint.sh
    ```

3. **Arquivos de Conectores**:
   - Coloque todos os arquivos de configuração dos conectores em formato `.json` na pasta `./utils/configs/connectors/`.
   - O script automaticamente registrará esses conectores ao iniciar o container do Kafka Connect.

### Exemplo de Arquivo de Configuração de Conector

Um arquivo de configuração de conector típico (`./utils/configs/connectors/sample-connector.json`) pode se parecer com isto:

```json
{
  "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
  "tasks.max": "1",
  "database.hostname": "postgres-lab",
  "database.port": "5432",
  "database.user": "postgres",
  "database.password": "postgres",
  "database.dbname": "postgres",
  "topic.prefix": "postgres",
  "plugin.name": "pgoutput",
  "slot.name": "debezium_slot",
  "publication.name": "debezium_publication",
  "database.history.kafka.bootstrap.servers": "kafka-broker-lab:9092",
  "database.history.kafka.topic": "schema-changes.inventory"
}
  
```

Esse arquivo define um conector Debezium para PostgreSQL, que captura alterações na tabela `my_table` do banco de dados PostgreSQL e as publica em um tópico Kafka.

### Considerações

- Certifique-se de que as portas mencionadas no script estejam corretamente mapeadas e que os serviços correspondentes estejam em execução.
- Os arquivos `.json` devem seguir o formato de configuração do Kafka Connect para serem registrados corretamente.
