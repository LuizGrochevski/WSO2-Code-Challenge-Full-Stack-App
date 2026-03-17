# WSO2 Code Challenge — Full Stack Notes App

Aplicação full stack de estudo com **Django + Django REST Framework** no backend e **React + Vite** no frontend, focada em autenticação JWT e operações básicas de notas.

> Este repositório representa um **MVP em desenvolvimento**. O objetivo atual é servir como base técnica para prática de arquitetura web, debugging e melhorias de segurança.

---

## Descrição breve

Este projeto implementa uma API para cadastro de usuários, geração de tokens JWT e gerenciamento de notas por usuário autenticado. No frontend, há a base de autenticação/consumo da API e estrutura inicial de páginas.

---

## Visão geral (fluxo da aplicação)

1. Usuário registra conta via endpoint de cadastro.
2. Usuário autentica via endpoint JWT e recebe `access` + `refresh`.
3. Frontend armazena tokens em `localStorage`.
4. Requisições autenticadas enviam token no header `Authorization`.
5. API retorna/cria notas vinculadas ao usuário autenticado.
6. Quando o token expira, o frontend tenta renovar via `refresh`.

---

## Stack

### Backend
- Python 3.10+
- Django 5
- Django REST Framework
- SimpleJWT (`djangorestframework-simplejwt`)
- `django-cors-headers`
- SQLite (banco padrão no projeto)

### Frontend
- React 18
- Vite 5
- Axios
- `react-router-dom`
- `jwt-decode`

---

## Como rodar

## Pré-requisitos
- Python 3.10+
- Node.js 18+ (ou superior)
- npm

### 1) Backend (Django + DRF)

```bash
cd backend
python -m venv .venv
source .venv/bin/activate
pip install -r requirement.txt
python manage.py migrate
python manage.py runserver
```

Backend padrão: `http://127.0.0.1:8000`

### 2) Frontend (React + Vite)

```bash
cd frontend
npm install
```

Crie um arquivo `.env` em `frontend/`:

```env
VITE_API_URL=http://127.0.0.1:8000
```

Inicie:

```bash
npm run dev
```

Frontend padrão: `http://127.0.0.1:5173`

---

## Arquitetura

```text
.
├── backend/
│   ├── backend/           # settings, urls, wsgi/asgi
│   ├── api/               # models, serializers, views, rotas da API
│   ├── manage.py
│   └── db.sqlite3
└── frontend/
    ├── src/
    │   ├── pages/         # Home, Login, Register (estado inicial)
    │   ├── components/    # ProtectedRoute
    │   ├── api.js         # cliente axios/interceptor
    │   └── constants.js   # chaves de token
    └── vite config + scripts
```

### Fluxo de dados

- **Frontend** envia requisições via `axios` para `VITE_API_URL`.
- **Backend DRF** aplica autenticação JWT por padrão nos endpoints protegidos.
- **Model `Note`** persiste notas com relação ao `User` autenticado.
- Endpoints de token (`/api/token/` e `/api/token/refresh/`) controlam sessão stateless.

---

## Endpoints principais

| Método | Rota | Auth | Descrição |
|---|---|---|---|
| POST | `/api/user/register/` | Não | Cadastra usuário |
| POST | `/api/token/` | Não | Gera token JWT (`access`, `refresh`) |
| POST | `/api/token/refresh/` | Não | Renova `access` com `refresh` |
| GET | `/api/notes/` | Sim | Lista notas do usuário autenticado |
| POST | `/api/notes/` | Sim | Cria nota para usuário autenticado |
| DELETE | `/api/notes/delete/<id>/` | Sim | Remove nota por ID |

### Exemplos de payload

#### 1) Registro

`POST /api/user/register/`

```json
{
  "username": "alice",
  "password": "senha123"
}
```

#### 2) Login JWT

`POST /api/token/`

```json
{
  "username": "alice",
  "password": "senha123"
}
```

Resposta esperada:

```json
{
  "refresh": "<jwt_refresh>",
  "access": "<jwt_access>"
}
```

#### 3) Criar nota

`POST /api/notes/`

Header esperado:

```http
Authorization: Bearer <jwt_access>
```

Body:

```json
{
  "title": "Minha nota",
  "content": "Conteúdo da nota"
}
```

---

## Estado atual do projeto

- **Status:** MVP inicial/incompleto.
- Backend possui estrutura funcional base de autenticação e notas.
- Frontend possui infraestrutura inicial (API client, proteção de rota), porém páginas de produto ainda estão em estado de placeholder.
- Não há suíte de testes automatizados implementada para cobrir os fluxos críticos.

---

## Problemas conhecidos (bugs reais)

1. Header de autenticação no frontend está com typo (`Beare` em vez de `Bearer`), comprometendo chamadas autenticadas.
2. Cálculo de expiração de token no `ProtectedRoute` usa `Date.now` sem invocação (`Date.now()`), afetando a validação de sessão.
3. `NoteDelete` não define queryset corretamente e não restringe exclusão por autor no método sobrescrito.
4. `__str__` de `Note` está incorreto (`super.title`), causando comportamento inesperado na representação do model.
5. Chave de configuração CORS para credenciais está nomeada incorretamente (`CORS_ALLOWS_CREDENTIALS`).

---

## Considerações de segurança (baseado no código atual)

- `SECRET_KEY` está hardcoded no código-fonte.
- `DEBUG=True` em settings.
- `ALLOWED_HOSTS` vazio (configuração de desenvolvimento).
- `CORS_ALLOW_ALL_ORIGINS=True` habilitado globalmente.
- Tokens JWT são persistidos em `localStorage` (maior superfície para XSS em comparação com cookies `HttpOnly`).

> Recomendação: mover segredos para variáveis de ambiente e separar settings por ambiente (`dev`, `staging`, `prod`).

---

## Melhorias futuras (curto prazo)

1. Corrigir bugs críticos de autenticação e autorização já identificados.
2. Implementar telas reais de Login/Register/Home com formulários e feedback de erro.
3. Criar testes mínimos:
   - Backend: testes de endpoint e permissão por usuário.
   - Frontend: testes de fluxo de autenticação e proteção de rotas.
4. Aplicar baseline de AppSec:
   - variáveis de ambiente para segredos,
   - configuração restritiva de CORS,
   - revisão de storage de token.
5. Padronizar documentação de setup com versões fixas e troubleshooting.

---

## Licença

Sem licença definida atualmente no repositório.
