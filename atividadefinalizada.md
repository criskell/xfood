## 1. Introdução

O banco de dados do sistema XFood foi projetado para gerenciar um **aplicativo de delivery de comida**, armazenando informações de restaurantes, usuários, pedidos, entregas, pagamentos e avaliações. A modelagem segue os princípios de **normalização** e **integridade referencial**, implementando um sistema completo de e-commerce gastronômico.

A implementação foi realizada no **MySQL**, utilizando comandos da **Data Definition Language (DDL)** para criar a estrutura e **Data Manipulation Language (DML)** para popular o banco com dados realistas.

---

## 2. Modelo Conceitual – Visão Geral

### 2.1 Entidades Principais

O sistema XFood é composto pelas seguintes entidades principais:

- **Users** → Usuários base do sistema (clientes, entregadores, funcionários)
- **Restaurants** → Estabelecimentos parceiros  
- **Menu Items** → Cardápio dos restaurantes
- **Orders** → Pedidos realizados pelos clientes
- **Deliveries** → Entregas dos pedidos
- **Payments** → Pagamentos dos pedidos
- **Reviews** → Avaliações de restaurantes e entregadores

### 2.2 Relacionamentos Principais

- Um **usuário** pode ser **cliente**, **entregador** ou **funcionário** de restaurante
- Um **restaurante** possui muitos **itens de cardápio** e **endereços**
- Um **pedido** contém muitos **itens** e possui um **pagamento** e uma **entrega**
- **Clientes** e **endereços** possuem relacionamento N:N (múltiplos endereços de entrega)

---

## 3. Implementação – DDL

### 3.1 Criação do Schema

```sql
CREATE SCHEMA IF NOT EXISTS xfood 
  DEFAULT CHARACTER SET utf8
  COLLATE utf8_general_ci;
  
USE xfood;
```

* **utf8** → suporte a caracteres especiais e acentos
* **utf8_general_ci** → comparações case-insensitive

---

### 3.2 Criação de Tabelas

#### Tabela `restaurants`

```sql
CREATE TABLE restaurants (
  id INT NOT NULL PRIMARY KEY,
  name VARCHAR(60) NULL,
  description TEXT NULL,
  cnpj VARCHAR(14) NULL,
  rating DECIMAL(2,2) NULL,                    -- Rating de 0.00 a 9.99
  phone_number VARCHAR(45) NULL
) ENGINE = InnoDB;
```

* `DECIMAL(2,2)` → rating preciso com 2 casas decimais
* `TEXT` → descrição longa sem limitação de tamanho
* `ENGINE=InnoDB` → suporte a transações e foreign keys

---

#### Tabela `users` (Usuários Base)

```sql
CREATE TABLE users (
  id INT NOT NULL PRIMARY KEY,
  name VARCHAR(45) NULL,
  tax_id VARCHAR(45) NOT NULL,                 -- CPF ou CNPJ
  email VARCHAR(45) NOT NULL,
  phone_number VARCHAR(45) NOT NULL
) ENGINE = InnoDB;
```

---

#### Tabelas de Especialização de Usuários

```sql
-- Clientes
CREATE TABLE customers (
  user_id INT NOT NULL PRIMARY KEY,
  CONSTRAINT fk_customers_users
    FOREIGN KEY (user_id) REFERENCES users (id)
) ENGINE = InnoDB;

-- Entregadores
CREATE TABLE riders (
  user_id INT NOT NULL PRIMARY KEY,
  driver_license VARCHAR(10) NULL,             -- CNH
  vehicle_type ENUM('CAR', 'BIKE', 'MOTORCYCLE') NULL,
  CONSTRAINT fk_riders_users
    FOREIGN KEY (user_id) REFERENCES users (id)
) ENGINE = InnoDB;

-- Funcionários de Restaurantes
CREATE TABLE restaurant_workers (
  id INT NOT NULL PRIMARY KEY,
  user_id INT NOT NULL,
  restaurant_id INT NOT NULL,
  CONSTRAINT fk_restaurant_workers_users
    FOREIGN KEY (user_id) REFERENCES users (id),
  CONSTRAINT fk_restaurant_workers_restaurants
    FOREIGN KEY (restaurant_id) REFERENCES restaurants (id)
) ENGINE = InnoDB;
```

* **Padrão de Herança** → users como tabela pai, especializações como filhas
* `ENUM` → controle de valores permitidos para tipo de veículo

---

#### Sistema de Endereços

```sql
CREATE TABLE addresses (
  id INT NOT NULL PRIMARY KEY,
  street INT NOT NULL,                         -- Código da rua
  neighborhood VARCHAR(45) NOT NULL,
  zip_code VARCHAR(10) NOT NULL,
  number VARCHAR(8) NOT NULL,
  state VARCHAR(45) NOT NULL,
  city VARCHAR(45) NOT NULL,
  country_code VARCHAR(3) NULL                 -- BRA, USA, etc.
) ENGINE = InnoDB;

-- Relacionamento N:N entre restaurantes e endereços
CREATE TABLE restaurant_addresses (
  id INT NOT NULL PRIMARY KEY,
  restaurant_id INT NOT NULL,
  address_id INT NOT NULL,
  UNIQUE INDEX restaurant_id_UNIQUE (restaurant_id),
  UNIQUE INDEX address_id_UNIQUE (address_id),
  CONSTRAINT fk_addresses_restaurants
    FOREIGN KEY (restaurant_id) REFERENCES restaurants (id),
  CONSTRAINT fk_restaurant_addresses_addresses
    FOREIGN KEY (address_id) REFERENCES addresses (id)
) ENGINE = InnoDB;

-- Endereços de entrega dos clientes (N:N)
CREATE TABLE customer_shipping_addresses (
  customer_id INT NOT NULL,
  address_id INT NOT NULL,
  PRIMARY KEY (customer_id, address_id),       -- Chave composta
  CONSTRAINT fk_customer_addresses_customers
    FOREIGN KEY (customer_id) REFERENCES customers (user_id),
  CONSTRAINT fk_customer_addresses_addresses
    FOREIGN KEY (address_id) REFERENCES addresses (id)
) ENGINE = InnoDB;
```

---

#### Sistema de Cardápio

```sql
CREATE TABLE menu_items (
  id INT NOT NULL PRIMARY KEY,
  restaurant_id INT NULL,
  name VARCHAR(45) NULL,
  description TEXT NULL,
  price DECIMAL(10,2) NULL,                    -- Preço com 2 casas decimais
  available TINYINT NULL DEFAULT 1,            -- 1=disponível, 0=indisponível
  CONSTRAINT fk_menu_items_restaurants
    FOREIGN KEY (restaurant_id) REFERENCES restaurants (id)
) ENGINE = InnoDB;
```

* `DECIMAL(10,2)` → preços até R$ 99.999.999,99
* `DEFAULT 1` → itens criados como disponíveis por padrão

---

#### Sistema de Pedidos

```sql
CREATE TABLE orders (
  id INT NOT NULL PRIMARY KEY,
  customer_id INT NOT NULL,
  restaurant_id INT NOT NULL,
  status ENUM('pending', 'confirmed', 'preparing', 'delivering', 'delivered', 'canceled') NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  CONSTRAINT fk_orders_customers
    FOREIGN KEY (customer_id) REFERENCES customers (user_id),
  CONSTRAINT fk_orders_restaurants
    FOREIGN KEY (restaurant_id) REFERENCES restaurants (id)
) ENGINE = InnoDB;

-- Itens do pedido
CREATE TABLE order_items (
  id INT NOT NULL PRIMARY KEY,
  order_id INT NOT NULL,
  menu_item_id INT NOT NULL,
  quantity INT NOT NULL,
  price DECIMAL(10,2) NOT NULL,                -- Preço no momento do pedido
  CONSTRAINT fk_order_items_orders
    FOREIGN KEY (order_id) REFERENCES orders (id),
  CONSTRAINT fk_order_items_menu_items
    FOREIGN KEY (menu_item_id) REFERENCES menu_items (id)
) ENGINE = InnoDB;
```

* **Histórico de Preço** → price em order_items preserva valor na data do pedido
* `CURRENT_TIMESTAMP` → registro automático de quando o pedido foi criado

---

#### Sistema de Entregas

```sql
CREATE TABLE deliveries (
  id INT NOT NULL PRIMARY KEY,
  order_id INT NOT NULL,
  rider_id INT NOT NULL,
  status ENUM('assigned', 'picked_up', 'delivered', 'failed') NOT NULL,
  assigned_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  CONSTRAINT fk_deliveries_riders
    FOREIGN KEY (rider_id) REFERENCES riders (user_id),
  CONSTRAINT fk_deliveries_orders
    FOREIGN KEY (order_id) REFERENCES orders (id)
) ENGINE = InnoDB;
```

---

#### Sistema de Pagamentos

```sql
CREATE TABLE payments (
  id INT NOT NULL PRIMARY KEY,
  order_id INT NOT NULL,
  amount DECIMAL(10,2) NOT NULL,
  method ENUM('credit_card', 'debit_card', 'pix', 'boleto', 'cash') NOT NULL,
  status ENUM('pending', 'paid', 'failed', 'refunded') NOT NULL,
  transaction_id VARCHAR(100) NOT NULL,
  paid_at TIMESTAMP NULL,
  created_at TIMESTAMP NULL DEFAULT CURRENT_TIMESTAMP,
  UNIQUE INDEX transaction_id_UNIQUE (transaction_id),
  CONSTRAINT fk_payments_orders
    FOREIGN KEY (order_id) REFERENCES orders (id)
) ENGINE = InnoDB;
```

* `transaction_id UNIQUE` → evita processamento duplicado de pagamentos
* `paid_at` NULL → permite pagamentos pendentes

---

#### Sistema de Avaliações

```sql
-- Avaliação geral do pedido
CREATE TABLE order_reviews (
  id INT NOT NULL PRIMARY KEY,
  order_id INT NOT NULL,
  reviewer_id INT NOT NULL,
  rating TINYINT NOT NULL,                     -- Nota de 1 a 5
  comment TEXT NULL,
  created_at TIMESTAMP NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE = InnoDB;

-- Avaliação específica do restaurante
CREATE TABLE restaurant_reviews (
  order_review_id INT NOT NULL PRIMARY KEY,
  rating TINYINT NOT NULL,
  comment TEXT NULL,
  CONSTRAINT fk_restaurant_reviews_order_reviews
    FOREIGN KEY (order_review_id) REFERENCES order_reviews (id)
) ENGINE = InnoDB;

-- Avaliação específica do entregador  
CREATE TABLE rider_reviews (
  order_review_id INT NOT NULL PRIMARY KEY,
  rating TINYINT NOT NULL,
  comment TEXT NULL,
  CONSTRAINT fk_rider_reviews_order_reviews
    FOREIGN KEY (order_review_id) REFERENCES order_reviews (id)
) ENGINE = InnoDB;
```

* **Avaliação Compartilhada** → uma avaliação geral com especializações para restaurante e entregador

---

#### Horários de Funcionamento

```sql
CREATE TABLE restaurants_intervals (
  restaurant_id INT NOT NULL PRIMARY KEY,
  week_day TINYINT NULL,                       -- 1=Segunda, 2=Terça, etc.
  open_hour TINYINT NULL,                      -- Hora de abertura (0-23)
  close_hour TINYINT NULL,                     -- Hora de fechamento (0-23)
  CONSTRAINT fk_restaurants_intervals_restaurants
    FOREIGN KEY (restaurant_id) REFERENCES restaurants (id)
) ENGINE = InnoDB;
```

---

## 4. Implementação – DML (Dados de Exemplo)

### 4.1 Inserção de Restaurantes e Usuários

```sql
-- Restaurantes
INSERT INTO restaurants (id, name, description, cnpj, rating, phone_number) VALUES
(1, 'Restaurante A', 'Comida boa e ambiente agradável', '12345678000100', 4.50, '11987654321'),
(2, 'Pizzaria B', 'As melhores pizzas da cidade', '98765432000199', 4.80, '11912345678'),
(3, 'Hamburgueria C', 'Hambúrgueres artesanais deliciosos', '11223344000155', 4.20, '11998877665');

-- Usuários base
INSERT INTO users (id, name, tax_id, email, phone_number) VALUES
(1001, 'João Silva', '11122233344', 'joao.silva@email.com', '11999998888'),
(1002, 'Maria Souza', '22233344455', 'maria.souza@email.com', '11988887777'),
(1003, 'Carlos Santos', '33344455566', 'carlos.santos@email.com', '11977776666'),
(1004, 'Ana Oliveira', '44455566677', 'ana.oliveira@email.com', '11966665555'),
(1005, 'Pedro Almeida', '55566677788', 'pedro.almeida@email.com', '11955554444');

-- Especializações de usuários
INSERT INTO customers (user_id) VALUES (1005);
INSERT INTO riders (user_id, driver_license, vehicle_type) VALUES 
(1003, 'AB12345678', 'MOTORCYCLE'),
(1004, 'CD98765432', 'BIKE');
INSERT INTO restaurant_workers (id, user_id, restaurant_id) VALUES
(1, 1001, 1),
(2, 1002, 2);
```

### 4.2 Simulação de Pedido Completo

```sql
-- Cardápio
INSERT INTO menu_items (id, restaurant_id, name, description, price, available) VALUES
(1, 1, 'Prato Feito', 'Arroz, feijão, bife e salada', 25.00, 1),
(2, 1, 'Lasanha', 'Lasanha à bolonhesa', 35.00, 1);

-- Pedido
INSERT INTO orders (id, customer_id, restaurant_id, status, created_at) 
VALUES (1, 1005, 1, 'pending', '2025-09-23 20:00:00');

-- Itens do pedido
INSERT INTO order_items (id, order_id, menu_item_id, quantity, price) VALUES
(1, 1, 1, 1, 25.00),
(2, 1, 2, 1, 35.00);

-- Pagamento
INSERT INTO payments (id, order_id, amount, method, status, transaction_id, created_at)
VALUES (1, 1, 60.00, 'credit_card', 'paid', 'TRANS123456789', '2025-09-23 20:05:00');

-- Entrega
INSERT INTO deliveries (id, order_id, rider_id, status, assigned_at) 
VALUES (1, 1, 1003, 'assigned', '2025-09-23 20:10:00');
```

---

## 5. Considerações Técnicas

| Conceito | Significado | Exemplo |
|----------|-------------|---------|
| **ENGINE=InnoDB** | Motor que suporta FK e transações | `CREATE TABLE ... ENGINE=InnoDB;` |
| **ENUM** | Lista de valores pré-definidos | `status ENUM('pending', 'confirmed')` |
| **DECIMAL(10,2)** | Número decimal preciso | Preços monetários |
| **TIMESTAMP** | Data e hora com timezone | `DEFAULT CURRENT_TIMESTAMP` |
| **UNIQUE INDEX** | Previne duplicatas | Email único, transaction_id único |
| **Chave Composta** | PK com múltiplas colunas | `PRIMARY KEY (customer_id, address_id)` |
| **Herança por Tabela** | Especialização de entidades | users → customers, riders, workers |

---

## 📖 Dicionário de Dados – Sistema XFood

### Tabela `restaurants`
| Coluna | Tipo | PK | FK | Not Null | Único | Default | Descrição |
|--------|------|----|----|----------|-------|---------|-----------|
| id | INT | ✓ | | ✓ | | | Identificador único do restaurante |
| name | VARCHAR(60) | | | | | | Nome do restaurante |
| description | TEXT | | | | | | Descrição detalhada do restaurante |
| cnpj | VARCHAR(14) | | | | | | CNPJ do estabelecimento |
| rating | DECIMAL(2,2) | | | | | | Avaliação média (0.00-9.99) |
| phone_number | VARCHAR(45) | | | | | | Telefone de contato |

### Tabela `restaurants_intervals`
| Coluna | Tipo | PK | FK | Not Null | Único | Default | Descrição |
|--------|------|----|----|----------|-------|---------|-----------|
| restaurant_id | INT | ✓ | restaurants.id | ✓ | | | Restaurante relacionado |
| week_day | TINYINT | | | | | | Dia da semana (1=Segunda, 2=Terça, etc.) |
| open_hour | TINYINT | | | | | | Hora de abertura (0-23) |
| close_hour | TINYINT | | | | | | Hora de fechamento (0-23) |

### Tabela `addresses`
| Coluna | Tipo | PK | FK | Not Null | Único | Default | Descrição |
|--------|------|----|----|----------|-------|---------|-----------|
| id | INT | ✓ | | ✓ | | | Identificador único do endereço |
| street | INT | | | ✓ | | | Código da rua |
| neighborhood | VARCHAR(45) | | | ✓ | | | Bairro |
| zip_code | VARCHAR(10) | | | ✓ | | | CEP |
| number | VARCHAR(8) | | | ✓ | | | Número do endereço |
| state | VARCHAR(45) | | | ✓ | | | Estado (UF) |
| city | VARCHAR(45) | | | ✓ | | | Cidade |
| country_code | VARCHAR(3) | | | | | | Código do país (BRA, USA, etc.) |

### Tabela `restaurant_addresses`
| Coluna | Tipo | PK | FK | Not Null | Único | Default | Descrição |
|--------|------|----|----|----------|-------|---------|-----------|
| id | INT | ✓ | | ✓ | | | Identificador único da associação |
| restaurant_id | INT | | restaurants.id | ✓ | ✓ | | Restaurante relacionado |
| address_id | INT | | addresses.id | ✓ | ✓ | | Endereço relacionado |

### Tabela `users`
| Coluna | Tipo | PK | FK | Not Null | Único | Default | Descrição |
|--------|------|----|----|----------|-------|---------|-----------|
| id | INT | ✓ | | ✓ | | | Identificador único do usuário |
| name | VARCHAR(45) | | | | | | Nome completo do usuário |
| tax_id | VARCHAR(45) | | | ✓ | | | CPF ou CNPJ |
| email | VARCHAR(45) | | | ✓ | | | Email do usuário |
| phone_number | VARCHAR(45) | | | ✓ | | | Telefone de contato |

### Tabela `customers`
| Coluna | Tipo | PK | FK | Not Null | Único | Default | Descrição |
|--------|------|----|----|----------|-------|---------|-----------|
| user_id | INT | ✓ | users.id | ✓ | | | Usuário que é cliente |

### Tabela `riders`
| Coluna | Tipo | PK | FK | Not Null | Único | Default | Descrição |
|--------|------|----|----|----------|-------|---------|-----------|
| user_id | INT | ✓ | users.id | ✓ | | | Usuário que é entregador |
| driver_license | VARCHAR(10) | | | | | | Número da CNH |
| vehicle_type | ENUM('CAR', 'BIKE', 'MOTORCYCLE') | | | | | | Tipo de veículo usado |

### Tabela `restaurant_workers`
| Coluna | Tipo | PK | FK | Not Null | Único | Default | Descrição |
|--------|------|----|----|----------|-------|---------|-----------|
| id | INT | ✓ | | ✓ | | | Identificador único do funcionário |
| user_id | INT | | users.id | ✓ | | | Usuário funcionário |
| restaurant_id | INT | | restaurants.id | ✓ | | | Restaurante onde trabalha |

### Tabela `menu_items`
| Coluna | Tipo | PK | FK | Not Null | Único | Default | Descrição |
|--------|------|----|----|----------|-------|---------|-----------|
| id | INT | ✓ | | ✓ | | | Identificador único do item |
| restaurant_id | INT | | restaurants.id | | | | Restaurante dono do item |
| name | VARCHAR(45) | | | | | | Nome do prato/produto |
| description | TEXT | | | | | | Descrição detalhada do item |
| price | DECIMAL(10,2) | | | | | | Preço do item |
| available | TINYINT | | | | | 1 | Disponibilidade (1=sim, 0=não) |

### Tabela `orders`
| Coluna | Tipo | PK | FK | Not Null | Único | Default | Descrição |
|--------|------|----|----|----------|-------|---------|-----------|
| id | INT | ✓ | | ✓ | | | Identificador único do pedido |
| customer_id | INT | | customers.user_id | ✓ | | | Cliente que fez o pedido |
| restaurant_id | INT | | restaurants.id | ✓ | | | Restaurante do pedido |
| status | ENUM('pending', 'confirmed', 'preparing', 'delivering', 'delivered', 'canceled') | | | ✓ | | | Status atual do pedido |
| created_at | TIMESTAMP | | | ✓ | | CURRENT_TIMESTAMP | Data/hora da criação |

### Tabela `order_items`
| Coluna | Tipo | PK | FK | Not Null | Único | Default | Descrição |
|--------|------|----|----|----------|-------|---------|-----------|
| id | INT | ✓ | | ✓ | | | Identificador único do item do pedido |
| order_id | INT | | orders.id | ✓ | | | Pedido relacionado |
| menu_item_id | INT | | menu_items.id | ✓ | | | Item do cardápio |
| quantity | INT | | | ✓ | | | Quantidade pedida |
| price | DECIMAL(10,2) | | | ✓ | | | Preço unitário no momento do pedido |

### Tabela `deliveries`
| Coluna | Tipo | PK | FK | Not Null | Único | Default | Descrição |
|--------|------|----|----|----------|-------|---------|-----------|
| id | INT | ✓ | | ✓ | | | Identificador único da entrega |
| order_id | INT | | orders.id | ✓ | | | Pedido a ser entregue |
| rider_id | INT | | riders.user_id | ✓ | | | Entregador responsável |
| status | ENUM('assigned', 'picked_up', 'delivered', 'failed') | | | ✓ | | | Status da entrega |
| assigned_at | TIMESTAMP | | | ✓ | | CURRENT_TIMESTAMP | Data/hora da atribuição |

### Tabela `customer_shipping_addresses`
| Coluna | Tipo | PK | FK | Not Null | Único | Default | Descrição |
|--------|------|----|----|----------|-------|---------|-----------|
| customer_id | INT | ✓* | customers.user_id | ✓ | | | Cliente relacionado |
| address_id | INT | ✓* | addresses.id | ✓ | | | Endereço de entrega |

> *PK composta (customer_id + address_id)

### Tabela `payments`
| Coluna | Tipo | PK | FK | Not Null | Único | Default | Descrição |
|--------|------|----|----|----------|-------|---------|-----------|
| id | INT | ✓ | | ✓ | | | Identificador único do pagamento |
| order_id | INT | | orders.id | ✓ | | | Pedido relacionado |
| amount | DECIMAL(10,2) | | | ✓ | | | Valor do pagamento |
| method | ENUM('credit_card', 'debit_card', 'pix', 'boleto', 'cash') | | | ✓ | | | Método de pagamento |
| status | ENUM('pending', 'paid', 'failed', 'refunded') | | | ✓ | | | Status do pagamento |
| transaction_id | VARCHAR(100) | | | ✓ | ✓ | | ID único da transação |
| paid_at | TIMESTAMP | | | | | | Data/hora do pagamento |
| created_at | TIMESTAMP | | | | | CURRENT_TIMESTAMP | Data/hora da criação |

### Tabela `order_reviews`
| Coluna | Tipo | PK | FK | Not Null | Único | Default | Descrição |
|--------|------|----|----|----------|-------|---------|-----------|
| id | INT | ✓ | | ✓ | | | Identificador único da avaliação |
| order_id | INT | | orders.id | ✓ | | | Pedido avaliado |
| reviewer_id | INT | | users.id | ✓ | | | Usuário que fez a avaliação |
| rating | TINYINT | | | ✓ | | | Nota geral (1-5) |
| comment | TEXT | | | | | | Comentário da avaliação |
| created_at | TIMESTAMP | | | | | CURRENT_TIMESTAMP | Data/hora da avaliação |

### Tabela `restaurant_reviews`
| Coluna | Tipo | PK | FK | Not Null | Único | Default | Descrição |
|--------|------|----|----|----------|-------|---------|-----------|
| order_review_id | INT | ✓ | order_reviews.id | ✓ | | | Avaliação geral relacionada |
| rating | TINYINT | | | ✓ | | | Nota específica do restaurante (1-5) |
| comment | TEXT | | | | | | Comentário sobre o restaurante |

### Tabela `rider_reviews`
| Coluna | Tipo | PK | FK | Not Null | Único | Default | Descrição |
|--------|------|----|----|----------|-------|---------|-----------|
| order_review_id | INT | ✓ | order_reviews.id | ✓ | | | A
