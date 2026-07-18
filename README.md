# Trabalho Final — Containers

Aplicação de gerenciamento de livros do Web Academy executada com Docker Compose.

A infraestrutura contém:

- banco de dados MySQL;
- backend em Node.js e TypeScript;
- frontend em React e TypeScript servido pelo Nginx;
- PHPMyAdmin para gerenciamento do banco de dados.

## Arquitetura

| Serviço        | Porta no contêiner | Porta externa |
| -------------- | -----------------: | ------------: |
| MySQL          |               3306 |          3306 |
| Backend        |               4444 |          4444 |
| Frontend/Nginx |                 80 |          8000 |
| PHPMyAdmin     |                 80 |          8080 |

O backend acessa o banco pelo hostname `mysql`, que corresponde ao nome do serviço definido no Docker Compose.

## Pré-requisitos

- Docker Engine;
- Docker Compose;
- portas `3306`, `4444`, `8000` e `8080` disponíveis.

Verificação de portas:

```bash
sudo ss -ltnp | grep -E ':(3306|4444|8000|8080)\b' || true
```

Caso exista um MySQL local ocupando a porta `3306`, no Ubuntu ele pode ser interrompido com:

```bash
sudo systemctl stop mysql
```

Para iniciar novamente:

```bash
sudo systemctl start mysql
```

## Execução

Clone o repositório:

```bash
git clone https://github.com/matiusX/tp-final-webacademy.git
cd tp-final-webacademy
```

Construir as imagens e iniciar os serviços:

```bash
docker compose up -d --build
```

Conferir estado:

```bash
docker compose ps
```

O MySQL deve aparecer com o estado `healthy`.

## Acessos

### Frontend

```text
http://localhost:8000
```

### Backend

```text
http://localhost:4444/api
```

Listagem de livros:

```text
http://localhost:4444/api/livros
```

### PHPMyAdmin

```text
http://localhost:8080
```

Credenciais:

```text
Servidor: mysql
Usuário: root
Senha: senhasegura
Banco: livraria
```

### MySQL

```text
Host: localhost
Porta: 3306
Usuário: root
Senha: senhasegura
Banco: livraria
```

## Configurações do backend

O arquivo `backend/.env` contém:

```env
ESCREVER_LOG_ARQUIVO=true
PORTA=4444
DATABASE_URL="mysql://root:senhasegura@mysql:3306/livraria"
```

Os arquivos de log são armazenados em:

```text
/app/log
```

Esse diretório está associado ao volume persistente `backend_logs`.

## Configurações do frontend

O arquivo `frontend/.env` contém:

```env
VITE_ROTA_LIVROS=http://localhost:4444/api/livros
VITE_ROTA_INFOS_API=http://localhost:4444/api
VITE_API_BASE_URL=http://localhost:4444/api
```

As variáveis são aplicadas durante a construção da aplicação React.

## Inicialização do banco

Na primeira execução, o MySQL executa automaticamente:

```text
bd/script.sql
bd/02-dados.sql
```

O primeiro arquivo cria a tabela `livros`. O segundo insere os seis livros iniciais.

Para consultar a quantidade de livros:

```bash
docker compose exec mysql \
  mysql -uroot -psenhasegura -D livraria \
  -e "SELECT COUNT(*) AS total_livros FROM livros;"
```

Para listar os livros com codificação UTF-8:

```bash
docker compose exec mysql \
  mysql --default-character-set=utf8mb4 \
  -uroot -psenhasegura -D livraria \
  -e "SELECT isbn, nome FROM livros;"
```

## Volumes

O projeto utiliza dois volumes:

```text
mysql_data
backend_logs
```

O volume `mysql_data` armazena os arquivos do MySQL em:

```text
/var/lib/mysql
```

O volume `backend_logs` armazena os logs do backend em:

```text
/app/log
```

Para conferir os volumes:

```bash
docker volume ls | grep tp-final-webacademy
```

## Logs

Todos os serviços:

```bash
docker compose logs
```

Backend:

```bash
docker compose logs backend
```

MySQL:

```bash
docker compose logs mysql
```

Arquivos persistentes do backend:

```bash
docker compose exec backend ls -lah /app/log
```

## Parar os serviços

Para remover os contêineres e preservar os volumes:

```bash
docker compose down
```

Para iniciar novamente:

```bash
docker compose up -d
```

Os registros do banco e os logs continuarão armazenados.

## Reinicialização completa

Este comando remove também os volumes e todos os dados persistentes:

```bash
docker compose down -v
```

Depois, para recriar o ambiente:

```bash
docker compose up -d --build
```

`down -v` somente quando for apagar completamente os dados do MySQL e os logs.

## Estrutura

```text
.
├── backend
│   ├── Dockerfile
│   ├── example.env
│   ├── package.json
│   ├── package-lock.json
│   ├── prisma
│   │   └── schema.prisma
│   ├── src
│   │   ├── app.ts
│   │   ├── common
│   │   │   ├── Dotenv.ts
│   │   │   ├── Formatador.ts
│   │   │   ├── Logger.ts
│   │   │   └── ObjectMapper.ts
│   │   ├── dominio
│   │   │   ├── dto
│   │   │   │   ├── LivroDTO.ts
│   │   │   │   ├── LivroParaAtualizarDTO.ts
│   │   │   │   └── LivroParaCriarDTO.ts
│   │   │   ├── excecoes
│   │   │   │   ├── ErroBase.ts
│   │   │   │   ├── ISBNUnicoError.ts
│   │   │   │   ├── RegistroNaoEncontradoError.ts
│   │   │   │   ├── RegistroNaoSalvoError.ts
│   │   │   │   └── ValidacaoError.ts
│   │   │   ├── modelos
│   │   │   │   └── Livro.ts
│   │   │   ├── objectmapper
│   │   │   │   ├── LivroDTOMapper.ts
│   │   │   │   └── LivroParaCriarMapper.ts
│   │   │   ├── repositorios
│   │   │   │   ├── BaseRepository.ts
│   │   │   │   └── LivrosRepository.ts
│   │   │   ├── servicos
│   │   │   │   └── LivrosService.ts
│   │   │   └── validacoes
│   │   │       └── ValidadorLivro.ts
│   │   ├── infraestrutura
│   │   │   ├── bd
│   │   │   │   ├── Entidades.ts
│   │   │   │   ├── index.ts
│   │   │   │   └── LivroEntityMapper.ts
│   │   │   ├── controllers
│   │   │   │   ├── HomeController.ts
│   │   │   │   └── LivrosController.ts
│   │   │   ├── middlewares
│   │   │   │   ├── capturarErro404.ts
│   │   │   │   ├── capturarExcecoes.ts
│   │   │   │   └── imprimirLogRequisicoes.ts
│   │   │   ├── repositorios
│   │   │   │   └── prisma
│   │   │   │       └── PrismaLivrosRepository.ts
│   │   │   └── rotas
│   │   │       ├── index.ts
│   │   │       └── livros.ts
│   │   └── servidor.ts
│   └── tsconfig.json
├── bd
│   ├── 02-dados.sql
│   ├── dados-livros.txt
│   └── script.sql
├── docker-compose.yml
├── frontend
│   ├── Dockerfile
│   ├── env.example
│   ├── index.html
│   ├── package.json
│   ├── package-lock.json
│   ├── public
│   │   ├── vite.svg
│   │   └── webacademylogo.png
│   ├── src
│   │   ├── App.tsx
│   │   ├── assets
│   │   │   └── react.svg
│   │   ├── data
│   │   │   ├── dto
│   │   │   │   ├── ErroApiDTO.ts
│   │   │   │   ├── InformacoesApi.ts
│   │   │   │   ├── LivroDTO.ts
│   │   │   │   └── TipoErroApiEnum.ts
│   │   │   ├── hooks
│   │   │   │   └── useInfosApi.ts
│   │   │   ├── services
│   │   │   │   └── ApiService.ts
│   │   │   └── utils
│   │   │       └── index.ts
│   │   ├── main.tsx
│   │   ├── pages
│   │   │   ├── FormularioLivroPage.tsx
│   │   │   ├── ListagemLivrosPage.tsx
│   │   │   └── VisualizacaoLivroPage.tsx
│   │   ├── routes.tsx
│   │   ├── ui
│   │   │   ├── components
│   │   │   │   ├── Alerta
│   │   │   │   │   └── index.tsx
│   │   │   │   ├── Cabecalho
│   │   │   │   │   ├── index.style.tsx
│   │   │   │   │   └── index.tsx
│   │   │   │   ├── Container
│   │   │   │   │   ├── index.style.tsx
│   │   │   │   │   └── index.tsx
│   │   │   │   ├── ModalDeletarLivro
│   │   │   │   │   └── index.tsx
│   │   │   │   ├── Rodape
│   │   │   │   │   ├── index.style.tsx
│   │   │   │   │   └── index.tsx
│   │   │   │   └── TextFieldMask
│   │   │   │       └── index.tsx
│   │   │   ├── contexts
│   │   │   │   ├── alertas.tsx
│   │   │   │   └── listagem-livros.tsx
│   │   │   ├── styles
│   │   │   │   ├── global.css
│   │   │   │   └── pages
│   │   │   │       └── LivrosPage.style.tsx
│   │   │   └── themes
│   │   │       └── index.ts
│   │   └── vite-env.d.ts
│   ├── tsconfig.json
│   ├── tsconfig.node.json
│   └── vite.config.ts
└── README.md

```
