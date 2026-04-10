# 🍳 Sistema de Receitas Culinárias

Este projeto consiste no desenvolvimento de uma API para um site de receitas culinárias, permitindo que usuários criem, visualizem e interajam com receitas de forma simples e organizada.

---

## 📌 Objetivo

Criar uma aplicação que possibilite o cadastro, busca e gerenciamento de receitas culinárias, incluindo funcionalidades como favoritos e avaliações.

---

## ⚙️ Funcionalidades

* 👨‍🍳 Cadastro de receitas
* 🔍 Busca de receitas por nome ou ingredientes
* ❤️ Favoritar receitas
* 📝 Edição e exclusão de receitas pelo autor
* ⭐ Avaliação de receitas com notas e comentários

---

## 🌐 Endpoints da API

### 📌 Receitas

* `GET /receitas` → Lista todas as receitas
* `GET /receitas/{id}` → Retorna uma receita específica
* `POST /receitas` → Cria uma nova receita
* `PUT /receitas/{id}` → Atualiza uma receita existente
* `DELETE /receitas/{id}` → Remove uma receita

### 📌 Ingredientes

* `POST /receitas/{id}/ingredientes` → Adiciona ingredientes a uma receita

### 📌 Favoritos

* `POST /favoritos` → Adiciona uma receita aos favoritos
* `GET /favoritos` → Lista receitas favoritas do usuário

### 📌 Avaliações

* `POST /receitas/{id}/avaliacoes` → Avalia uma receita
* `GET /receitas/{id}/avaliacoes` → Lista avaliações de uma receita

---

## 🗄️ Estrutura do Banco de Dados

### 👤 Tabela: usuarios

* id
* nome
* email
* senha

### 🍲 Tabela: receitas

* id
* titulo
* descricao
* modo_preparo
* tempo_preparo
* porcoes
* dificuldade
* imagem_url
* usuario_id

### 🧂 Tabela: ingredientes

* id
* receita_id
* nome
* quantidade
* unidade

### 🏷️ Tabela: categorias

* id
* nome

### 🔗 Tabela: receita_categoria

* receita_id
* categoria_id

### ❤️ Tabela: favoritos

* id
* usuario_id
* receita_id

### ⭐ Tabela: avaliacoes

* id
* usuario_id
* receita_id
* nota
* comentario
* data

---

## 💡 Possíveis Melhorias

* Upload de imagens para receitas
* Filtros avançados (categoria, tempo, dificuldade)
* Sistema de autenticação com JWT
* Geração automática de lista de compras
* URL amigável para receitas (slug)

---

## 🚀 Tecnologias (exemplo)

* Backend: Node.js / Java / Spring Boot
* Banco de Dados: PostgreSQL / MySQL
* Frontend: HTML, CSS, JavaScript

---

## 📄 Licença

Este projeto é apenas para fins educacionais.
