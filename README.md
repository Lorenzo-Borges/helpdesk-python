<code>
Help Desk — Microsserviços com Python

Sistema de gerenciamento de chamados desenvolvido utilizando arquitetura de microsserviços, com duas APIs independentes, PostgreSQL e integração com Amazon S3 para armazenamento de relatórios em PDF.

1. Sobre o projeto

O projeto tem como objetivo estudar e aplicar conceitos de arquitetura de microsserviços utilizando Python e FastAPI.

A aplicação é dividida em dois serviços:

Ticket Service: responsável pelo gerenciamento de usuários e chamados.
Report Service: responsável pela consulta dos chamados, geração de relatórios em PDF e armazenamento dos arquivos no Amazon S3.

Cada microsserviço possui seu próprio banco de dados PostgreSQL, mantendo a independência dos dados e das responsabilidades.

2. Arquitetura
text
helpdesk/
│
├── ticket-service/
│   └── API de gerenciamento de chamados
│
├── report-service/
│   └── API de geração de relatórios
│
└── README.md

Fluxo da arquitetura
text
              Cliente / Postman
                      |
             +--------+--------+
             |                 |
       Ticket Service    Report Service
          :8000               :8001
             |                 |
        PostgreSQL         PostgreSQL
             |                 |
             |          Comunicação HTTP
             |                 |
             +<----------------+
                               |
                         Geração de PDF
                               |
                          Amazon S3

3. Tecnologias utilizadas
Tecnologia	Finalidade
Python	Linguagem principal
FastAPI	Desenvolvimento das APIs
PostgreSQL	Banco de dados
SQLAlchemy	ORM
Pydantic	Validação de dados
Alembic	Migrações
HTTPX	Comunicação entre APIs
ReportLab	Geração de PDFs
Boto3	Integração com Amazon S3
Uvicorn	Servidor das APIs
4. Microsserviço 1 — Ticket Service

Responsabilidade: gerenciamento de usuários e chamados.

Estrutura de pastas
text
ticket-service/
│
├── app/
│   ├── main.py
│   │
│   ├── database/
│   │   ├── connection.py
│   │   └── base.py
│   │
│   ├── models/
│   │   ├── user.py
│   │   └── ticket.py
│   │
│   ├── schemas/
│   │   ├── user.py
│   │   └── ticket.py
│   │
│   ├── routes/
│   │   ├── user_routes.py
│   │   └── ticket_routes.py
│   │
│   └── services/
│       └── ticket_service.py
│
├── migrations/
├── tests/
├── requirements.txt
└── .env

Banco de dados: tickets_db
Tabela: users
Campo	Tipo
id	UUID (PK)
name	VARCHAR(150)
email	VARCHAR(150)
role	VARCHAR(30)
created_at	TIMESTAMP
Tabela: tickets
Campo	Tipo
id	UUID (PK)
title	VARCHAR(200)
description	TEXT
status	VARCHAR(30)
priority	VARCHAR(20)
user_id	UUID (FK)
created_at	TIMESTAMP
updated_at	TIMESTAMP

Status possíveis:

OPEN
IN_PROGRESS
RESOLVED
CLOSED

Prioridades possíveis:

LOW
MEDIUM
HIGH
URGENT
Endpoints
Método	Endpoint	Descrição
POST	/users	Criar usuário
GET	/users	Listar usuários
GET	/users/{id}	Buscar usuário
POST	/tickets	Criar chamado
GET	/tickets	Listar chamados
GET	/tickets/{id}	Buscar chamado
PUT	/tickets/{id}	Atualizar chamado
DELETE	/tickets/{id}	Excluir chamado

Filtros disponíveis:

http
GET /tickets?status=OPEN
GET /tickets?priority=HIGH
GET /tickets?user_id=UUID

5. Microsserviço 2 — Report Service

Responsabilidade: consultar os chamados, gerar relatórios em PDF e armazená-los no Amazon S3.

Estrutura de pastas
text
report-service/
│
├── app/
│   ├── main.py
│   │
│   ├── database/
│   │   ├── connection.py
│   │   └── base.py
│   │
│   ├── models/
│   │   └── report.py
│   │
│   ├── schemas/
│   │   └── report.py
│   │
│   ├── routes/
│   │   └── report_routes.py
│   │
│   ├── services/
│   │   └── report_service.py
│   │
│   └── integrations/
│       ├── ticket_api.py
│       ├── pdf_generator.py
│       └── s3_storage.py
│
├── migrations/
├── tests/
├── requirements.txt
└── .env

Banco de dados: reports_db
Tabela: reports
Campo	Tipo
id	UUID (PK)
report_type	VARCHAR(50)
file_name	VARCHAR(255)
s3_key	VARCHAR(500)
total_tickets	INTEGER
generated_by	UUID
created_at	TIMESTAMP

Os arquivos PDF ficam armazenados no Amazon S3. O PostgreSQL guarda somente os metadados e a localização dos arquivos.

Endpoints
Método	Endpoint	Descrição
POST	/reports/generate	Gerar relatório
GET	/reports	Listar relatórios
GET	/reports/{id}	Buscar relatório
GET	/reports/{id}/download	Obter link para download
6. Comunicação entre microsserviços

A comunicação entre os serviços acontece através de HTTP utilizando JSON.

O Report Service consome os endpoints do Ticket Service para obter os dados necessários à geração dos relatórios.

Fluxo de geração do relatório
O cliente solicita a geração de um relatório.
O Report Service recebe a requisição.
O Report Service consulta o Ticket Service.
O Ticket Service retorna os chamados em JSON.
O Report Service processa os dados.
O Report Service gera um arquivo PDF utilizando ReportLab.
O arquivo é enviado para o Amazon S3.
Os metadados do relatório são registrados no PostgreSQL.
O serviço retorna as informações do relatório ao cliente.
7. Armazenamento no Amazon S3

O Amazon S3 será utilizado para armazenar os relatórios PDF gerados pelo sistema.

Exemplo de organização:

text
helpdesk-reports/
│
└── reports/
    ├── report-001.pdf
    ├── report-002.pdf
    └── report-003.pdf


O acesso aos arquivos será realizado através de URLs pré-assinadas (Presigned URLs), evitando a necessidade de tornar o bucket público.

8. Configuração do ambiente

O projeto será executado diretamente no ambiente Python, sem Docker.

É necessário possuir:

Python instalado.
PostgreSQL instalado e em execução.
Dois bancos de dados criados no PostgreSQL.
Credenciais AWS configuradas para integração com S3.
Criando os bancos de dados

No PostgreSQL, execute:

sql
CREATE DATABASE tickets_db;
CREATE DATABASE reports_db;

Executando o Ticket Service

Entre na pasta do serviço:

bash
cd ticket-service


Crie um ambiente virtual:

bash
python -m venv venv


Ative o ambiente virtual.

Windows:

bash
venv\Scripts\activate


Linux / macOS:

bash
source venv/bin/activate


Instale as dependências:

bash
pip install -r requirements.txt


Inicie a API:

bash
uvicorn app.main:app --reload --port 8000

Executando o Report Service

Abra outro terminal e entre na pasta do serviço:

bash
cd report-service


Crie outro ambiente virtual:

bash
python -m venv venv


Ative o ambiente virtual.

Windows:

bash
venv\Scripts\activate


Linux / macOS:

bash
source venv/bin/activate


Instale as dependências:

bash
pip install -r requirements.txt


Inicie a API:

bash
uvicorn app.main:app --reload --port 8001


Importante: cada serviço deve ser executado em seu próprio terminal, com seu próprio ambiente virtual.

9. Variáveis de ambiente

Cada microsserviço terá seu próprio arquivo .env.

Ticket Service
env
DATABASE_URL=postgresql://postgres:password@localhost:5432/tickets_db

Report Service
env
DATABASE_URL=postgresql://postgres:password@localhost:5432/reports_db

TICKET_API_URL=http://localhost:8000

AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=us-east-1
S3_BUCKET_NAME=


As credenciais AWS devem ser fornecidas de forma segura e nunca devem ser versionadas no Git.

10. Documentação das APIs

O FastAPI disponibiliza documentação automática através do Swagger.

Após iniciar os serviços:

Ticket Service: http://localhost:8000/docs
Report Service: http://localhost:8001/docs
11. Objetivos de aprendizado

O desenvolvimento deste projeto busca aplicar os seguintes conceitos:

Arquitetura de microsserviços.
Separação de responsabilidades.
Desenvolvimento de APIs REST com FastAPI.
Persistência de dados com PostgreSQL.
Modelagem e relacionamento de entidades.
Comunicação HTTP entre microsserviços.
Geração de documentos PDF.
Integração com serviços AWS.
Armazenamento de arquivos no Amazon S3.
Gerenciamento de ambientes virtuais Python.

Spring Boot                FastAPI / Python

@RestController            @app.get, @app.post

@GetMapping                @app.get

@PostMapping               @app.post

DTO                        Pydantic Schema

@Entity                    SQLAlchemy Model

JpaRepository              SQLAlchemy Session / consultas

@Service                   Classe ou módulo Service

application.properties     .env + Pydantic Settings

Hibernate                  SQLAlchemy

Maven / Gradle             pip + requirements.txt

Projeto desenvolvido para fins de estudo e prática de arquitetura Backend com Python e microsserviços.
</code>
