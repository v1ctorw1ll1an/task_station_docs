# Plano de Implementação — MVP
**Sistema de Gestão de Projetos e Tasks**
Stack: Next.js · NestJS · PostgreSQL · Prisma · JWT

---

## Filosofia de entrega

Cada fase entrega algo **funcional e testável de ponta a ponta** — do banco ao frontend. Nenhuma fase deixa camadas incompletas. A ordem respeita dependências: você não pode construir workspaces sem ter empresas, nem tasks sem ter Kanban.

---

## Fase 0 — Setup e Infraestrutura
> **Objetivo:** Projeto rodando localmente do zero ao "Hello World" nas duas pontas.

- [x] Repositório com backend (NestJS) criado
- [x] NestJS configurado com TypeScript, Prisma, variáveis de ambiente
- [x] PostgreSQL local via Docker Compose
- [x] Prisma schema completo com todas as tabelas e migration inicial rodando
- [x] Rota de healthcheck na API (`GET /api/v1/health`) — valida conexão com banco
- [x] ESLint + Prettier configurados
- [ ] Next.js configurado com TypeScript e estrutura de pastas base
- [ ] Commit hooks configurados (Husky)

**Entregável:** `docker compose up` sobe banco + api + web sem erros.

---

## Fase 1 — Autenticação
> **Objetivo:** Qualquer usuário consegue fazer login, ser redirecionado e sair.
> **RFs cobertos:** RF001, RF002, RF003

### Backend
- [x] Endpoint `POST /auth/login` — valida email/senha, retorna JWT
- [x] Guard JWT global — protege todas as rotas autenticadas via `APP_GUARD`
- [x] Verificação de `is_active` e `deleted_at` no `validateUser` da local strategy
- [x] Endpoint `POST /auth/reset-password` — redefine senha e atualiza `must_reset_password`
- [x] Endpoint `POST /auth/logout` — sem estado no servidor (JWT), apenas resposta 200
- [x] Seed do superusuário inicial (`pnpm seed`)
- [x] Swagger UI em `GET /api/docs` com suporte a Bearer token

### Frontend
- [ ] Página de login (`/login`)
- [ ] Página de redefinição de senha obrigatória (`/first-access`)
- [ ] Lógica de armazenamento do token (httpOnly cookie ou memory)
- [ ] Redirecionamento pós-login conforme role do usuário
- [ ] Botão e fluxo de logout

**Entregável:** Superusuário consegue logar, é redirecionado ao painel e consegue sair.

---

## Fase 2 — Recuperação de Senha
> **Objetivo:** Usuário que esqueceu a senha consegue redefini-la via email.
> **RFs cobertos:** RF004

### Backend
- [ ] Endpoint `POST /auth/forgot-password` — gera token, salva hash, envia email
- [ ] Endpoint `POST /auth/reset-password/:token` — valida token, redefine senha, marca `used_at`
- [ ] Integração com serviço de email (Resend, Nodemailer ou similar)
- [ ] Expiração e uso único do token garantidos no banco

### Frontend
- [ ] Página "Esqueci minha senha" (`/forgot-password`)
- [ ] Página de redefinição via link (`/reset-password/:token`)

**Entregável:** Fluxo completo de recuperação de senha funcionando via email.

---

## Fase 3 — Painel do Superusuário + Gestão de Empresas
> **Objetivo:** Superusuário consegue criar, listar, editar, inativar e deletar empresas e seus admins.
> **RFs cobertos:** RF005, RF006, RF007, RF008, RF009, RF010, RF011

### Backend
- [ ] Guard de superusuário (`is_superuser = true`)
- [ ] `POST /superadmin/empresas` — cria empresa + admin em transação atômica, envia email
- [ ] `GET /superadmin/empresas` — listagem com filtros de status e busca
- [ ] `PATCH /superadmin/empresas/:id` — edição de razão social, CNPJ, status
- [ ] `PATCH /superadmin/empresas/:id/inativar` — seta `is_active = false`
- [ ] `DELETE /superadmin/empresas/:id` — soft delete com `deleted_at`
- [ ] `GET /superadmin/usuarios` — listagem de todos os usuários
- [ ] `PATCH /superadmin/usuarios/:id` — inativar, reativar, soft delete

### Frontend
- [ ] Layout do painel do superusuário (sidebar + área de conteúdo)
- [ ] Listagem de empresas com filtros
- [ ] Formulário de criação de empresa (empresa + admin no mesmo form)
- [ ] Formulário de edição
- [ ] Modal de confirmação para inativação e soft delete
- [ ] Listagem de usuários com ações

**Entregável:** Superusuário opera o ciclo completo de empresas pelo painel.

---

## Fase 4 — Painel da Empresa + Gestão de Workspaces
> **Objetivo:** Admin de empresa cria e gerencia workspaces e seus admins.
> **RFs cobertos:** RF012, RF013, RF014, RF015, RF016, RF017, RF018, RF019

### Backend
- [ ] Guard de empresa admin — verifica membership `resource_type=empresa, role=admin`
- [ ] `POST /empresa/workspaces` — cria workspace + admin em transação atômica
- [ ] `GET /empresa/workspaces` — listagem dos workspaces da empresa autenticada
- [ ] `PATCH /empresa/workspaces/:id` — edição de nome, descrição, status
- [ ] `PATCH /empresa/workspaces/:id/inativar`
- [ ] `DELETE /empresa/workspaces/:id` — soft delete
- [ ] `GET /empresa/membros` — lista membros da empresa
- [ ] `PATCH /empresa/membros/:id` — inativar, reativar, soft delete
- [ ] `POST /empresa/admins` — promove membro a admin da empresa
- [ ] `DELETE /empresa/admins/:id` — revoga papel de admin

### Frontend
- [ ] Layout do painel da empresa
- [ ] Dashboard com contagem de workspaces e membros
- [ ] Listagem de workspaces com filtros
- [ ] Formulário de criação de workspace (workspace + admin)
- [ ] Formulário de edição de workspace
- [ ] Listagem de membros com ações de gestão

**Entregável:** Admin de empresa opera o ciclo completo de workspaces pelo painel.

---

## Fase 5 — Painel do Workspace + Gestão de Projetos e Membros
> **Objetivo:** Admin de workspace cria projetos e gerencia membros do workspace.
> **RFs cobertos:** RF020, RF021, RF022, RF023, RF024, RF025

### Backend
- [ ] Guard de workspace admin
- [ ] `POST /workspace/:id/projetos` — cria projeto + colunas Kanban padrão automaticamente
- [ ] `GET /workspace/:id/projetos` — listagem de projetos do workspace
- [ ] `GET /workspace/:id/membros` — lista membros
- [ ] `POST /workspace/:id/membros` — adiciona membro (deve ser da empresa)
- [ ] `DELETE /workspace/:id/membros/:userId` — soft delete do membership
- [ ] `POST /workspace/:id/admins` — promove membro a workspace admin
- [ ] `DELETE /workspace/:id/admins/:userId` — revoga papel

### Frontend
- [ ] Layout do painel do workspace
- [ ] Dashboard com projetos ativos, membros e tasks com prazo próximo
- [ ] Listagem de projetos
- [ ] Formulário de criação de projeto
- [ ] Gestão de membros do workspace

**Entregável:** Admin de workspace cria projetos e gerencia seu time.

---

## Fase 6 — Kanban e Tasks
> **Objetivo:** Membros visualizam e operam o quadro Kanban com todas as funcionalidades de task.
> **RFs cobertos:** RF026, RF027, RF028, RF029, RF030, RF031, RF032, RF033, RF034, RF035, RF036, RF037, RF038, RF039

### Backend
- [ ] `GET /projetos/:id/kanban` — retorna colunas + tasks ordenadas
- [ ] `POST /projetos/:id/colunas` — cria nova coluna
- [ ] `PATCH /projetos/:id/colunas/:colId` — edita nome/cor
- [ ] `PATCH /projetos/:id/colunas/reorder` — reordena colunas (atualiza `ordem`)
- [ ] `DELETE /projetos/:id/colunas/:colId` — soft delete (exige migração de tasks)
- [ ] `POST /projetos/:id/tasks` — cria task com todos os campos
- [ ] `PATCH /projetos/:id/tasks/:taskId` — edição de task
- [ ] `PATCH /projetos/:id/tasks/:taskId/move` — move task de coluna ou reordena
- [ ] `DELETE /projetos/:id/tasks/:taskId` — soft delete de task
- [ ] `PATCH /projetos/:id/tasks/:taskId/assign` — atribui ou remove assigned
- [ ] Lógica de rebalanceamento de `ordem` em background
- [ ] `POST /projetos/:id/colaboradores` — adiciona membro externo (RF031)

### Frontend
- [ ] Quadro Kanban com colunas e cards
- [ ] Drag-and-drop de tasks entre colunas e dentro da mesma coluna
- [ ] Drag-and-drop de reordenação de colunas
- [ ] Modal/drawer de detalhes da task com edição inline
- [ ] Formulário de criação de task
- [ ] Indicadores visuais de prioridade e prazo expirado
- [ ] Confirmação de exclusão de coluna com seleção de destino das tasks

**Entregável:** Quadro Kanban completo e operacional.

---

## Fase 7 — Perfil e Permissões Finais
> **Objetivo:** Usuário edita seu perfil. Sistema de permissões revisado e robusto.
> **RFs cobertos:** RF040, RF041, RF042, RF043, RF044

### Backend
- [ ] `GET /perfil` — retorna dados do usuário autenticado
- [ ] `PATCH /perfil` — edita nome, telefone, foto
- [ ] `PATCH /perfil/senha` — altera senha com confirmação da atual
- [ ] Revisão completa de todos os guards e regras de membership
- [ ] Verificação de `is_active` da empresa pai ao autenticar membros
- [ ] Garantia de que `created_at`, `updated_at`, `created_by` são preenchidos automaticamente via Prisma middleware

### Frontend
- [ ] Página de perfil do usuário
- [ ] Formulário de edição de dados pessoais
- [ ] Formulário de alteração de senha

**Entregável:** Sistema completo, revisado e pronto para testes de aceitação.

---

## Resumo das Fases

| Fase | Nome | RFs | Dependências |
|---|---|---|---|
| 0 | Setup e Infraestrutura | — | Nenhuma |
| 1 | Autenticação | RF001, RF002, RF003 | Fase 0 |
| 2 | Recuperação de Senha | RF004 | Fase 1 |
| 3 | Superusuário + Empresas | RF005–RF011 | Fase 1 |
| 4 | Empresa + Workspaces | RF012–RF019 | Fase 3 |
| 5 | Workspace + Projetos | RF020–RF025 | Fase 4 |
| 6 | Kanban + Tasks | RF026–RF039 | Fase 5 |
| 7 | Perfil + Permissões finais | RF040–RF044 | Fase 6 |

---

## Regras para cada entrega

1. Nenhuma fase avança sem a anterior estar funcionando de ponta a ponta.
2. Todo endpoint novo tem validação de entrada (DTOs com class-validator no NestJS).
3. Todo endpoint novo tem guard de autenticação e autorização corretos.
4. Soft delete é garantido — nenhuma query lista registros com `deleted_at` preenchido sem filtro explícito.
5. Erros retornam mensagens claras e padronizadas — nunca expõem stack trace em produção.
