# Sistema de Agendamento de Test Drive

> Documentação técnica para desenvolvimento de aplicação de locação e agendamento de test drives em concessionárias.

---

## Visão Geral

O sistema conecta **clientes** que desejam agendar um test drive com **lojistas (concessionárias)** que gerenciam veículos, disponibilidade e atendimento. A aplicação deve cobrir o ciclo completo: descoberta de veículos → agendamento → confirmação → realização → avaliação.

---

## 1. Funcionalidades

### 1.1 Cadastro e Autenticação
Usuários se cadastram com papel definido (`cliente` ou `lojista`). Clientes informam CPF, CNH e telefone. Lojistas vinculam sua conta a uma concessionária via CNPJ. A autenticação utiliza JWT com refresh token para manter sessão segura.

### 1.2 Catálogo de Veículos
A concessionária cadastra os veículos disponíveis para test drive com marca, modelo, ano, cor, placa, fotos e especificações. O cliente pode filtrar por marca, modelo, ano e cidade. Cada veículo tem um status que impede novos agendamentos quando em manutenção ou já alocado.

### 1.3 Agendamento de Test Drive
O cliente seleciona o veículo, a concessionária, a data e o horário disponíveis. O sistema valida automaticamente a disponibilidade do slot e impede duplo agendamento. Antes de confirmar, exige CNH válida (número + validade) e aceite dos termos de uso do veículo.

### 1.4 Gestão de Agenda (Lojista)
O lojista configura os horários de funcionamento por dia da semana, a duração de cada slot de test drive e o limite de agendamentos simultâneos. Pode bloquear datas específicas (feriados, eventos internos) e acompanhar em tempo real todos os agendamentos com filtros por status, veículo e período. Também pode confirmar, remarcar ou cancelar agendamentos.

### 1.5 Notificações Automáticas
Disparos automáticos de e-mail e/ou SMS nos momentos: confirmação do agendamento, lembrete 24 horas antes, lembrete 1 hora antes e solicitação de avaliação após a realização. O lojista também recebe notificação a cada novo agendamento e cancelamento.

### 1.6 Avaliação Pós Test Drive
Após o agendamento ser marcado como `concluído`, o cliente recebe um link para avaliação com nota de 1 a 5 e comentário livre. As avaliações ficam vinculadas ao veículo e são visíveis para outros clientes, gerando reputação para a concessionária.

### 1.7 Painel e Relatórios (Lojista)
Dashboard com totais de agendamentos por período, taxa de comparecimento, veículos mais solicitados, avaliações médias e cancelamentos. Dados exportáveis para apoio à tomada de decisão.

> **Nota:** foram adicionadas as funcionalidades 1.6 (Avaliação) e 1.7 (Relatórios) para completar o ciclo do negócio. A funcionalidade original de "notificações" foi expandida para cobrir todos os gatilhos relevantes.

---

## 2. Endpoints da API

> Convenção: todos os endpoints utilizam prefixo `/api/v1`. Autenticação via `Authorization: Bearer <token>` exceto nas rotas públicas marcadas com `[público]`.

---

### 2.1 Autenticação

| Método | Endpoint | Descrição | Acesso |
|--------|----------|-----------|--------|
| `POST` | `/auth/register` | Cadastro de usuário (cliente ou lojista) | público |
| `POST` | `/auth/login` | Login — retorna `access_token` e `refresh_token` | público |
| `POST` | `/auth/refresh` | Renovar access token usando refresh token | público |
| `POST` | `/auth/logout` | Invalidar refresh token | autenticado |
| `POST` | `/auth/forgot-password` | Solicitar reset de senha por e-mail | público |
| `POST` | `/auth/reset-password` | Confirmar nova senha com token do e-mail | público |

---

### 2.2 Concessionárias

| Método | Endpoint | Descrição | Acesso |
|--------|----------|-----------|--------|
| `GET` | `/dealerships` | Listar concessionárias ativas (com filtros de cidade) | público |
| `GET` | `/dealerships/{id}` | Detalhes de uma concessionária | público |
| `POST` | `/dealerships` | Cadastrar concessionária | lojista |
| `PUT` | `/dealerships/{id}` | Atualizar dados da concessionária | lojista |
| `PATCH` | `/dealerships/{id}/status` | Ativar ou desativar concessionária | lojista |

> **Melhoria:** endpoints públicos de listagem de concessionárias são essenciais para que o cliente descubra onde agendar.

---

### 2.3 Veículos

| Método | Endpoint | Descrição | Acesso |
|--------|----------|-----------|--------|
| `GET` | `/vehicles` | Listar veículos disponíveis (filtros: marca, modelo, ano, cidade) | público |
| `GET` | `/vehicles/{id}` | Detalhes e fotos de um veículo | público |
| `GET` | `/vehicles/{id}/reviews` | Avaliações do veículo | público |
| `POST` | `/vehicles` | Cadastrar veículo | lojista |
| `PUT` | `/vehicles/{id}` | Atualizar dados do veículo | lojista |
| `PATCH` | `/vehicles/{id}/status` | Alterar status (available / in_use / maintenance) | lojista |
| `DELETE` | `/vehicles/{id}` | Remover veículo (soft delete) | lojista |

---

### 2.4 Disponibilidade

| Método | Endpoint | Descrição | Acesso |
|--------|----------|-----------|--------|
| `GET` | `/availability` | Consultar slots disponíveis (params: `dealership_id`, `vehicle_id`, `date`) | público |
| `GET` | `/availability/config` | Ver configuração de horários da concessionária | lojista |
| `POST` | `/availability/config` | Criar regra de disponibilidade semanal | lojista |
| `PUT` | `/availability/config/{id}` | Atualizar regra de disponibilidade | lojista |
| `POST` | `/availability/block` | Bloquear data específica (feriado, evento) | lojista |
| `DELETE` | `/availability/block/{id}` | Remover bloqueio de data | lojista |

> **Melhoria:** separar configuração semanal de bloqueios pontuais é fundamental para gerenciar exceções sem reescrever toda a grade.

---

### 2.5 Agendamentos

| Método | Endpoint | Descrição | Acesso |
|--------|----------|-----------|--------|
| `POST` | `/appointments` | Criar agendamento | cliente |
| `GET` | `/appointments` | Listar todos os agendamentos com filtros | lojista |
| `GET` | `/appointments/my` | Meus agendamentos (histórico do cliente) | cliente |
| `GET` | `/appointments/{id}` | Detalhes de um agendamento | cliente / lojista |
| `PATCH` | `/appointments/{id}/confirm` | Confirmar agendamento | lojista |
| `PATCH` | `/appointments/{id}/complete` | Marcar como concluído | lojista |
| `PATCH` | `/appointments/{id}/cancel` | Cancelar agendamento | cliente / lojista |
| `PATCH` | `/appointments/{id}/reschedule` | Remarcar data/horário | cliente / lojista |

> **Melhoria:** separar as ações de status (`confirm`, `complete`, `cancel`, `reschedule`) em endpoints distintos evita lógica condicional pesada no backend e facilita controle de permissão por papel.

---

### 2.6 Avaliações

| Método | Endpoint | Descrição | Acesso |
|--------|----------|-----------|--------|
| `POST` | `/appointments/{id}/review` | Enviar avaliação pós test drive | cliente |
| `GET` | `/vehicles/{id}/reviews` | Ver avaliações de um veículo | público |
| `GET` | `/dealerships/{id}/reviews` | Ver avaliações de uma concessionária | público |

---

### 2.7 Painel do Lojista

| Método | Endpoint | Descrição | Acesso |
|--------|----------|-----------|--------|
| `GET` | `/dashboard/summary` | Totais: agendamentos, taxa de comparecimento, avaliação média | lojista |
| `GET` | `/dashboard/appointments/by-period` | Agendamentos por dia/semana/mês | lojista |
| `GET` | `/dashboard/vehicles/top` | Veículos mais agendados | lojista |

---

## 3. Tabelas do Banco de Dados

> Banco relacional (PostgreSQL recomendado). Todos os IDs utilizam `UUID`. Todas as tabelas possuem `created_at TIMESTAMP` e `updated_at TIMESTAMP` com atualização automática.

---

### 3.1 `users` — Usuários

| Campo | Tipo | Restrição | Descrição |
|-------|------|-----------|-----------|
| `id` | UUID | PK | Identificador único |
| `name` | VARCHAR(150) | NOT NULL | Nome completo |
| `email` | VARCHAR(255) | NOT NULL, UNIQUE | E-mail de acesso |
| `password_hash` | VARCHAR(255) | NOT NULL | Senha criptografada (bcrypt) |
| `phone` | VARCHAR(20) | NOT NULL | Telefone com DDD |
| `cpf` | VARCHAR(14) | UNIQUE | CPF (apenas para clientes) |
| `driver_license_number` | VARCHAR(20) | — | Número da CNH |
| `driver_license_expiry` | DATE | — | Validade da CNH |
| `role` | ENUM | NOT NULL | `customer` ou `dealer` |
| `is_active` | BOOLEAN | DEFAULT true | Conta ativa ou bloqueada |
| `created_at` | TIMESTAMP | — | — |
| `updated_at` | TIMESTAMP | — | — |

> **Melhoria:** CNH (número + validade) movida para `users`, já que pertence ao motorista e não ao agendamento. O agendamento apenas referencia o usuário.

---

### 3.2 `refresh_tokens` — Tokens de Sessão

| Campo | Tipo | Restrição | Descrição |
|-------|------|-----------|-----------|
| `id` | UUID | PK | — |
| `user_id` | UUID | FK users | Dono do token |
| `token_hash` | VARCHAR(255) | NOT NULL, UNIQUE | Hash do refresh token |
| `expires_at` | TIMESTAMP | NOT NULL | Expiração |
| `revoked` | BOOLEAN | DEFAULT false | Token invalidado manualmente |
| `created_at` | TIMESTAMP | — | — |

> **Adição:** tabela necessária para suportar logout seguro e rotação de refresh tokens.

---

### 3.3 `dealerships` — Concessionárias

| Campo | Tipo | Restrição | Descrição |
|-------|------|-----------|-----------|
| `id` | UUID | PK | — |
| `owner_id` | UUID | FK users | Usuário lojista responsável |
| `name` | VARCHAR(150) | NOT NULL | Nome fantasia |
| `cnpj` | VARCHAR(18) | NOT NULL, UNIQUE | CNPJ formatado |
| `address` | VARCHAR(255) | NOT NULL | Logradouro e número |
| `city` | VARCHAR(100) | NOT NULL | Cidade |
| `state` | VARCHAR(2) | NOT NULL | UF |
| `zip_code` | VARCHAR(9) | NOT NULL | CEP |
| `phone` | VARCHAR(20) | NOT NULL | Telefone da concessionária |
| `email` | VARCHAR(255) | NOT NULL | E-mail de contato |
| `is_active` | BOOLEAN | DEFAULT true | — |
| `created_at` | TIMESTAMP | — | — |
| `updated_at` | TIMESTAMP | — | — |

> **Melhoria:** campos `state`, `zip_code` e `email` adicionados — essenciais para localização geográfica e notificações.

---

### 3.4 `vehicles` — Veículos

| Campo | Tipo | Restrição | Descrição |
|-------|------|-----------|-----------|
| `id` | UUID | PK | — |
| `dealership_id` | UUID | FK dealerships | Concessionária dona |
| `brand` | VARCHAR(80) | NOT NULL | Marca (ex: Toyota) |
| `model` | VARCHAR(100) | NOT NULL | Modelo (ex: Corolla) |
| `year` | SMALLINT | NOT NULL | Ano de fabricação |
| `color` | VARCHAR(50) | NOT NULL | Cor |
| `fuel_type` | ENUM | NOT NULL | `flex`, `gasoline`, `diesel`, `electric`, `hybrid` |
| `transmission` | ENUM | NOT NULL | `manual` ou `automatic` |
| `plate` | VARCHAR(10) | UNIQUE | Placa (Mercosul ou antiga) |
| `status` | ENUM | NOT NULL | `available`, `in_use`, `maintenance` |
| `is_active` | BOOLEAN | DEFAULT true | Soft delete |
| `created_at` | TIMESTAMP | — | — |
| `updated_at` | TIMESTAMP | — | — |

> **Melhoria:** `fuel_type` e `transmission` adicionados — são filtros de busca relevantes para o cliente.

---

### 3.5 `vehicle_images` — Fotos dos Veículos

| Campo | Tipo | Restrição | Descrição |
|-------|------|-----------|-----------|
| `id` | UUID | PK | — |
| `vehicle_id` | UUID | FK vehicles | Veículo relacionado |
| `url` | VARCHAR(500) | NOT NULL | URL da imagem no storage |
| `is_primary` | BOOLEAN | DEFAULT false | Foto de capa |
| `order` | SMALLINT | DEFAULT 0 | Ordem de exibição |
| `created_at` | TIMESTAMP | — | — |

> **Adição:** separar imagens em tabela própria permite múltiplas fotos por veículo sem poluir a tabela principal.

---

### 3.6 `availability_rules` — Grade de Horários

| Campo | Tipo | Restrição | Descrição |
|-------|------|-----------|-----------|
| `id` | UUID | PK | — |
| `dealership_id` | UUID | FK dealerships | Concessionária |
| `day_of_week` | SMALLINT | NOT NULL | 0 = Domingo … 6 = Sábado |
| `start_time` | TIME | NOT NULL | Horário de abertura |
| `end_time` | TIME | NOT NULL | Horário de encerramento |
| `slot_duration_min` | SMALLINT | NOT NULL | Duração de cada slot em minutos |
| `max_slots_parallel` | SMALLINT | DEFAULT 1 | Agendamentos simultâneos por slot |
| `is_active` | BOOLEAN | DEFAULT true | — |
| `created_at` | TIMESTAMP | — | — |

> **Melhoria:** `day_of_week` como `SMALLINT` (0–6) é mais portável e simples de consultar do que `ENUM` textual. `max_slots_parallel` substitui `max_per_slot` com nome mais claro.

---

### 3.7 `availability_blocks` — Bloqueios Pontuais

| Campo | Tipo | Restrição | Descrição |
|-------|------|-----------|-----------|
| `id` | UUID | PK | — |
| `dealership_id` | UUID | FK dealerships | Concessionária |
| `blocked_date` | DATE | NOT NULL | Data bloqueada |
| `start_time` | TIME | — | Bloqueio parcial: início (null = dia todo) |
| `end_time` | TIME | — | Bloqueio parcial: fim (null = dia todo) |
| `reason` | VARCHAR(200) | — | Motivo (feriado, evento, etc.) |
| `created_at` | TIMESTAMP | — | — |

> **Adição:** tabela separada para exceções pontuais na grade — feriados, eventos, manutenções programadas.

---

### 3.8 `appointments` — Agendamentos

| Campo | Tipo | Restrição | Descrição |
|-------|------|-----------|-----------|
| `id` | UUID | PK | — |
| `customer_id` | UUID | FK users | Cliente que agendou |
| `vehicle_id` | UUID | FK vehicles | Veículo escolhido |
| `dealership_id` | UUID | FK dealerships | Concessionária |
| `scheduled_date` | DATE | NOT NULL | Data do test drive |
| `scheduled_time` | TIME | NOT NULL | Horário de início |
| `status` | ENUM | NOT NULL | `pending`, `confirmed`, `completed`, `cancelled`, `no_show` |
| `cancelled_by` | ENUM | — | `customer` ou `dealer` (preenchido no cancelamento) |
| `cancellation_reason` | TEXT | — | Motivo do cancelamento |
| `notes` | TEXT | — | Observações do cliente |
| `confirmed_at` | TIMESTAMP | — | Momento da confirmação pelo lojista |
| `completed_at` | TIMESTAMP | — | Momento da conclusão |
| `created_at` | TIMESTAMP | — | — |
| `updated_at` | TIMESTAMP | — | — |

> **Melhorias:** status `no_show` adicionado (cliente não compareceu); `cancelled_by` e `cancellation_reason` adicionados para relatórios; `confirmed_at` e `completed_at` para rastreabilidade; CNH removida daqui e centralizada em `users`.

---

### 3.9 `reviews` — Avaliações

| Campo | Tipo | Restrição | Descrição |
|-------|------|-----------|-----------|
| `id` | UUID | PK | — |
| `appointment_id` | UUID | FK appointments, UNIQUE | Apenas uma avaliação por agendamento |
| `customer_id` | UUID | FK users | Avaliador |
| `vehicle_id` | UUID | FK vehicles | Veículo avaliado |
| `dealership_id` | UUID | FK dealerships | Concessionária avaliada |
| `vehicle_rating` | SMALLINT | NOT NULL | Nota do veículo (1–5) |
| `service_rating` | SMALLINT | NOT NULL | Nota do atendimento (1–5) |
| `comment` | TEXT | — | Comentário livre |
| `created_at` | TIMESTAMP | — | — |

> **Melhoria:** notas separadas para veículo e atendimento oferecem granularidade. `UNIQUE` em `appointment_id` impede avaliações duplicadas.

---

### 3.10 `notifications` — Notificações

| Campo | Tipo | Restrição | Descrição |
|-------|------|-----------|-----------|
| `id` | UUID | PK | — |
| `user_id` | UUID | FK users | Destinatário |
| `appointment_id` | UUID | FK appointments | Agendamento relacionado |
| `type` | ENUM | NOT NULL | `booking_confirmed`, `reminder_24h`, `reminder_1h`, `review_request`, `cancelled` |
| `channel` | ENUM | NOT NULL | `email` ou `sms` |
| `sent_at` | TIMESTAMP | — | Momento do envio (null = pendente) |
| `status` | ENUM | NOT NULL | `pending`, `sent`, `failed` |
| `created_at` | TIMESTAMP | — | — |

> **Adição:** tabela de notificações permite rastrear disparos, reprocessar falhas e auditar comunicações.

---

## 4. Resumo das Tabelas

| # | Tabela | Finalidade |
|---|--------|-----------|
| 1 | `users` | Clientes e lojistas |
| 2 | `refresh_tokens` | Controle de sessão e logout |
| 3 | `dealerships` | Concessionárias cadastradas |
| 4 | `vehicles` | Veículos disponíveis para test drive |
| 5 | `vehicle_images` | Galeria de fotos dos veículos |
| 6 | `availability_rules` | Grade semanal de horários |
| 7 | `availability_blocks` | Bloqueios pontuais (feriados, eventos) |
| 8 | `appointments` | Agendamentos de test drive |
| 9 | `reviews` | Avaliações pós test drive |
| 10 | `notifications` | Histórico de notificações enviadas |

---

## 5. Melhorias Aplicadas em Relação à Versão Inicial

| Ponto | Versão inicial | Versão revisada |
|-------|---------------|-----------------|
| Funcionalidades | 5 | 7 (notificações expandidas + relatórios) |
| Endpoints de auth | Login e registro | + refresh token, logout, reset de senha |
| Endpoints de concessionária | Ausentes | Adicionados (público e lojista) |
| Endpoints de disponibilidade | Básicos | Separados em regras semanais e bloqueios pontuais |
| Endpoints de agendamento | Status genérico via PATCH | Ações separadas: confirm, complete, cancel, reschedule |
| Tabelas | 6 | 10 |
| CNH no agendamento | `driver_license` em `appointments` | Movida para `users` (pertence ao motorista) |
| Imagens do veículo | `image_url` (um campo) | Tabela `vehicle_images` (múltiplas fotos) |
| Disponibilidade | Uma tabela | Separada em `availability_rules` + `availability_blocks` |
| Status de agendamento | 4 valores | 5 valores (adicionado `no_show`) |
| Avaliação | Uma nota geral | Duas notas: veículo e atendimento |
| Notificações | Apenas funcionalidade descrita | Tabela `notifications` para rastreamento |
| Sessão/Auth | Sem refresh token | Tabela `refresh_tokens` para segurança |
