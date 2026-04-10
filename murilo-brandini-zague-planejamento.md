# App de Receitas — Receitalhada

> Documentação técnica para desenvolvimento de aplicação de **descoberta, cadastro, organização e interação com receitas culinárias**.

---

## Visão Geral

O **Receitalhada** conecta **usuários comuns** (que querem encontrar e salvar receitas) com **criadores de conteúdo culinário** (que publicam receitas detalhadas, fotos e dicas).  
A aplicação cobre o ciclo completo: descoberta de receitas → salvamento na coleção pessoal → execução (modo passo a passo) → avaliação/comentário → análise de engajamento para criadores.

---

## 1. Funcionalidades

### 1.1 Cadastro e Autenticação
Usuários se cadastram com papel definido (`leitor` ou `criador`).

- Leitores podem favoritar receitas, montar listas e avaliar.
- Criadores podem publicar e gerenciar receitas.

A autenticação utiliza JWT com refresh token para manter sessão segura.

Dados de cadastro:
- Leitor: nome, e-mail, senha, opcionalmente preferências alimentares (ex.: vegetariano, sem glúten).
- Criador: nome, e-mail, senha, bio curta, link de redes sociais.

---

### 1.2 Catálogo de Receitas
Criadores cadastram receitas com:

- título, descrição, tempo de preparo, porções;
- lista de ingredientes estruturada;
- passos do preparo;
- fotos e categoria (ex.: sobremesa, almoço, vegano etc.).

Leitores podem filtrar receitas por:

- categoria,
- tempo máximo de preparo,
- nível de dificuldade,
- restrição alimentar (tags),
- ingredientes disponíveis (busca por “tenho em casa”).

Cada receita possui status que controla visibilidade:

- `published` (visível a todos),
- `draft` (apenas para o criador),
- `archived` (oculta do catálogo público).

---

### 1.3 Listas e Favoritos (Coleções Pessoais)
O usuário leitor pode:

- favoritar receitas;
- criar coleções personalizadas (ex.: “Jantar rápido”, “Domingo em família”);
- adicionar/remover receitas das coleções.

Essas coleções funcionam como um “caderno de receitas digital” personalizável.

---

### 1.4 Modo de Preparo Passo a Passo
Ao abrir uma receita, o usuário pode ativar o **modo passo a passo**, que:

- exibe um passo por vez, em tela focada;
- permite marcar um passo como concluído;
- permite ajustar quantidade de porções, recalculando automaticamente a lista de ingredientes.

---

### 1.5 Comentários e Avaliações
Após preparar a receita, o usuário pode:

- atribuir uma nota de 1 a 5;
- deixar um comentário textual.

As avaliações:

- aparecem na página da receita;
- influenciam a nota média exibida;
- ajudam outros usuários a decidir se vão testar.

---

### 1.6 Feed Social e Seguir Criadores
Usuários podem:

- seguir criadores específicos;
- ver no feed as novas receitas publicadas por criadores que seguem;
- receber recomendações baseadas em histórico de visualização e avaliação.

---

### 1.7 Painel e Relatórios (Criador)
Para criadores, há um dashboard com:

- total de visualizações por receita e período;
- total de salvamentos (favoritos/coleções);
- avaliações médias por receita;
- comentários recentes.

Dados exportáveis para apoio à tomada de decisão (ex.: entender quais tipos de receita geram mais engajamento).

> **Nota:** novas funcionalidades foram adaptadas ao contexto de receitas (favoritos, coleções, feed social e painel do criador), cumprindo o “ciclo do negócio”: publicar → descobrir → salvar → cozinhar → avaliar → analisar.

---

## 2. Endpoints da API

> Convenção: todos os endpoints utilizam prefixo `/api/v1`.  
> Autenticação via `Authorization: Bearer <token>` exceto nas rotas públicas marcadas com `[público]`.

---

### 2.1 Autenticação

| Método | Endpoint | Descrição | Acesso |
|--------|----------|-----------|--------|
| `POST` | `/auth/register` | Cadastro de usuário (leitor ou criador) | público |
| `POST` | `/auth/login` | Login — retorna `access_token` e `refresh_token` | público |
| `POST` | `/auth/refresh` | Renovar access token usando refresh token | público |
| `POST` | `/auth/logout` | Invalidar refresh token | autenticado |
| `POST` | `/auth/forgot-password` | Solicitar reset de senha por e-mail | público |
| `POST` | `/auth/reset-password` | Confirmar nova senha com token do e-mail | público |

---

### 2.2 Usuários e Criadores

| Método | Endpoint | Descrição | Acesso |
|--------|----------|-----------|--------|
| `GET` | `/users/me` | Dados do usuário logado | autenticado |
| `PUT` | `/users/me` | Atualizar perfil (nome, preferências alimentares, bio, redes sociais) | autenticado |
| `GET` | `/creators` | Listar criadores de receita com filtros (nome, mais populares) | público |
| `GET` | `/creators/{id}` | Perfil público de um criador com suas principais receitas | público |
| `POST` | `/creators/{id}/follow` | Seguir criador | autenticado |
| `DELETE` | `/creators/{id}/follow` | Deixar de seguir criador | autenticado |

---

### 2.3 Receitas

| Método | Endpoint | Descrição | Acesso |
|--------|----------|-----------|--------|
| `GET` | `/recipes` | Listar receitas públicas (filtros: categoria, tempo, dificuldade, ingrediente) | público |
| `GET` | `/recipes/{id}` | Detalhes da receita, ingredientes, passos e fotos | público |
| `GET` | `/recipes/{id}/reviews` | Avaliações da receita | público |
| `POST` | `/recipes` | Cadastrar receita | criador |
| `PUT` | `/recipes/{id}` | Atualizar receita | criador (dono da receita) |
| `PATCH` | `/recipes/{id}/status` | Alterar status (`draft`, `published`, `archived`) | criador |
| `DELETE` | `/recipes/{id}` | Remover receita (soft delete) | criador |

---

### 2.4 Ingredientes e Passos da Receita

| Método | Endpoint | Descrição | Acesso |
|--------|----------|-----------|--------|
| `GET` | `/recipes/{id}/ingredients` | Listar ingredientes da receita | público |
| `POST` | `/recipes/{id}/ingredients` | Criar/atualizar lista de ingredientes | criador |
| `GET` | `/recipes/{id}/steps` | Listar passos da receita (modo leitura) | público |
| `POST` | `/recipes/{id}/steps` | Criar/atualizar passos da receita | criador |

> **Melhoria:** separar ingredientes e passos em endpoints próprios permite edição independente (ex.: ajustar apenas um passo sem mexer em todo o corpo da receita).

---

### 2.5 Favoritos e Coleções

| Método | Endpoint | Descrição | Acesso |
|--------|----------|-----------|--------|
| `POST` | `/recipes/{id}/favorite` | Marcar receita como favorita | leitor |
| `DELETE` | `/recipes/{id}/favorite` | Remover dos favoritos | leitor |
| `GET` | `/me/favorites` | Listar receitas favoritas do usuário | leitor |
| `GET` | `/collections` | Listar coleções do usuário | leitor |
| `POST` | `/collections` | Criar coleção de receitas | leitor |
| `PUT` | `/collections/{id}` | Renomear/editar coleção | leitor |
| `DELETE` | `/collections/{id}` | Excluir coleção | leitor |
| `POST` | `/collections/{id}/recipes` | Adicionar receita à coleção | leitor |
| `DELETE` | `/collections/{id}/recipes/{recipeId}` | Remover receita da coleção | leitor |

---

### 2.6 Comentários e Avaliações

| Método | Endpoint | Descrição | Acesso |
|--------|----------|-----------|--------|
| `POST` | `/recipes/{id}/reviews` | Enviar avaliação e comentário da receita | leitor |
| `GET` | `/recipes/{id}/reviews` | Ver avaliações de uma receita | público |

---

### 2.7 Feed Social

| Método | Endpoint | Descrição | Acesso |
|--------|----------|-----------|--------|
| `GET` | `/feed` | Feed de receitas de criadores seguidos + recomendações | autenticado |

---

### 2.8 Painel do Criador

| Método | Endpoint | Descrição | Acesso |
|--------|----------|-----------|--------|
| `GET` | `/dashboard/summary` | Totais: visualizações, favoritos, avaliações médias | criador |
| `GET` | `/dashboard/recipes/top` | Receitas com maior engajamento (views + favoritos + avaliações) | criador |
| `GET` | `/dashboard/recipes/{id}` | Métricas detalhadas de uma receita específica | criador |

---

## 3. Tabelas do Banco de Dados

> Banco relacional (PostgreSQL recomendado).  
> Todos os IDs utilizam `UUID`.  
> Todas as tabelas possuem `created_at TIMESTAMP` e `updated_at TIMESTAMP` (quando fizer sentido) com atualização automática.

---

### 3.1 `users` — Usuários

| Campo | Tipo | Restrição | Descrição |
|-------|------|-----------|-----------|
| `id` | UUID | PK | Identificador único |
| `name` | VARCHAR(150) | NOT NULL | Nome completo |
| `email` | VARCHAR(255) | NOT NULL, UNIQUE | E-mail de acesso |
| `password_hash` | VARCHAR(255) | NOT NULL | Senha criptografada (bcrypt) |
| `role` | ENUM | NOT NULL | `reader` ou `creator` |
| `bio` | TEXT | — | Biografia (para criadores) |
| `avatar_url` | VARCHAR(500) | — | Foto de perfil |
| `food_preferences` | VARCHAR(255) | — | Preferências alimentares (tags simples) |
| `instagram_url` | VARCHAR(255) | — | Rede social (criador) |
| `tiktok_url` | VARCHAR(255) | — | Rede social (criador) |
| `is_active` | BOOLEAN | DEFAULT true | Conta ativa ou bloqueada |
| `created_at` | TIMESTAMP | — | — |
| `updated_at` | TIMESTAMP | — | — |

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

> Mantida a mesma ideia do sistema original: suportar logout seguro e rotação de tokens.

---

### 3.3 `recipes` — Receitas

| Campo | Tipo | Restrição | Descrição |
|-------|------|-----------|-----------|
| `id` | UUID | PK | — |
| `creator_id` | UUID | FK users | Usuário criador da receita |
| `title` | VARCHAR(150) | NOT NULL | Título da receita |
| `description` | TEXT | — | Descrição/resumo |
| `category` | VARCHAR(100) | NOT NULL | Categoria (sobremesa, almoço, etc.) |
| `difficulty` | ENUM | NOT NULL | `easy`, `medium`, `hard` |
| `prep_time_min` | SMALLINT | NOT NULL | Tempo de preparo em minutos |
| `servings` | SMALLINT | NOT NULL | Número padrão de porções |
| `status` | ENUM | NOT NULL | `draft`, `published`, `archived` |
| `is_active` | BOOLEAN | DEFAULT true | Soft delete |
| `average_rating` | NUMERIC(2,1) | DEFAULT 0 | Nota média (cache) |
| `views_count` | INTEGER | DEFAULT 0 | Visualizações |
| `created_at` | TIMESTAMP | — | — |
| `updated_at` | TIMESTAMP | — | — |

---

### 3.4 `recipe_images` — Fotos das Receitas

| Campo | Tipo | Restrição | Descrição |
|-------|------|-----------|-----------|
| `id` | UUID | PK | — |
| `recipe_id` | UUID | FK recipes | Receita relacionada |
| `url` | VARCHAR(500) | NOT NULL | URL da imagem no storage |
| `is_primary` | BOOLEAN | DEFAULT false | Foto de capa |
| `order` | SMALLINT | DEFAULT 0 | Ordem de exibição |
| `created_at` | TIMESTAMP | — | — |

> Análogo a `vehicle_images`: permite múltiplas fotos por receita sem poluir a tabela principal.

---

### 3.5 `recipe_ingredients` — Ingredientes da Receita

| Campo | Tipo | Restrição | Descrição |
|-------|------|-----------|-----------|
| `id` | UUID | PK | — |
| `recipe_id` | UUID | FK recipes | Receita |
| `name` | VARCHAR(150) | NOT NULL | Nome do ingrediente |
| `quantity` | VARCHAR(50) | — | Quantidade (ex.: "2 xícaras") |
| `order` | SMALLINT | DEFAULT 0 | Ordem na lista de ingredientes |
| `created_at` | TIMESTAMP | — | — |

> Quantidade deixada como texto para flexibilidade (ex.: “a gosto”).

---

### 3.6 `recipe_steps` — Passos da Receita

| Campo | Tipo | Restrição | Descrição |
|-------|------|-----------|-----------|
| `id` | UUID | PK | — |
| `recipe_id` | UUID | FK recipes | Receita |
| `step_number` | SMALLINT | NOT NULL | Ordem do passo |
| `description` | TEXT | NOT NULL | Instrução do passo |
| `created_at` | TIMESTAMP | — | — |

> Essa tabela habilita o modo passo a passo, com controle de sequência.

---

### 3.7 `favorites` — Favoritos

| Campo | Tipo | Restrição | Descrição |
|-------|------|-----------|-----------|
| `user_id` | UUID | FK users, PK (composta) | Usuário |
| `recipe_id` | UUID | FK recipes, PK (composta) | Receita |
| `created_at` | TIMESTAMP | — | Quando foi favoritada |

> PK composta (`user_id`, `recipe_id`) evita favoritos duplicados.

---

### 3.8 `collections` — Coleções de Receitas

| Campo | Tipo | Restrição | Descrição |
|-------|------|-----------|-----------|
| `id` | UUID | PK | — |
| `owner_id` | UUID | FK users | Dono da coleção |
| `name` | VARCHAR(150) | NOT NULL | Nome da coleção |
| `is_default` | BOOLEAN | DEFAULT false | Indica se é coleção padrão (ex.: "Favoritas") |
| `created_at` | TIMESTAMP | — | — |
| `updated_at` | TIMESTAMP | — | — |

---

### 3.9 `collection_recipes` — Relação Coleção ↔ Receitas

| Campo | Tipo | Restrição | Descrição |
|-------|------|-----------|-----------|
| `collection_id` | UUID | FK collections, PK (composta) | Coleção |
| `recipe_id` | UUID | FK recipes, PK (composta) | Receita |
| `added_at` | TIMESTAMP | — | Quando foi adicionada |

---

### 3.10 `reviews` — Avaliações das Receitas

| Campo | Tipo | Restrição | Descrição |
|-------|------|-----------|-----------|
| `id` | UUID | PK | — |
| `recipe_id` | UUID | FK recipes | Receita avaliada |
| `user_id` | UUID | FK users | Usuário que avaliou |
| `rating` | SMALLINT | NOT NULL | Nota de 1 a 5 |
| `comment` | TEXT | — | Comentário livre |
| `created_at` | TIMESTAMP | — | — |

> `rating` será usado para recalcular `average_rating` em `recipes`.

---

### 3.11 `follows` — Seguidores de Criadores

| Campo | Tipo | Restrição | Descrição |
|-------|------|-----------|-----------|
| `follower_id` | UUID | FK users, PK (composta) | Usuário seguidor |
| `creator_id` | UUID | FK users, PK (composta) | Criador seguido |
| `created_at` | TIMESTAMP | — | Quando começou a seguir |

---

### 3.12 `recipe_metrics` — Métricas de Engajamento (opcional/cache)

| Campo | Tipo | Restrição | Descrição |
|-------|------|-----------|-----------|
| `recipe_id` | UUID | PK, FK recipes | Receita |
| `views_count` | INTEGER | DEFAULT 0 | Contagem de visualizações |
| `favorites_count` | INTEGER | DEFAULT 0 | Contagem de favoritos |
| `reviews_count` | INTEGER | DEFAULT 0 | Número de avaliações |

> Pode ser usada para consultas rápidas no dashboard, evitando agregações pesadas nas tabelas de log.

---

### 3.13 `notifications` — Notificações (opcional)

| Campo | Tipo | Restrição | Descrição |
|-------|------|-----------|-----------|
| `id` | UUID | PK | — |
| `user_id` | UUID | FK users | Destinatário |
| `recipe_id` | UUID | FK recipes | Receita relacionada (quando houver) |
| `type` | ENUM | NOT NULL | `new_recipe_from_followed`, `new_comment_on_recipe` etc. |
| `channel` | ENUM | NOT NULL | `in_app`, `email` |
| `sent_at` | TIMESTAMP | — | Momento do envio (null = pendente) |
| `status` | ENUM | NOT NULL | `pending`, `sent`, `failed` |
| `created_at` | TIMESTAMP | — | — |

> Adaptado do conceito de notificações do sistema de test drive, agora voltado a eventos sociais (novas receitas, comentários, etc.).

---

## 4. Resumo das Tabelas

| # | Tabela | Finalidade |
|---|--------|-----------|
| 1 | `users` | Leitores e criadores de receitas |
| 2 | `refresh_tokens` | Controle de sessão e logout |
| 3 | `recipes` | Receitas cadastradas |
| 4 | `recipe_images` | Galeria de fotos das receitas |
| 5 | `recipe_ingredients` | Ingredientes de cada receita |
| 6 | `recipe_steps` | Passo a passo das receitas |
| 7 | `favorites` | Receitas favoritas |
| 8 | `collections` | Coleções/cadernos de receitas do usuário |
| 9 | `collection_recipes` | Liga receitas às coleções |
| 10 | `reviews` | Avaliações das receitas |
| 11 | `follows` | Relação seguidor ↔ criador |
| 12 | `recipe_metrics` | Cache de métricas de engajamento |
| 13 | `notifications` | Histórico de notificações enviadas |

---

## 5. Melhorias/Adaptações em Relação ao Modelo de Test Drive

| Ponto | Sistema de Test Drive | Receitalhada (App de Receitas) |
|-------|-----------------------|---------------------------------|
| Entidade principal | Veículos e concessionárias | Receitas e criadores |
| Fluxo principal | Agendar test drive | Descobrir, salvar e cozinhar receitas |
| Agenda/slots | `availability_rules` / `availability_blocks` | Substituídos por `recipe_ingredients` e `recipe_steps` (foco em conteúdo, não em tempo) |
| Agendamentos | Tabela `appointments` | Não há agendamento; foco em interação (favoritos, coleções, reviews) |
| Avaliação | Pós test drive (veículo/serviço) | Avaliação da receita (sabor, execução) |
| Notificações | Lembretes e confirmações de agendamento | Novas receitas de criadores seguidos, novos comentários |
| Dashboard | Métricas de test drives | Métricas de engajamento de receitas |
