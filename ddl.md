```sql
CREATE SCHEMA IF NOT EXISTS xfood DEFAULT CHARACTER SET utf8;
USE xfood;

CREATE TABLE restaurants (
  id INT NOT NULL,
  name VARCHAR(60) NULL,
  description TEXT NULL,
  cnpj VARCHAR(14) NULL,
  rating DECIMAL(2,2) NULL,
  phone_number VARCHAR(45) NULL,
  PRIMARY KEY (id))
ENGINE = InnoDB;

CREATE TABLE restaurants_intervals (
  restaurant_id INT NOT NULL,
  week_day TINYINT NULL,
  open_hour TINYINT NULL,
  close_hour TINYINT NULL,
  PRIMARY KEY (restaurant_id),
  CONSTRAINT fk_restaurants_intervals_restaurants
    FOREIGN KEY (restaurant_id)
    REFERENCES restaurants (id)
ENGINE = InnoDB;

CREATE TABLE addresses (
  id INT NOT NULL,
  street INT NOT NULL,
  neighborhood VARCHAR(45) NOT NULL,
  zip_code VARCHAR(10) NOT NULL,
  number VARCHAR(8) NOT NULL,
  state VARCHAR(45) NOT NULL,
  city VARCHAR(45) NOT NULL,
  country_code VARCHAR(3) NULL,
  PRIMARY KEY (id),
  UNIQUE INDEX restaurant_id_UNIQUE ())
ENGINE = InnoDB;

CREATE TABLE restaurant_addresses (
  id INT NOT NULL,
  restaurant_id INT NOT NULL,
  address_id INT NOT NULL,
  PRIMARY KEY (id),
  UNIQUE INDEX restaurant_id_UNIQUE (restaurant_id),
  UNIQUE INDEX address_id_UNIQUE (address_id),
  CONSTRAINT fk_addresses_restaurants
    FOREIGN KEY (restaurant_id)
    REFERENCES restaurants (id),
  CONSTRAINT fk_restaurant_addresses_addresses
    FOREIGN KEY (address_id)
    REFERENCES addresses (id)
ENGINE = InnoDB;

CREATE TABLE users (
  id INT NULL,
  name VARCHAR(45) NULL,
  tax_id VARCHAR(45) NOT NULL,
  email VARCHAR(45) NOT NULL,
  phone_number VARCHAR(45) NOT NULL,
  PRIMARY KEY (id))
ENGINE = InnoDB;

CREATE TABLE restaurant_workers (
  id INT NOT NULL,
  user_id INT NOT NULL,
  restaurant_id INT NOT NULL,
  PRIMARY KEY (id),
  CONSTRAINT fk_restaurant_workers_users
    FOREIGN KEY (user_id)
    REFERENCES users (id,
  CONSTRAINT fk_restaurant_workers_restaurants
    FOREIGN KEY (restaurant_id)
    REFERENCES restaurants (id)
ENGINE = InnoDB;

CREATE TABLE riders (
  user_id INT NOT NULL,
  driver_license VARCHAR(10) NULL,
  vehicle_type ENUM('CAR', 'BIKE', 'MOTORCYCLE') NULL,
  PRIMARY KEY (user_id),
  CONSTRAINT fk_riders_users
    FOREIGN KEY (user_id)
    REFERENCES users (id)
ENGINE = InnoDB;

CREATE TABLE menu_items (
  id INT NOT NULL,
  restaurant_id INT NULL,
  name VARCHAR(45) NULL,
  description TEXT NULL,
  price DECIMAL(10,2) NULL,
  available TINYINT NULL DEFAULT 1,
  PRIMARY KEY (id),
  CONSTRAINT fk_menu_items_restaurants
    FOREIGN KEY (restaurant_id)
    REFERENCES restaurants (id)
ENGINE = InnoDB;

CREATE TABLE customers (
  user_id INT NOT NULL,
  PRIMARY KEY (user_id),
  CONSTRAINT fk_riders_users10
    FOREIGN KEY (user_id)
    REFERENCES users (id)
ENGINE = InnoDB;

CREATE TABLE orders (
  id INT NOT NULL,
  customer_id INT NOT NULL,
  restaurant_id INT NOT NULL,
  status ENUM('pending', 'confirmed', 'preparing', 'delivering', 'delivered', 'canceled') NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (id),
  CONSTRAINT fk_orders_customers
    FOREIGN KEY (customer_id)
    REFERENCES customers (user_id,
  CONSTRAINT fk_orders_restaurants
    FOREIGN KEY (restaurant_id)
    REFERENCES restaurants (id)
ENGINE = InnoDB;

CREATE TABLE order_items (
  id INT NOT NULL,
  order_id INT NOT NULL,
  menu_item_id INT NOT NULL,
  quantity INT NOT NULL,
  price DECIMAL(10,2) NOT NULL,
  PRIMARY KEY (id),
  CONSTRAINT fk_order_items_orders
    FOREIGN KEY (order_id)
    REFERENCES orders (id,
  CONSTRAINT fk_order_items_menu_items
    FOREIGN KEY (menu_item_id)
    REFERENCES menu_items (id)
ENGINE = InnoDB;

CREATE TABLE deliveries (
  id INT NOT NULL,
  order_id INT NOT NULL,
  rider_id INT NOT NULL,
  status ENUM('assigned', 'picked_up', 'delivered', 'failed') NOT NULL,
  assigned_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (id),
  CONSTRAINT fk_deliveries_riders
    FOREIGN KEY (rider_id)
    REFERENCES riders (user_id,
  CONSTRAINT fk_deliveries_orders
    FOREIGN KEY (order_id)
    REFERENCES orders (id)
ENGINE = InnoDB;

CREATE TABLE customer_shipping_addresses (
  customer_id INT NOT NULL,
  address_id INT NOT NULL,
  PRIMARY KEY (customer_id, address_id),
  CONSTRAINT fk_customer_addresses_customers
    FOREIGN KEY (customer_id)
    REFERENCES customers (user_id,
  CONSTRAINT fk_customer_addresses_addresses
    FOREIGN KEY (address_id)
    REFERENCES addresses (id)
ENGINE = InnoDB;

CREATE TABLE payments (
  id INT NOT NULL,
  order_id INT NOT NULL,
  amount DECIMAL(10,2) NOT NULL,
  method ENUM('credit_card', 'debit_card', 'pix', 'boleto', 'cash') NOT NULL,
  status ENUM('pending', 'paid', 'failed', 'refunded') NOT NULL,
  transaction_id VARCHAR(100) NOT NULL,
  paid_at TIMESTAMP NULL,
  created_at TIMESTAMP NULL DEFAULT DEFAULT_TIMESTAMP,
  PRIMARY KEY (id),
  UNIQUE INDEX transaction_id_UNIQUE (transaction_id),
  CONSTRAINT fk_payments_orders
    FOREIGN KEY (order_id)
    REFERENCES orders (id)
ENGINE = InnoDB;

CREATE TABLE order_reviews (
  id INT NOT NULL,
  order_id INT NOT NULL,
  reviewer_id INT NOT NULL,
  rating TINYINT NOT NULL,
  comment TEXT NULL,
  created_at TIMESTAMP NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (id))
ENGINE = InnoDB;

CREATE TABLE restaurant_reviews (
  order_review_id INT NOT NULL,
  rating TINYINT NOT NULL,
  comment TEXT NULL,
  PRIMARY KEY (order_review_id),
  CONSTRAINT fk_restaurant_reviews_order_reviews
    FOREIGN KEY (order_review_id)
    REFERENCES order_reviews (id)
ENGINE = InnoDB;

CREATE TABLE rider_reviews (
  order_review_id INT NOT NULL,
  rating TINYINT NOT NULL,
  comment TEXT NULL,
  PRIMARY KEY (order_review_id),
  CONSTRAINT fk_rider_reviews_order_reviews
    FOREIGN KEY (order_review_id)
    REFERENCES order_reviews (id)
ENGINE = InnoDB;
```
