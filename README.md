# CuraSys — Sistema de Gestão Hospitalar

[![status](https://img.shields.io/badge/status-MVP-orange)]() [![license](https://img.shields.io/badge/license-MIT-blue)]()

**CuraSys** é um sistema de gestão hospitalar pensado para pequenas clínicas e hospitais — começando por um MVP leve (SQLite + Flask + HTML/JS) e evoluindo para uma versão profissional (PostgreSQL + FastAPI + React + TypeScript) com autenticação, auditoria e integração interoperável.

---

## 🧾 Sumário

* [Nomes das versões](#nomes-das-vers%C3%B5es)
* [Visão geral e objetivos](#vis%C3%A3o-geral-e-objetivos)
* [Escopo do MVP (Core)](#escopo-do-mvp-core)
* [Escopo da Versão Final (Pro)](#escopo-da-vers%C3%A3o-final-pro)
* [Tech stack sugerido](#tech-stack-sugerido)
* [Estrutura do repositório](#estrutura-do-reposit%C3%B3rio)
* [Banco de dados (schema SQLite)](#banco-de-dados-schema-sqlite)
* [API — Endpoints principais](#api--endpoints-principais)
* [Front-end MVP](#front-end-mvp)
* [Como rodar o MVP localmente (guia rápido)](#como-rodar-o-mvp-localmente-guia-r%C3%A1pido)
* [Boas práticas (tests, lint, CI/CD)](#boas-pr%C3%A1ticas-tests-lint-cicd)
* [Segurança e conformidade](#seguran%C3%A7a-e-conformidade)
* [Roadmap / Milestones](#roadmap--milestones)
* [Contribuição](#contribui%C3%A7%C3%A3o)
* [Licença](#licen%C3%A7a)

---

## Nomes das versões

* **Versão inicial (MVP):** `CuraSys — Core`
* **Versão final (Profissional):** `CuraSys — Pro`

> Use esses nomes em branches, tags e releases (ex.: `v0.1-core`, `v1.0-pro`).

---

## Visão geral e objetivos

Criar um sistema de gestão hospitalar simples, seguro e modular que permita:

* Cadastrar e gerenciar **médicos**, **pacientes**, **consultas** e **exames**.
* Gerar listagens e relatórios básicos para administração.
* Possuir autenticação para papéis (admin, recepção, médico).
* Ser modular para escalar: migrar SQLite → PostgreSQL, Flask → FastAPI, frontend simples → React+TypeScript.

### Público-alvo

Pequenas clínicas e hospitais que precisam de um sistema acessível para controle de fluxo clínico e administrativo.

---

## Escopo do MVP (Core)

Funcionalidades mínimas para um produto testável:

* CRUD de **pacientes**.
* CRUD de **médicos**.
* Agendamento de **consultas** (associar médico + paciente + data/hora).
* Registro de **exames** e seus resultados.
* Frontend simples (HTML/CSS/JS) para testes e validação de fluxo.
* API REST básica com Flask e SQLite.
* Autenticação simples (login básico / senha) para acessar endpoints administrativos.

---

## Escopo da Versão Final (Pro)

Funcionalidades avançadas para produção:

* Migração para **PostgreSQL** e uso de ORM (SQLAlchemy / Alembic migrations).
* Substituir Flask por **FastAPI** (tipagem, documentação automática `/docs`).
* Frontend em **React + TypeScript** com design responsivo.
* Autenticação robusta (JWT), roles (admin, recepção, médico), e permissões.
* Auditoria de ações (logs imutáveis), backups automáticos, monitoramento.
* Integração com padrões de saúde (FHIR/HL7) para interoperabilidade.
* Deploy em containers (Docker, docker-compose / k8s) com pipeline CI/CD.
* Relatórios e dashboards analíticos.

---

## Tech stack sugerido

**MVP**

* Backend: Python, Flask, sqlite3, flask-cors
* Frontend: HTML, CSS, Vanilla JS (fetch)
* Tests: pytest

**Pro**

* Backend: Python, FastAPI, SQLAlchemy, PostgreSQL, Alembic, Uvicorn, Gunicorn
* Frontend: React, TypeScript, Vite, Tailwind (ou design system da sua escolha)
* Auth: JWT, OAuth2 (se necessário)
* Infra: Docker, Docker Compose, GitHub Actions, Terraform (opcional)

---

## Estrutura do repositório (sugestão profissional)

```
CuraSys/
├── README.md
├── LICENSE
├── .gitignore
├── docs/                      # documentação e decisões arquiteturais
│   ├── architecture.md
│   └── api-contracts.md
├── backend/
│   ├── requirements.txt
│   ├── pyproject.toml
│   ├── init_db.py             # script para criar o .db e tabelas (MVP)
│   ├── app.py                 # entrypoint (Flask) para MVP
│   ├── config.py
│   ├── /src/
│   │   ├── __init__.py
│   │   ├── db.py              # conexão e helpers
│   │   ├── models.py          # DDL ou mapeamentos leves
│   │   ├── blueprints/        # endpoints por recurso
│   │   │   ├── pacientes.py
│   │   │   ├── medicos.py
│   │   │   ├── consultas.py
│   │   │   └── exames.py
│   │   └── services/          # regras de negócio (opcional)
│   └── tests/
├── frontend/
│   ├── mvp/
│   │   ├── index.html
│   │   ├── pacientes.html
│   │   └── assets/
│   └── pro/                   # esqueleto para React + TS (quando migrar)
│       └── README.md
├── docker/                    # docker-compose e imagens de produção
│   └── docker-compose.yml
└── .github/
    └── workflows/
        └── ci.yml
```

---

## Banco de dados (schema SQLite) — script inicial (MVP)

> Coloque esse SQL em `backend/init_db.py` ou como `.sql` para execução.

```sql
CREATE TABLE medicos (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nome TEXT NOT NULL,
    especialidade TEXT NOT NULL,
    crm TEXT UNIQUE NOT NULL,
    email TEXT,
    telefone TEXT,
    criado_em DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE pacientes (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nome TEXT NOT NULL,
    data_nascimento DATE NOT NULL,
    cpf TEXT UNIQUE NOT NULL,
    telefone TEXT,
    endereco TEXT,
    criado_em DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE consultas (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    id_medico INTEGER NOT NULL,
    id_paciente INTEGER NOT NULL,
    data_consulta DATETIME NOT NULL,
    motivo TEXT,
    status TEXT DEFAULT 'agendada', -- ex: agendada, realizada, cancelada
    criado_em DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (id_medico) REFERENCES medicos(id),
    FOREIGN KEY (id_paciente) REFERENCES pacientes(id)
);

CREATE TABLE exames (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    id_paciente INTEGER NOT NULL,
    tipo TEXT NOT NULL,
    resultado TEXT,
    data_exame DATETIME NOT NULL,
    arquivo_path TEXT, -- caso queira referenciar arquivo salvo
    criado_em DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (id_paciente) REFERENCES pacientes(id)
);

-- Tabela simples de usuários (para autenticação MVP)
CREATE TABLE usuarios (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT UNIQUE NOT NULL,
    senha_hash TEXT NOT NULL,
    role TEXT NOT NULL DEFAULT 'recepcao', -- recepcao, medico, admin
    criado_em DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

---

## API — Endpoints principais (exemplo REST para MVP)

> Use Blueprints (Flask) por recurso; mais tarde transformar em rotas com FastAPI.

### Pacientes

* `GET /pacientes` → lista todos os pacientes
* `POST /pacientes` → cria novo paciente (body JSON)
* `GET /pacientes/:id` → detalha paciente
* `PUT /pacientes/:id` → atualiza paciente
* `DELETE /pacientes/:id` → remove paciente

**Exemplo — POST /pacientes**

```json
{
  "nome": "João Silva",
  "data_nascimento": "1980-05-01",
  "cpf": "123.456.789-00",
  "telefone": "(85) 9 9999-9999"
}
```

### Médicos

* `GET /medicos`
* `POST /medicos`
* `GET /medicos/:id`
* `PUT /medicos/:id`
* `DELETE /medicos/:id`

### Consultas

* `GET /consultas` (filtráveis por médico/paciente/data)
* `POST /consultas` (agenda)
* `PUT /consultas/:id` (alterar / cancelar)
* `DELETE /consultas/:id`

**Exemplo — POST /consultas**

```json
{
  "id_medico": 1,
  "id_paciente": 3,
  "data_consulta": "2025-09-20T10:30:00",
  "motivo": "Revisão"
}
```

### Exames

* `GET /exames?paciente_id=`
* `POST /exames` (registrar resultado)
* `GET /exames/:id`

### Autenticação (MVP)

* `POST /auth/login` → recebe username + senha, retorna token de sessão (simples) ou JWT (recomendado)

> Observação: habilite CORS no Flask para permitir que o frontend local faça requisições: `Flask-CORS`.

---

## Front-end (MVP)

* Páginas HTML simples para testar fluxos: `pacientes.html`, `medicos.html`, `consultas.html`, `exames.html`.
* Comunicação via `fetch` com a API (JSON).
* Durante desenvolvimento local, sirva o frontend com `python -m http.server 8000` ou abra os arquivos direto no navegador (preferível usar um servidor simples para evitar problemas de CORS).

---

## Como rodar o MVP localmente (guia rápido)

**Pré-requisitos**: Python 3.10+, git

```bash
# clonar repositório
git clone <repo-url>
cd CuraSys/backend

# criar virtualenv (Unix/macOS)
python -m venv venv
source venv/bin/activate

# Windows (PowerShell)
# python -m venv venv
# .\venv\Scripts\Activate.ps1

pip install -r requirements.txt

# criar banco e tabelas (script init_db.py)
python init_db.py

# rodar o servidor Flask (MVP)
export FLASK_APP=app.py
export FLASK_ENV=development
flask run --host=127.0.0.1 --port=5000

# abrir frontend em outro terminal (opcional)
cd ../frontend/mvp
python -m http.server 8000
# abrir http://127.0.0.1:8000
```

> Dica: se receber erro `CORS`, instale e configure `flask-cors` no `app.py`:

```python
from flask_cors import CORS
app = Flask(__name__)
CORS(app)
```

---

## Boas práticas (tests, lint, CI/CD)

* **Lint/format**: black, isort, flake8 para backend; Prettier + ESLint para frontend.
* **Tests**: pytest para backend, testes unitários e de integração (simular banco em memória `sqlite://:memory:`).
* **CI**: GitHub Actions com workflows para `lint`, `test` e `build`.
* **Branches**: usar `main` (produção), `develop` (integração), feature branches `feat/<descrição>`.
* **PRs**: templates de PR com checklist (tests, lint, documentação atualizada).

---

## Segurança e conformidade

* **Hash de senhas**: nunca guardar senha em texto. Use `bcrypt`/`passlib`.
* **TLS/HTTPS**: obrigatório em produção.
* **Backups**: rotina automática de dump do banco (Postgres) e retenção.
* **Logs e auditoria**: gravar quem fez alterações sensíveis.
* **LGPD (Brasil)**: cuidado ao armazenar dados pessoais (CPF, saúde). Minimize dados, permita exportação/exclusão mediante solicitação.

---

## Roadmap / Milestones (priorizados)

1. `setup-repo` — scaffold do repositório + README + scripts básicos
2. `db-models` — criar schema e script `init_db.py`
3. `backend-core` — endpoints CRUD pacientes/medicos/consultas/exames
4. `frontend-mvp` — páginas para testar flows
5. `auth-basic` — login simples e roles
6. `tests` — testes unitários para endpoints críticos
7. `ci` — GitHub Actions: lint/test
8. `dockerize` — Dockerfile + docker-compose para ambiente de testes
9. `migrate-to-pro` — migrar para Postgres + FastAPI + React+TS
10. `pro-features` — auditoria, backups, FHIR

> Mantenha essa seção atualizada com milestones fechados (usar Projects/Milestones do GitHub).

---

## Contribuição

1. Fork e crie uma branch `feat/<seu-issue>`
2. Faça commits pequenos e claros: `feat(pacientes): adicionar endpoint POST /pacientes`
3. Abra um Pull Request para `develop` com descrição e testes
4. Use template de PR e aguarde review

**Labels sugeridos**: `bug`, `enhancement`, `help wanted`, `good first issue`, `docs`.

---

## Licença

Recomendo **MIT License** para começar. Crie arquivo `LICENSE` com o template MIT.

---

## Contato / mantenedores

* **Nome:** \Kelvyn Leôncio Andrade Lima
* **E-mail:** seu\kermerlima10@gmail.com
* **GitHub:** @Kelvinl14

---

## Evolução para versão final (LEMBRETE / checklist)

Este bloco serve como "lista de tarefas" para quando o MVP estiver estável:

* Migrar banco para **PostgreSQL**.
* Substituir Flask por **FastAPI** com tipagem e docs automáticos.
* Implementar **ORM** robusto (SQLAlchemy) e **migrations** (Alembic).
* Autenticação com **JWT** + refresh tokens, RBAC (roles/permissions).
* Dashboard em **React + TypeScript** com testes end-to-end.
* Adicionar **auditoria** de eventos (quem mudou o quê e quando).
* Implementar **backups**, monitoramento (Prometheus/Grafana) e alertas.
* Criar **pipeline CI/CD** (builds, testes, imagem Docker, deploy automático em staging).
* Avaliar **interoperabilidade** (FHIR) e integrações com laboratórios/ERPs.
* Revisar **LGPD/HIPAA** conforme mercado-alvo.

---

