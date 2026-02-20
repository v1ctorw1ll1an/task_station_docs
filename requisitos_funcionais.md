# Especificação de Requisitos Funcionais
**Sistema de Gestão de Projetos e Tasks**
Versão 1.0 — MVP | 20/02/2026
Total de requisitos: 44

---

## Módulo: Autenticação

### RF001 — Login de Usuário `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Autenticação |
| **Ator** | Todos os usuários |
| **Descrição** | O sistema deve permitir que qualquer usuário autenticado acesse a plataforma através de email e senha cadastrados. |

**Regras de Negócio:**
1. O email deve ser único no sistema.
2. A senha deve ser validada contra o hash armazenado no banco de dados.
3. Em caso de credenciais inválidas, o sistema deve exibir mensagem de erro genérica sem revelar qual campo está incorreto.
4. Após autenticação bem-sucedida, o sistema deve gerar um token JWT.
5. O sistema deve redirecionar o usuário para sua tela inicial conforme seu papel (role).

---

### RF002 — Redefinição de Senha Obrigatória no Primeiro Acesso `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Autenticação |
| **Ator** | Usuários com `must_reset_password = true` |
| **Descrição** | Usuários criados pelo superusuário ou por admins recebem uma senha temporária e são forçados a redefini-la no primeiro login. |

**Regras de Negócio:**
1. Ao fazer login com `must_reset_password = true`, o sistema deve redirecionar obrigatoriamente para a tela de redefinição de senha.
2. O usuário não pode acessar nenhuma outra parte do sistema antes de redefinir a senha.
3. A nova senha deve atender aos critérios mínimos de segurança definidos pelo sistema.
4. Após redefinição bem-sucedida, o campo `must_reset_password` deve ser atualizado para `false`.
5. O sistema deve redirecionar o usuário para sua tela inicial após a redefinição.

---

### RF003 — Logout de Usuário `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Autenticação |
| **Ator** | Todos os usuários autenticados |
| **Descrição** | O sistema deve permitir que o usuário encerre sua sessão de forma segura. |

**Regras de Negócio:**
1. O token JWT deve ser descartado no cliente.
2. O usuário deve ser redirecionado para a tela de login.
3. Dados sensíveis em memória devem ser limpos.

---

### RF004 — Recuperação de Senha via Email `● Média`

| Campo | Detalhe |
|---|---|
| **Módulo** | Autenticação |
| **Ator** | Todos os usuários |
| **Descrição** | O sistema deve permitir que o usuário solicite a recuperação de senha através do email cadastrado. |

**Regras de Negócio:**
1. O sistema deve enviar um email com link de redefinição de senha.
2. O link deve ter validade de tempo limitado (ex: 2 horas).
3. O link deve ser de uso único e invalidado após utilização.
4. Se o email não existir no sistema, a resposta deve ser idêntica à do email válido (segurança contra enumeração).

---

## Módulo: Superusuário

### RF005 — Acesso Total ao Sistema pelo Superusuário `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Superusuário |
| **Ator** | Superusuário |
| **Descrição** | O superusuário deve ter acesso irrestrito a todos os dados e funcionalidades da plataforma. |

**Regras de Negócio:**
1. O superusuário é identificado pelo campo `is_superuser = true` na tabela `users`.
2. Nenhuma regra de escopo ou membership se aplica ao superusuário.
3. O superusuário pode visualizar, criar, editar, inativar e excluir (soft delete) qualquer entidade do sistema.
4. O superusuário tem acesso a um painel administrativo exclusivo.

---

### RF006 — Criação de Empresa pelo Superusuário `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Superusuário |
| **Ator** | Superusuário |
| **Descrição** | O superusuário deve criar empresas no sistema, provisionando simultaneamente os dados da empresa e do seu primeiro administrador. |

**Regras de Negócio:**
1. O formulário de criação deve conter: razão social, CNPJ, e dados do admin (nome, email, telefone).
2. O sistema deve criar a empresa e o usuário admin em uma única transação atômica.
3. O email do admin deve ser único no sistema; caso já exista, a operação deve ser rejeitada com mensagem clara.
4. Após criação, o sistema deve gerar uma senha temporária para o admin e enviá-la por email.
5. O campo `must_reset_password` do admin criado deve ser definido como `true`.
6. O sistema deve criar automaticamente um registro em `user_memberships` vinculando o admin à empresa com `role = 'admin'`.
7. O campo `created_by` da empresa deve registrar o id do superusuário.

---

### RF007 — Listagem de Empresas pelo Superusuário `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Superusuário |
| **Ator** | Superusuário |
| **Descrição** | O superusuário deve visualizar todas as empresas cadastradas no sistema. |

**Regras de Negócio:**
1. A listagem deve exibir razão social, CNPJ, status (ativo/inativo) e data de criação.
2. Deve ser possível filtrar por status e buscar por nome/CNPJ.
3. Registros com soft delete (`deleted_at` preenchido) não devem aparecer na listagem padrão.
4. O superusuário deve poder visualizar registros deletados através de filtro específico.

---

### RF008 — Edição de Empresa pelo Superusuário `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Superusuário |
| **Ator** | Superusuário |
| **Descrição** | O superusuário deve poder editar os dados de qualquer empresa. |

**Regras de Negócio:**
1. Campos editáveis: razão social, CNPJ, status.
2. Alterações devem atualizar o campo `updated_at` automaticamente.
3. O CNPJ deve ser validado quanto ao formato e unicidade.

---

### RF009 — Inativação de Empresa pelo Superusuário `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Superusuário |
| **Ator** | Superusuário |
| **Descrição** | O superusuário deve poder inativar uma empresa, suspendendo o acesso de todos os seus membros. |

**Regras de Negócio:**
1. Ao inativar uma empresa, o campo `is_active` deve ser definido como `false`.
2. Todos os usuários vinculados à empresa devem ter o acesso bloqueado enquanto a empresa estiver inativa.
3. Os dados da empresa e seus vínculos não devem ser apagados.
4. A empresa pode ser reativada pelo superusuário a qualquer momento.

---

### RF010 — Exclusão Lógica de Empresa pelo Superusuário `● Média`

| Campo | Detalhe |
|---|---|
| **Módulo** | Superusuário |
| **Ator** | Superusuário |
| **Descrição** | O superusuário deve poder realizar a exclusão lógica (soft delete) de uma empresa. |

**Regras de Negócio:**
1. A exclusão lógica deve preencher o campo `deleted_at` com a data/hora atual.
2. Empresas com `deleted_at` preenchido não devem aparecer em listagens padrão.
3. Todos os workspaces, projetos e tasks vinculados devem ser inacessíveis após o soft delete da empresa.
4. O sistema deve exigir confirmação antes de executar a operação.

---

### RF011 — Listagem e Gerenciamento de Usuários pelo Superusuário `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Superusuário |
| **Ator** | Superusuário |
| **Descrição** | O superusuário deve poder visualizar e gerenciar todos os usuários da plataforma. |

**Regras de Negócio:**
1. A listagem deve exibir nome, email, status e empresa(s) vinculada(s).
2. O superusuário pode inativar ou reativar qualquer conta de usuário.
3. O superusuário pode realizar soft delete de qualquer usuário.
4. O superusuário não pode excluir sua própria conta.

---

## Módulo: Empresa

### RF012 — Visualização do Painel da Empresa pelo Admin `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Empresa |
| **Ator** | Admin de Empresa |
| **Descrição** | O admin de empresa deve ter acesso a um painel com visão geral de seus workspaces, membros e atividades recentes. |

**Regras de Negócio:**
1. O painel deve exibir apenas dados pertencentes à empresa do admin autenticado.
2. Deve listar todos os workspaces ativos da empresa.
3. Deve exibir contagem de membros ativos.
4. Deve exibir atividades recentes (tasks criadas, atualizadas).

---

### RF013 — Criação de Workspace pelo Admin de Empresa `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Empresa |
| **Ator** | Admin de Empresa |
| **Descrição** | O admin de empresa deve poder criar workspaces, provisionando simultaneamente o workspace e seu primeiro administrador. |

**Regras de Negócio:**
1. O formulário deve conter: nome do workspace, descrição e dados do admin do workspace (nome, email, telefone).
2. O sistema deve criar o workspace e o usuário admin em uma única transação atômica.
3. Caso o email do admin já exista no sistema, o sistema deve rejeitar com mensagem clara.
4. Após criação, o sistema deve gerar uma senha temporária para o admin e enviá-la por email.
5. O campo `must_reset_password` do admin criado deve ser `true`.
6. O sistema deve criar automaticamente um registro em `user_memberships` vinculando o admin ao workspace com `role = 'workspace_admin'`.
7. O campo `created_by` do workspace deve registrar o id do admin da empresa.
8. O workspace deve ser vinculado à empresa do admin autenticado.

---

### RF014 — Listagem de Workspaces pelo Admin de Empresa `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Empresa |
| **Ator** | Admin de Empresa |
| **Descrição** | O admin de empresa deve visualizar todos os workspaces de sua empresa. |

**Regras de Negócio:**
1. A listagem deve exibir nome, descrição, status e data de criação.
2. Deve ser possível filtrar por status (ativo/inativo).
3. Registros com soft delete não devem aparecer na listagem padrão.

---

### RF015 — Edição de Workspace pelo Admin de Empresa `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Empresa |
| **Ator** | Admin de Empresa |
| **Descrição** | O admin de empresa deve poder editar os dados de qualquer workspace de sua empresa. |

**Regras de Negócio:**
1. Campos editáveis: nome, descrição, status.
2. Alterações devem atualizar o campo `updated_at` automaticamente.
3. O admin de empresa só pode editar workspaces pertencentes à sua empresa.

---

### RF016 — Inativação de Workspace pelo Admin de Empresa `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Empresa |
| **Ator** | Admin de Empresa |
| **Descrição** | O admin de empresa deve poder inativar um workspace. |

**Regras de Negócio:**
1. Ao inativar, o campo `is_active` deve ser `false`.
2. Membros do workspace não devem conseguir acessá-lo enquanto inativo.
3. O workspace pode ser reativado a qualquer momento pelo admin de empresa.

---

### RF017 — Exclusão Lógica de Workspace pelo Admin de Empresa `● Média`

| Campo | Detalhe |
|---|---|
| **Módulo** | Empresa |
| **Ator** | Admin de Empresa |
| **Descrição** | O admin de empresa deve poder realizar soft delete de um workspace. |

**Regras de Negócio:**
1. O campo `deleted_at` deve ser preenchido com a data/hora atual.
2. Todos os projetos e tasks vinculados devem se tornar inacessíveis.
3. O sistema deve exigir confirmação antes de executar.

---

### RF018 — Gerenciamento de Membros da Empresa `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Empresa |
| **Ator** | Admin de Empresa |
| **Descrição** | O admin de empresa deve gerenciar os membros cadastrados em sua empresa. |

**Regras de Negócio:**
1. O admin pode visualizar todos os usuários vinculados à sua empresa via `user_memberships`.
2. O admin pode inativar a conta de um membro (`is_active = false`).
3. O admin pode reativar a conta de um membro.
4. O admin pode realizar soft delete de um membro da empresa.
5. O admin não pode gerenciar contas de usuários de outras empresas.
6. O admin não pode excluir sua própria conta.
7. Ao inativar ou deletar um membro, seus vínculos em `user_memberships` devem ser mantidos para auditoria.

---

### RF019 — Adição de Admin Adicional à Empresa `● Média`

| Campo | Detalhe |
|---|---|
| **Módulo** | Empresa |
| **Ator** | Admin de Empresa |
| **Descrição** | O admin de empresa deve poder promover um membro existente a admin da empresa. |

**Regras de Negócio:**
1. Uma empresa pode ter múltiplos admins.
2. A promoção cria um novo registro em `user_memberships` com `resource_type = 'empresa'`, `role = 'admin'`.
3. O admin pode revogar o papel de admin de outro admin (mas não o próprio).
4. Deve haver sempre pelo menos um admin ativo na empresa.

---

## Módulo: Workspace

### RF020 — Visualização do Painel do Workspace pelo Admin `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Workspace |
| **Ator** | Admin de Workspace |
| **Descrição** | O admin de workspace deve ter acesso a um painel com visão geral de seus projetos e membros. |

**Regras de Negócio:**
1. O painel deve exibir apenas dados do workspace do admin autenticado.
2. Deve listar todos os projetos ativos do workspace.
3. Deve exibir contagem de membros do workspace.
4. Deve exibir tasks recentes e com prazo próximo.

---

### RF021 — Criação de Projeto pelo Admin de Workspace `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Workspace |
| **Ator** | Admin de Workspace, Admin de Empresa |
| **Descrição** | O admin de workspace deve poder criar projetos dentro do workspace. |

**Regras de Negócio:**
1. O formulário deve conter: nome e descrição do projeto.
2. O projeto deve ser vinculado automaticamente ao workspace do admin autenticado.
3. O sistema deve criar automaticamente um conjunto de colunas Kanban padrão para o projeto.
4. O campo `created_by` deve registrar o id do usuário criador.
5. O admin de empresa pode criar projetos em qualquer workspace de sua empresa.

---

### RF022 — Adição de Membros ao Workspace `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Workspace |
| **Ator** | Admin de Workspace, Admin de Empresa |
| **Descrição** | O admin de workspace deve poder adicionar membros ao workspace. |

**Regras de Negócio:**
1. Somente usuários já cadastrados na empresa podem ser adicionados ao workspace.
2. A adição cria um registro em `user_memberships` com `resource_type = 'workspace'`.
3. O role padrão ao adicionar um membro é `'membro'`.
4. O admin pode definir o role no momento da adição.
5. Um usuário pode pertencer a múltiplos workspaces da mesma empresa com roles diferentes.

---

### RF023 — Remoção de Membros do Workspace `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Workspace |
| **Ator** | Admin de Workspace, Admin de Empresa |
| **Descrição** | O admin de workspace deve poder remover membros do workspace sem excluir a conta do usuário. |

**Regras de Negócio:**
1. A remoção realiza soft delete no registro de `user_memberships` correspondente.
2. O usuário removido perde acesso ao workspace mas sua conta permanece ativa no sistema.
3. O admin de workspace não pode remover outros admins de workspace (apenas o admin de empresa pode).
4. O admin de workspace não pode se auto-remover.

---

### RF024 — Adição de Admin Adicional ao Workspace `● Média`

| Campo | Detalhe |
|---|---|
| **Módulo** | Workspace |
| **Ator** | Admin de Workspace, Admin de Empresa |
| **Descrição** | O admin de workspace deve poder promover um membro a admin do workspace. |

**Regras de Negócio:**
1. Um workspace pode ter múltiplos admins.
2. A promoção cria ou atualiza o registro em `user_memberships` com `role = 'workspace_admin'`.
3. O admin pode revogar o papel de admin de workspace de outro admin.
4. Deve haver sempre pelo menos um admin ativo no workspace.

---

### RF025 — Listagem de Projetos do Workspace `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Workspace |
| **Ator** | Admin de Workspace, Membros do Workspace |
| **Descrição** | Os membros do workspace devem visualizar todos os projetos ativos do workspace. |

**Regras de Negócio:**
1. A listagem deve exibir nome, descrição, status e data de criação.
2. Projetos com `deleted_at` preenchido não devem aparecer.
3. Projetos inativos só devem ser visíveis para admins.

---

## Módulo: Projeto

### RF026 — Visualização do Projeto e Kanban `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Projeto |
| **Ator** | Membros do Workspace com acesso ao projeto |
| **Descrição** | Os membros devem visualizar o projeto em formato Kanban com suas colunas e tasks. |

**Regras de Negócio:**
1. O Kanban deve exibir todas as colunas do projeto na ordem definida pelo campo `ordem`.
2. Cada coluna deve exibir suas tasks na ordem definida pelo campo `ordem` das tasks.
3. Tasks com `deleted_at` preenchido não devem aparecer.
4. O usuário deve visualizar apenas projetos de workspaces aos quais pertence.

---

### RF027 — Gerenciamento de Colunas Kanban `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Projeto |
| **Ator** | Admin de Workspace, Admin de Empresa |
| **Descrição** | O admin deve poder criar, editar, reordenar e excluir colunas do Kanban de um projeto. |

**Regras de Negócio:**
1. O sistema deve criar colunas padrão ao criar um projeto: `A Fazer` (1000), `Em Andamento` (2000), `Concluído` (3000).
2. O admin pode adicionar novas colunas informando nome, ordem e cor.
3. O admin pode editar nome e cor de colunas existentes.
4. O admin pode reordenar colunas via drag-and-drop; o campo `ordem` deve ser atualizado.
5. Ao excluir uma coluna que contém tasks, o sistema deve exigir que o usuário escolha para qual coluna mover as tasks antes de confirmar.
6. A exclusão de coluna vazia realiza soft delete (`deleted_at`).
7. Deve haver sempre pelo menos uma coluna ativa no projeto.

---

### RF028 — Edição de Projeto `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Projeto |
| **Ator** | Admin de Workspace, Admin de Empresa |
| **Descrição** | O admin deve poder editar os dados de um projeto. |

**Regras de Negócio:**
1. Campos editáveis: nome, descrição, status.
2. Alterações devem atualizar o campo `updated_at`.
3. O admin de workspace só pode editar projetos do seu workspace.

---

### RF029 — Inativação de Projeto `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Projeto |
| **Ator** | Admin de Workspace, Admin de Empresa |
| **Descrição** | O admin deve poder inativar um projeto. |

**Regras de Negócio:**
1. Ao inativar, `is_active` deve ser `false`.
2. Membros não devem conseguir criar ou editar tasks em projetos inativos.
3. O projeto pode ser reativado pelo admin.

---

### RF030 — Exclusão Lógica de Projeto `● Média`

| Campo | Detalhe |
|---|---|
| **Módulo** | Projeto |
| **Ator** | Admin de Workspace, Admin de Empresa |
| **Descrição** | O admin deve poder realizar soft delete de um projeto. |

**Regras de Negócio:**
1. O campo `deleted_at` deve ser preenchido.
2. Todas as tasks do projeto devem se tornar inacessíveis.
3. O sistema deve exigir confirmação antes de executar.

---

### RF031 — Adição de Membros Externos ao Projeto `● Média`

| Campo | Detalhe |
|---|---|
| **Módulo** | Projeto |
| **Ator** | Admin de Workspace, Admin de Empresa |
| **Descrição** | O admin deve poder adicionar como colaboradores de um projeto membros de outros workspaces da mesma empresa. |

**Regras de Negócio:**
1. O membro adicionado ao projeto não precisa ser membro do workspace.
2. A adição cria um registro em `user_memberships` com `resource_type = 'projeto'`.
3. O membro externo tem acesso apenas ao projeto específico, não ao workspace inteiro.
4. O admin pode remover o acesso externo a qualquer momento.

---

## Módulo: Tasks

### RF032 — Criação de Task `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Tasks |
| **Ator** | Membros do Projeto |
| **Descrição** | Os membros com acesso ao projeto devem poder criar tasks no Kanban. |

**Regras de Negócio:**
1. O formulário deve conter: título, descrição, prioridade, data de início, data de vencimento, assigned.
2. A task deve ser vinculada a uma coluna do Kanban no momento da criação.
3. O campo `reporter_id` deve ser preenchido automaticamente com o id do usuário autenticado.
4. O campo `ordem` deve ser definido como o último da coluna (inserção no final).
5. O campo `created_by` deve registrar o id do criador.
6. O campo `status` deve refletir a coluna em que a task está.

---

### RF033 — Edição de Task `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Tasks |
| **Ator** | Membros do Projeto, Reporter, Assigned |
| **Descrição** | Os membros devem poder editar os dados de uma task. |

**Regras de Negócio:**
1. Campos editáveis: título, descrição, prioridade, data de início, data de vencimento, assigned.
2. Alterações devem atualizar o campo `updated_at`.
3. O campo `reporter_id` não deve ser alterável após a criação.
4. Apenas membros com acesso ao projeto podem editar tasks daquele projeto.

---

### RF034 — Movimentação de Task entre Colunas (Kanban) `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Tasks |
| **Ator** | Membros do Projeto |
| **Descrição** | Os membros devem poder mover tasks entre colunas e reordenar tasks dentro de uma mesma coluna via drag-and-drop. |

**Regras de Negócio:**
1. Ao mover para outra coluna, o campo `kanban_column_id` da task deve ser atualizado.
2. O campo `ordem` deve ser recalculado com base na posição de destino usando espaçamento numérico.
3. Ao reordenar dentro da mesma coluna, apenas o campo `ordem` deve ser atualizado.
4. A operação deve ser otimista no frontend e confirmada via API.
5. Quando os valores de `ordem` ficarem muito fragmentados, o sistema deve rebalancear os valores em background.

---

### RF035 — Exclusão Lógica de Task `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Tasks |
| **Ator** | Admin de Workspace, Admin de Empresa, Reporter |
| **Descrição** | O usuário autorizado deve poder realizar soft delete de uma task. |

**Regras de Negócio:**
1. O campo `deleted_at` deve ser preenchido com a data/hora atual.
2. A task não deve aparecer no Kanban após o soft delete.
3. O sistema deve exigir confirmação antes de executar.
4. Membros comuns (não reporter) não podem excluir tasks de outros usuários.

---

### RF036 — Atribuição de Responsável (Assigned) à Task `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Tasks |
| **Ator** | Membros do Projeto, Admin de Workspace |
| **Descrição** | O usuário deve poder atribuir ou alterar o responsável (assigned) de uma task. |

**Regras de Negócio:**
1. Somente membros com acesso ao projeto podem ser atribuídos como assigned.
2. A task deve ter no máximo um assigned por vez (MVP).
3. O assigned pode ser removido, deixando o campo `assigned_id` como `null`.
4. O sistema deve notificar o usuário atribuído (quando notificações forem implementadas).

---

### RF037 — Definição de Prioridade da Task `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Tasks |
| **Ator** | Membros do Projeto |
| **Descrição** | O usuário deve poder definir a prioridade de uma task. |

**Regras de Negócio:**
1. Os valores possíveis são: `Baixa`, `Média`, `Alta`, `Urgente`.
2. O valor padrão ao criar uma task é `Média`.
3. A prioridade deve ser visualmente diferenciada no card do Kanban.

---

### RF038 — Definição de Datas da Task `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Tasks |
| **Ator** | Membros do Projeto |
| **Descrição** | O usuário deve poder definir data de início e data de vencimento para uma task. |

**Regras de Negócio:**
1. A data de vencimento não pode ser anterior à data de início.
2. Ambas as datas são opcionais.
3. Tasks com data de vencimento expirada devem ser visualmente sinalizadas no Kanban.
4. O sistema deve suportar apenas data (sem hora) no MVP.

---

### RF039 — Visualização de Detalhes da Task `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Tasks |
| **Ator** | Membros do Projeto |
| **Descrição** | O usuário deve poder abrir uma task e visualizar todos os seus detalhes. |

**Regras de Negócio:**
1. A tela de detalhes deve exibir: título, descrição, prioridade, reporter, assigned, datas, coluna atual e histórico de alterações.
2. A tela de detalhes deve permitir edição inline dos campos.
3. Deve ser possível acessar a tela de detalhes diretamente pelo card no Kanban.

---

## Módulo: Permissões

### RF040 — Controle de Acesso por Membership `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Permissões |
| **Ator** | Sistema |
| **Descrição** | O sistema deve controlar todo acesso a recursos com base nos registros de `user_memberships`. |

**Regras de Negócio:**
1. Toda requisição a um recurso protegido deve verificar a existência de um membership válido (`deleted_at null`) para o usuário autenticado.
2. O campo `resource_type` identifica o tipo do recurso (`empresa`, `workspace`, `projeto`).
3. O campo `resource_id` identifica o recurso específico.
4. O campo `role` determina quais ações o usuário pode executar naquele recurso.
5. A hierarquia de permissões é: `superusuario > admin > workspace_admin > membro`.
6. Um admin de empresa herda implicitamente todos os poderes de workspace_admin e membro para os recursos de sua empresa.
7. Um membership inativo (soft deleted) deve ser tratado como inexistente para fins de controle de acesso.

---

### RF041 — Usuário Membro sem Workspace `● Média`

| Campo | Detalhe |
|---|---|
| **Módulo** | Permissões |
| **Ator** | Admin de Empresa |
| **Descrição** | O sistema deve suportar usuários vinculados à empresa mas ainda não alocados em nenhum workspace. |

**Regras de Negócio:**
1. Um usuário pode ter membership apenas no nível de empresa (`resource_type = 'empresa'`, `role = 'membro'`).
2. Esse usuário não deve visualizar nenhum workspace até ser adicionado a um.
3. Esse estado representa um usuário recém-cadastrado aguardando alocação.

---

## Módulo: Perfil

### RF042 — Edição de Perfil pelo Usuário `● Média`

| Campo | Detalhe |
|---|---|
| **Módulo** | Perfil |
| **Ator** | Todos os usuários autenticados |
| **Descrição** | O usuário deve poder editar seus próprios dados de perfil. |

**Regras de Negócio:**
1. Campos editáveis: nome, telefone, foto de perfil.
2. O email não deve ser editável pelo próprio usuário (alteração de email exige fluxo específico).
3. A senha pode ser alterada mediante confirmação da senha atual.
4. Alterações devem atualizar o campo `updated_at`.

---

## Módulo: Auditoria

### RF043 — Registro de Auditoria `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Auditoria |
| **Ator** | Sistema |
| **Descrição** | O sistema deve registrar automaticamente timestamps de criação, atualização e exclusão em todas as entidades. |

**Regras de Negócio:**
1. O campo `created_at` deve ser preenchido automaticamente no momento da criação do registro.
2. O campo `updated_at` deve ser atualizado automaticamente em qualquer alteração do registro.
3. O campo `deleted_at` deve ser preenchido apenas em operações de soft delete, permanecendo `null` caso contrário.
4. Esses campos não devem ser editáveis manualmente por nenhum usuário, incluindo o superusuário.

---

### RF044 — Registro do Criador (created_by) `● Alta`

| Campo | Detalhe |
|---|---|
| **Módulo** | Auditoria |
| **Ator** | Sistema |
| **Descrição** | O sistema deve registrar o usuário responsável pela criação de cada entidade. |

**Regras de Negócio:**
1. O campo `created_by` deve ser preenchido automaticamente com o id do usuário autenticado no momento da criação.
2. Para entidades criadas pelo superusuário, `created_by` deve registrar o id do superusuário.
3. O campo `created_by` não deve ser alterável após a criação.
4. O campo deve existir nas tabelas: `empresas`, `workspaces`, `projetos`, `tasks`.
