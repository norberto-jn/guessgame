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