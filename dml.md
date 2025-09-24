```sql
USE xfood;

INSERT INTO restaurants (id, name, description, cnpj, rating, phone_number) VALUES
(1, 'Restaurante A', 'Comida boa e ambiente agradável', '12345678000100', 4.50, '11987654321'),
(2, 'Pizzaria B', 'As melhores pizzas da cidade', '98765432000199', 4.80, '11912345678'),
(3, 'Hamburgueria C', 'Hambúrgueres artesanais deliciosos', '11223344000155', 4.20, '11998877665');

INSERT INTO restaurants_intervals (restaurant_id, week_day, open_hour, close_hour) VALUES
(1, 2, 10, 22), -- Restaurante A: Terça-feira, das 10h às 22h
(1, 3, 10, 22),
(2, 5, 18, 23), -- Pizzaria B: Sexta-feira, das 18h às 23h
(3, 1, 11, 23); -- Hamburgueria C: Segunda-feira, das 11h às 23h

INSERT INTO addresses (id, street, neighborhood, zip_code, number, state, city, country_code) VALUES
(101, 100, 'Centro', '01000-000', '123', 'SP', 'São Paulo', 'BRA'),
(102, 200, 'Jardins', '02000-000', '456', 'SP', 'São Paulo', 'BRA'),
(103, 300, 'Vila Olímpia', '03000-000', '789', 'SP', 'São Paulo', 'BRA'),
(104, 400, 'Santa Cecília', '04000-000', '10', 'SP', 'São Paulo', 'BRA');

INSERT INTO restaurant_addresses (id, restaurant_id, address_id) VALUES
(1, 1, 101),
(2, 2, 102),
(3, 3, 103);

INSERT INTO users (id, name, tax_id, email, phone_number) VALUES
(1001, 'João Silva', '11122233344', 'joao.silva@email.com', '11999998888'),
(1002, 'Maria Souza', '22233344455', 'maria.souza@email.com', '11988887777'),
(1003, 'Carlos Santos', '33344455566', 'carlos.santos@email.com', '11977776666'),
(1004, 'Ana Oliveira', '44455566677', 'ana.oliveira@email.com', '11966665555'),
(1005, 'Pedro Almeida', '55566677788', 'pedro.almeida@email.com', '11955554444');

INSERT INTO restaurant_workers (id, user_id, restaurant_id) VALUES
(1, 1001, 1), -- João Silva trabalha no Restaurante A
(2, 1002, 2); -- Maria Souza trabalha na Pizzaria B

INSERT INTO riders (user_id, driver_license, vehicle_type) VALUES
(1003, 'AB12345678', 'MOTORCYCLE'),
(1004, 'CD98765432', 'BIKE');

INSERT INTO menu_items (id, restaurant_id, name, description, price, available) VALUES
(1, 1, 'Prato Feito', 'Arroz, feijão, bife e salada', 25.00, 1),
(2, 1, 'Lasanha', 'Lasanha à bolonhesa', 35.00, 1),
(3, 2, 'Pizza Calabresa', 'Pizza com molho, queijo e calabresa', 45.00, 1),
(4, 2, 'Pizza Margherita', 'Pizza com molho, queijo e manjericão', 40.00, 1),
(5, 3, 'Hambúrguer Clássico', 'Pão, carne, queijo, alface, tomate', 30.00, 1),
(6, 3, 'Batata Frita', 'Porção de batata frita', 15.00, 1);

INSERT INTO customers (user_id) VALUES
(1005);

INSERT INTO orders (id, customer_id, restaurant_id, status, created_at) VALUES
(1, 1005, 1, 'pending', '2025-09-23 20:00:00'),
(2, 1005, 2, 'confirmed', '2025-09-23 20:30:00');

INSERT INTO order_items (id, order_id, menu_item_id, quantity, price) VALUES
(1, 1, 1, 1, 25.00),
(2, 1, 6, 1, 15.00),
(3, 2, 3, 1, 45.00);

INSERT INTO deliveries (id, order_id, rider_id, status, assigned_at) VALUES
(1, 1, 1003, 'assigned', '2025-09-23 20:10:00');

INSERT INTO customer_shipping_addresses (customer_id, address_id) VALUES
(1005, 104);

INSERT INTO payments (id, order_id, amount, method, status, transaction_id, paid_at, created_at) VALUES
(1, 1, 40.00, 'credit_card', 'paid', 'TRANS123456789', '2025-09-23 20:05:00', '2025-09-23 20:05:00'),
(2, 2, 45.00, 'pix', 'pending', 'TRANS987654321', NULL, '2025-09-23 20:35:00');

INSERT INTO order_reviews (id, order_id, reviewer_id, rating, comment, created_at) VALUES
(1, 1, 1005, 5, 'Comida excelente e entrega rápida!', '2025-09-23 21:00:00');

INSERT INTO restaurant_reviews (order_review_id, rating, comment) VALUES
(1, 5, 'Ótimo restaurante, super recomendo!');

INSERT INTO rider_reviews (order_review_id, rating, comment) VALUES
(1, 5, 'Motoboy super educado e ágil!');

```
