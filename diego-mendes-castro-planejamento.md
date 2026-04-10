# Planejamento — Sistema de Restaurante (Individual)

## 1. Tema escolhido

**Sistema de Restaurante**

O sistema tem como objetivo gerenciar um restaurante específico, permitindo:
- cadastro de produtos (cardápio);
- controle de pedidos;
- gerenciamento de clientes;
- controle básico de usuários (funcionários).

---

## 2. Quais informações (campos) precisam ser salvas?

### Restaurante
- idRestaurante
- nome
- telefone
- email
- endereco
- horario_funcionamento

---

### Funcionários (Usuários)
- idUsuario
- nome
- email
- telefone
- senha
- cargo (admin, caixa, cozinha)
- ativo
- created

---

### Categorias do cardápio
- idCategoria
- nome
- descricao
- ativo

---

### Itens do cardápio
- idItem
- idCategoria
- nome
- descricao
- preco
- imagem
- ativo

---

### Clientes
- idCliente
- nome
- telefone
- email
- created

---

### Endereço do cliente
- idEndereco
- idCliente
- rua
- numero
- bairro
- cidade
- complemento

---

### Pedido
- idPedido
- idCliente
- tipo (delivery, retirada)
- status (pendente, preparo, pronto, entregue, cancelado)
- subtotal
- taxa_entrega
- total
- observacoes
- created

---

### Itens do pedido
- idPedidoItem
- idPedido
- idItem
- quantidade
- preco_unitario
- subtotal

---

## 3. Quais rotas (endpoints) são necessárias?

### Usuários
- `POST /api/usuarios/login`
- `POST /api/usuarios/cadastrar`
- `POST /api/usuarios/listar`

---

### Categorias
- `POST /api/categorias/cadastrar`
- `POST /api/categorias/listar`
- `POST /api/categorias/editar`
- `POST /api/categorias/excluir`

---

### Itens do cardápio
- `POST /api/itens/cadastrar`
- `POST /api/itens/listar`
- `POST /api/itens/editar`
- `POST /api/itens/excluir`

---

### Clientes
- `POST /api/clientes/cadastrar`
- `POST /api/clientes/listar`

---

### Pedidos
- `POST /api/pedidos/cadastrar`
- `POST /api/pedidos/listar`
- `POST /api/pedidos/detalhes`
- `POST /api/pedidos/atualizar-status`

---

## 4. Lista de 5 funcionalidades

### 1. Cadastro de cardápio
Permite cadastrar categorias e produtos com nome, descrição e preço.

---

### 2. Realizar pedidos
O sistema permite criar pedidos com múltiplos itens, calcular valores automaticamente e registrar informações do cliente.

---

### 3. Controle de status do pedido
Permite acompanhar o pedido nas etapas:
- Pendente
- Em preparo
- Pronto
- Entregue

---

### 4. Cadastro de clientes
Permite salvar informações dos clientes para facilitar pedidos futuros.

---

### 5. Controle de funcionários
Permite cadastrar usuários com diferentes cargos, como administrador, caixa e cozinha.

---

## 5. Lista de tabelas do banco

- restaurante
- usuarios
- categorias
- itens
- clientes
- enderecos_cliente
- pedidos
- pedido_itens

---

## 6. Melhorias e crítica do planejamento

Esse planejamento é simples e funcional, porém pode ser melhorado com:

- inclusão de pagamentos (Pix/cartão);
- sistema de entrega com cálculo por distância;
- histórico detalhado de pedidos;
- painel com relatórios de vendas;
- controle de estoque.

Mesmo sendo simples, o sistema já cobre o essencial para funcionamento de um restaurante.

---

## 7. Conclusão

O sistema proposto permite gerenciar um restaurante de forma prática, organizando cardápio, pedidos e clientes.

Ele é simples de implementar e pode ser expandido no futuro com novas funcionalidades conforme a necessidade do restaurante.