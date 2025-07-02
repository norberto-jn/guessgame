## Passo a Passo para Rodar o Projeto em Sua Máquina

Para configurar e executar o projeto em sua máquina, siga os passos abaixo:

1.  **Criar o diretório do projeto:**
    Crie uma pasta para organizar o back-end e o front-end:

    ```bash
    mkdir project
    ```

2.  **Acessar o diretório criado:**
    Entre na pasta `project` que você acabou de criar:

    ```bash
    cd project
    ```

3.  **Clonar o repositório do projeto:**
    Agora, clone o repositório `guessgame` dentro da pasta `project`:

    ```bash
    git clone https://github.com/norberto-jn/guessgame.git
    ```

4.  **Instalar Docker Compose (se necessário):**
    Se você ainda não tem o Docker Compose instalado em sua máquina, siga o tutorial de instalação oficial disponível [aqui](https://docs.docker.com/compose/install/).

5.  **Iniciar o Docker Compose:**
    Com o Docker Compose instalado, e ainda dentro da pasta `project`, execute o seguinte comando para subir os serviços:

    ```bash
    docker compose up -d
    ```

6.  **Verificar o status do projeto:**
    Após o Docker Compose iniciar os contêineres, você pode verificar o status do projeto com:

    ```bash
    docker compose ps
    ```

7.  **Acessar a aplicação no navegador:**
    Abra seu navegador e acesse a URL para interagir com a aplicação:

    ```
    http://localhost:80
    ```

-----

### Estrutura de Pastas

Após clonar o repositório e iniciar o projeto, a estrutura de pastas deve ser similar a esta:

```
./
├── guessgame
│   ├── guessgame-api
│   ├── guessgame-ui
│   ├── nginx
│   └── postgres
```
guessgame-api :
API principal do projeto, responsável pela lógica de negócio e pelo processamento das requisições do frontend.

guessgame-ui :
Interface web do projeto (frontend), onde os usuários interagem com o jogo.

nginx :
Responsável pela configuração do proxy reverso. Ele expõe a aplicação para acesso externo, garantindo que o cliente não acesse diretamente a API ou o frontend.

postgres :
Banco de dados da aplicação, utilizado para persistência dos dados.

-----

### Estrutura de Dockerfiles

guessgame-api :

```Dockerfile
# Usa a imagem base do Python 3.10 com menos camadas extras (slim)
# A versão 3.10 foi escolhida com base nos requisitos do backend.
FROM python:3.10-slim

# Define o diretório de trabalho dentro do container
WORKDIR /usr/src/workspace/guessgame-api

# Copia todos os arquivos do diretório atual para o diretório de trabalho no container
COPY . .

# Desativa a barra de progresso do pip para reduzir logs
# Código de referência: https://github.com/fams/gcs-test/blob/main/Dockerfile
RUN pip config --user set global.progress_bar off

# Instala as dependências listadas em requirements.txt
# Código de referência: https://github.com/fams/gcs-test/blob/main/Dockerfile
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Expõe a porta 5000, usada pela aplicação Flask
EXPOSE 5000

# Define variáveis de ambiente para o Flask
ENV FLASK_APP=run.py
ENV FLASK_RUN_HOST=0.0.0.0

# Executa o script de inicialização da aplicação (entrypoint)
# CMD ["flask", "run", "--host=0.0.0.0"]

CMD ["./start-backend.sh"]
```

guessgame-ui

```Dockerfile
# FROM node:18.17.0-alpine3.19
# Usa a imagem base do Node.js v20.17.0 com Alpine Linux 3.19
FROM node:20.17.0-alpine3.19 

# Define o diretório de trabalho dentro do container
WORKDIR /usr/src/workspace/guessgame-ui

# Comando que verifica se a pasta node_modules existe; 
# se sim, executa 'npm run start'; se não, instala as dependências primeiro com 'npm i' e depois inicia
# Código de referência: https://github.com/norberto-jn/cit-api/blob/main/Dockerfile.dev
CMD [ -d "node_modules" ] && npm run start || npm i && npm run start

# Expõe a porta 3000, usada pela aplicação React
EXPOSE 3000
```

-----

### Estrutura do docker-compose.yaml

```docker-compose.yaml
# Código de referência: https://github.com/norberto-jn/cit-api/blob/main/docker-compose.yaml
# Versão da especificação do Docker Compose utilizada
version: '3'

# Definição de redes customizadas para comunicação entre containers
networks:
  guessgame-network:
    # Rede do tipo bridge, padrão para containers se comunicarem entre si
    driver: bridge

services:

  # Serviço do banco de dados PostgreSQL
  guessgame-postgres:
    # Imagem oficial do PostgreSQL na versão 15
    image: postgres:15
    # Nome do container
    container_name: guessgame-postgres
    # Reinicia o container automaticamente em caso de falha
    restart: always
    # Variáveis de ambiente do banco de dados
    environment:
      POSTGRES_DB: postgres
      POSTGRES_USER: postgres
      # Senha base no codigo : https://github.com/fams/guess_game/blob/main/start-backend.sh
      POSTGRES_PASSWORD: secretpass
    # Volume para persistência de dados do banco
    volumes:
      - ./postgres:/var/lib/postgresql/data
    # Mapeamento de porta: 5440 (host) → 5432 (container)
    ports:
      - "5440:5432"
    # Conecta o container à rede definida
    networks:
      - guessgame-network

  # Serviço da API do jogo (backend)
  guessgame-api:
    # Caminho do código-fonte da API
    build:
      context: ./guessgame-api
      # Dockerfile usado para o build (ambiente de desenvolvimento)
      dockerfile: Dockerfile.dev
    # Nome do container
    container_name: guessgame-api
    # Reinicia o container automaticamente em caso de falha
    restart: always
    # Monta o código local dentro do container para facilitar o desenvolvimento
    volumes:
      - ./guessgame-api/:/usr/src/workspace/guessgame-api
    # Conecta o container à rede definida
    networks:
      - guessgame-network

  # Serviço da interface web do jogo (frontend React)
  guessgame-ui:
    # Caminho do código-fonte da interface
    build:
      context: ./guessgame-ui
      # Dockerfile usado no ambiente de desenvolvimento
      dockerfile: Dockerfile.dev
    # Nome do container
    container_name: guessgame-ui
    # Reinicia o container automaticamente em caso de falha
    restart: always
    # Variável de ambiente com o endpoint da API para o frontend
    environment:
      REACT_APP_BACKEND_URL: http://localhost/api
    # Monta o código local dentro do container
    volumes:
      - ./guessgame-ui/:/usr/src/workspace/guessgame-ui
    # Conecta o container à rede definida
    networks:
      - guessgame-network

  # Serviço do servidor Nginx (proxy reverso)
  nginx:
    # Imagem mais recente do Nginx
    image: nginx:latest
    # Nome do container
    container_name: guessgame-nginx
    # Reinicia o container automaticamente em caso de falha
    restart: always
    # Monta o arquivo de configuração customizado do Nginx (somente leitura)
    volumes:
       - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    # Porta 80 do host exposta como 80 do container
    ports:
      - "80:80"
    # Conecta o container à rede definida
    networks:
      - guessgame-network
    # Aguarda os serviços da API e UI estarem prontos antes de iniciar
    depends_on:
      - guessgame-api
      - guessgame-ui
```

-----

### Estrutura de ./guessgame/nginx/nginx.conf

```nginx.conf
# Código de referência: https://nginx.org/en/docs/example.html
# Bloco 'events' define configurações relacionadas à manipulação de conexões.
events {
    # Define o número máximo de conexões simultâneas permitidas por processo worker.
    worker_connections 1024;
}

# Bloco 'http' configura o comportamento geral do NGINX para requisições HTTP.
http {
    # Inclui a lista de tipos MIME para definir o Content-Type dos arquivos servidos.
    include       mime.types;

    # Define o tipo de conteúdo padrão quando não é possível determinar o tipo do arquivo.
    default_type  application/octet-stream;

    # Habilita o uso de sendfile() para melhorar a performance ao servir arquivos estáticos.
    sendfile        on;

    # Define o tempo de espera (em segundos) para conexões keep-alive.
    keepalive_timeout  65;

    # Bloco do servidor que escuta na porta 80 (HTTP).
    server {
        # Define a porta de escuta e o nome do servidor.
        listen 80;
        server_name localhost;

        # Local para a raiz do site (frontend).
        location / {
            # Redireciona as requisições para o serviço 'guessgame-ui' rodando na porta 3000.
            proxy_pass http://guessgame-ui:3000;

            # Encaminha os cabeçalhos do cliente original para o backend.
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # Local para as requisições da API.
        location /api/ {
            # Redireciona as requisições para o serviço 'guessgame-api' rodando na porta 5000.
            proxy_pass http://guessgame-api:5000/;

            # Encaminha os mesmos cabeçalhos do cliente para manter o rastreamento correto.
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```


Compreendido. Aqui estão as explicações dos volumes no formato Markdown, exatamente como você solicitou:

-----

### Explicação dos Volumes no seu `docker-compose.yml`

#### 1\. `guessgame-postgres`

```yaml
volumes:
  - ./postgres:/var/lib/postgresql/data
```

Esse volume monta a pasta local **./postgres** no host para dentro do container, no caminho **/var/lib/postgresql/data**. O PostgreSQL armazena seus dados persistentes nesse caminho, então esse volume garante que os dados do banco não se percam mesmo que o container seja reiniciado ou destruído.


#### 2\. `guessgame-api`

```yaml
volumes:
  - ./guessgame-api/:/usr/src/workspace/guessgame-api
```

Esse volume mapeia o código da API (no host, pasta **./guessgame-api/**) para dentro do container, em **/usr/src/workspace/guessgame-api**. Isso permite atualizar o código na máquina local e ver as mudanças no container em tempo real, sem precisar rebuildar a imagem.


#### 3\. `guessgame-ui`

```yaml
volumes:
  - ./guessgame-ui/:/usr/src/workspace/guessgame-ui
```

Mesma lógica do anterior, mas para o projeto da interface. Permite ver as mudanças no frontend automaticamente.



#### 4\. `nginx`

```yaml
volumes:
  - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
```

  - Esse volume monta um arquivo de configuração personalizado do Nginx (**nginx.conf**) do host para dentro do container.
  - O sufixo **:ro** indica que o arquivo será acessado em modo somente leitura dentro do container, evitando alterações acidentais.