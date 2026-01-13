# Схема базы данных в 3 нормальной форме

## Обоснование 3НФ

Схема базы данных соответствует третьей нормальной форме (3НФ):

1. **1НФ (Первая нормальная форма)**:
   - Все атрибуты атомарны (нет составных или многозначных атрибутов)
   - Каждая таблица имеет первичный ключ
   - Нет повторяющихся групп

2. **2НФ (Вторая нормальная форма)**:
   - Выполнены требования 1НФ
   - Отсутствуют частичные зависимости от составных ключей
   - Все неключевые атрибуты полностью зависят от первичного ключа

3. **3НФ (Третья нормальная форма)**:
   - Выполнены требования 2НФ
   - Отсутствуют транзитивные зависимости
   - Все неключевые атрибуты зависят только от первичного ключа

## Диаграмма ER (Entity-Relationship)

```
┌─────────────────────────────────────────────────────────────────────┐
│                         USERS                                       │
├─────────────────────────────────────────────────────────────────────┤
│ PK  id: INTEGER                                                     │
│     email: VARCHAR(255) UNIQUE NOT NULL                             │
│     password_digest: VARCHAR(255) NOT NULL                          │
│     full_name: VARCHAR(255) NOT NULL                                │
│     phone: VARCHAR(20)                                              │
│     role: ENUM('client', 'admin') DEFAULT 'client'                  │
│     created_at: TIMESTAMP                                           │
│     updated_at: TIMESTAMP                                           │
└──────────────────────┬──────────────────────────────────────────────┘
                       │
                       │ 1
                       │
                       │ *
┌──────────────────────▼──────────────────────────────────────────────┐
│                       BOOKINGS                                      │
├─────────────────────────────────────────────────────────────────────┤
│ PK  id: INTEGER                                                     │
│ FK  user_id: INTEGER NOT NULL → users.id                            │
│ FK  tour_id: INTEGER NOT NULL → tours.id                            │
│     booking_date: TIMESTAMP NOT NULL                                │
│     number_of_people: INTEGER NOT NULL CHECK (> 0)                  │
│     total_price: DECIMAL(10,2) NOT NULL CHECK (>= 0)                │
│     status: ENUM('pending','confirmed','cancelled') DEFAULT 'pending'│
│     notes: TEXT                                                     │
│     created_at: TIMESTAMP                                           │
│     updated_at: TIMESTAMP                                           │
└──────────────────────┬──────────────────────────────────────────────┘
                       │
                       │ *
                       │
                       │ 1
┌──────────────────────▼──────────────────────────────────────────────┐
│                         TOURS                                       │
├─────────────────────────────────────────────────────────────────────┤
│ PK  id: INTEGER                                                     │
│     title: VARCHAR(255) NOT NULL                                    │
│     description: TEXT                                               │
│     destination: VARCHAR(255) NOT NULL                              │
│     duration_days: INTEGER NOT NULL CHECK (> 0)                     │
│     price: DECIMAL(10,2) NOT NULL CHECK (> 0)                       │
│     available_seats: INTEGER NOT NULL CHECK (>= 0)                  │
│     start_date: DATE NOT NULL                                       │
│     end_date: DATE NOT NULL CHECK (end_date > start_date)           │
│ FK  category_id: INTEGER → categories.id                            │
│     created_at: TIMESTAMP                                           │
│     updated_at: TIMESTAMP                                           │
└──────────────────────┬──────────────────────────────────────────────┘
                       │
                       │ *
                       │
                       │ 1
┌──────────────────────▼──────────────────────────────────────────────┐
│                      CATEGORIES                                     │
├─────────────────────────────────────────────────────────────────────┤
│ PK  id: INTEGER                                                     │
│     name: VARCHAR(100) UNIQUE NOT NULL                              │
│     description: TEXT                                               │
│     created_at: TIMESTAMP                                           │
│     updated_at: TIMESTAMP                                           │
└─────────────────────────────────────────────────────────────────────┘


┌─────────────────────────────────────────────────────────────────────┐
│                         REVIEWS                                     │
├─────────────────────────────────────────────────────────────────────┤
│ PK  id: INTEGER                                                     │
│ FK  user_id: INTEGER NOT NULL → users.id                            │
│ FK  tour_id: INTEGER NOT NULL → tours.id                            │
│     rating: INTEGER NOT NULL CHECK (rating BETWEEN 1 AND 5)         │
│     comment: TEXT                                                   │
│     created_at: TIMESTAMP                                           │
│     updated_at: TIMESTAMP                                           │
│     UNIQUE(user_id, tour_id)                                        │
└─────────────────────────────────────────────────────────────────────┘
         │                              │
         │ *                            │ *
         │                              │
         │ 1                            │ 1
         └──────────► USERS             └──────────► TOURS
```

## SQL для создания таблиц

### 1. Таблица USERS

```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_digest VARCHAR(255) NOT NULL,
    full_name VARCHAR(255) NOT NULL,
    phone VARCHAR(20),
    role VARCHAR(20) DEFAULT 'client' CHECK (role IN ('client', 'admin')),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);
```

### 2. Таблица CATEGORIES

```sql
CREATE TABLE categories (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_categories_name ON categories(name);
```

### 3. Таблица TOURS

```sql
CREATE TABLE tours (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    destination VARCHAR(255) NOT NULL,
    duration_days INTEGER NOT NULL CHECK (duration_days > 0),
    price DECIMAL(10,2) NOT NULL CHECK (price > 0),
    available_seats INTEGER NOT NULL CHECK (available_seats >= 0),
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    category_id INTEGER REFERENCES categories(id) ON DELETE SET NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT check_dates CHECK (end_date > start_date)
);

CREATE INDEX idx_tours_category ON tours(category_id);
CREATE INDEX idx_tours_destination ON tours(destination);
CREATE INDEX idx_tours_dates ON tours(start_date, end_date);
CREATE INDEX idx_tours_price ON tours(price);
```

### 4. Таблица BOOKINGS

```sql
CREATE TABLE bookings (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    tour_id INTEGER NOT NULL REFERENCES tours(id) ON DELETE CASCADE,
    booking_date TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    number_of_people INTEGER NOT NULL CHECK (number_of_people > 0),
    total_price DECIMAL(10,2) NOT NULL CHECK (total_price >= 0),
    status VARCHAR(20) DEFAULT 'pending' CHECK (status IN ('pending', 'confirmed', 'cancelled')),
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_bookings_user ON bookings(user_id);
CREATE INDEX idx_bookings_tour ON bookings(tour_id);
CREATE INDEX idx_bookings_status ON bookings(status);
CREATE INDEX idx_bookings_date ON bookings(booking_date);
```

### 5. Таблица REVIEWS

```sql
CREATE TABLE reviews (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    tour_id INTEGER NOT NULL REFERENCES tours(id) ON DELETE CASCADE,
    rating INTEGER NOT NULL CHECK (rating BETWEEN 1 AND 5),
    comment TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(user_id, tour_id)
);

CREATE INDEX idx_reviews_user ON reviews(user_id);
CREATE INDEX idx_reviews_tour ON reviews(tour_id);
CREATE INDEX idx_reviews_rating ON reviews(rating);
```

## Связи между таблицами

### One-to-Many (1:*)

1. **USERS → BOOKINGS**
   - Один пользователь может иметь много бронирований
   - `bookings.user_id` → `users.id`

2. **USERS → REVIEWS**
   - Один пользователь может оставить много отзывов
   - `reviews.user_id` → `users.id`

3. **TOURS → BOOKINGS**
   - Один тур может иметь много бронирований
   - `bookings.tour_id` → `tours.id`

4. **TOURS → REVIEWS**
   - Один тур может иметь много отзывов
   - `reviews.tour_id` → `tours.id`

5. **CATEGORIES → TOURS**
   - Одна категория может содержать много туров
   - `tours.category_id` → `categories.id`

### Ограничения уникальности

1. **USERS**: `email` - уникальный
2. **CATEGORIES**: `name` - уникальный
3. **REVIEWS**: `(user_id, tour_id)` - один отзыв на тур от пользователя

## Функциональные зависимости

### USERS
- `id` → `email, password_digest, full_name, phone, role`
- Все атрибуты зависят только от первичного ключа

### CATEGORIES
- `id` → `name, description`
- Все атрибуты зависят только от первичного ключа

### TOURS
- `id` → `title, description, destination, duration_days, price, available_seats, start_date, end_date, category_id`
- Все атрибуты зависят только от первичного ключа
- `category_id` - внешний ключ, не создает транзитивной зависимости

### BOOKINGS
- `id` → `user_id, tour_id, booking_date, number_of_people, total_price, status, notes`
- Все атрибуты зависят только от первичного ключа
- `total_price` вычисляется как `tour.price * number_of_people`, но хранится для исторических данных

### REVIEWS
- `id` → `user_id, tour_id, rating, comment`
- Все атрибуты зависят только от первичного ключа

## Обоснование решений по нормализации

### Почему 5 таблиц?

1. **USERS** - отдельная таблица для пользователей, избегаем дублирования данных пользователя в бронированиях
2. **CATEGORIES** - вынесены в отдельную таблицу, чтобы избежать дублирования названий категорий в турах
3. **TOURS** - основная сущность, содержит только данные о туре
4. **BOOKINGS** - связующая таблица между пользователями и турами, содержит данные о бронировании
5. **REVIEWS** - отзывы вынесены отдельно, чтобы не дублировать данные в турах или пользователях

### Почему total_price хранится в BOOKINGS?

Хотя `total_price` можно вычислить как `tour.price * number_of_people`, мы храним его явно по следующим причинам:
- **Исторические данные**: цена тура может измениться, но цена бронирования должна остаться прежней
- **Производительность**: не нужно делать JOIN для получения цены
- **Бизнес-логика**: могут быть скидки, промокоды и т.д.

Это не нарушает 3НФ, так как это не транзитивная зависимость, а бизнес-требование.

### Индексы

Индексы созданы для:
- Первичных ключей (автоматически)
- Внешних ключей (для JOIN операций)
- Полей, используемых в WHERE и ORDER BY (email, status, dates, price)
- Составных условий поиска

## Примеры запросов

### 1. Получить все туры с категориями

```sql
SELECT t.*, c.name as category_name
FROM tours t
LEFT JOIN categories c ON t.category_id = c.id
ORDER BY t.start_date;
```

### 2. Получить все бронирования пользователя с деталями тура

```sql
SELECT b.*, t.title, t.destination, t.start_date, t.end_date, u.full_name
FROM bookings b
JOIN tours t ON b.tour_id = t.id
JOIN users u ON b.user_id = u.id
WHERE b.user_id = ?
ORDER BY b.booking_date DESC;
```

### 3. Получить средний рейтинг тура

```sql
SELECT t.id, t.title, AVG(r.rating) as average_rating, COUNT(r.id) as reviews_count
FROM tours t
LEFT JOIN reviews r ON t.id = r.tour_id
GROUP BY t.id, t.title;
```

### 4. Отчет: Статистика по турам

```sql
SELECT 
    t.id,
    t.title,
    t.destination,
    c.name as category,
    COUNT(DISTINCT b.id) as bookings_count,
    SUM(b.total_price) as total_revenue,
    AVG(r.rating) as average_rating,
    t.available_seats,
    (SELECT SUM(number_of_people) FROM bookings WHERE tour_id = t.id AND status != 'cancelled') as booked_seats
FROM tours t
LEFT JOIN categories c ON t.category_id = c.id
LEFT JOIN bookings b ON t.id = b.tour_id AND b.status != 'cancelled'
LEFT JOIN reviews r ON t.id = r.tour_id
GROUP BY t.id, t.title, t.destination, c.name, t.available_seats
ORDER BY total_revenue DESC;
```

### 5. Отчет: Анализ клиентов

```sql
SELECT 
    u.id,
    u.full_name,
    u.email,
    COUNT(DISTINCT b.id) as bookings_count,
    SUM(b.total_price) as total_spent,
    AVG(r.rating) as average_rating_given,
    COUNT(DISTINCT r.id) as reviews_count
FROM users u
LEFT JOIN bookings b ON u.id = b.user_id AND b.status != 'cancelled'
LEFT JOIN reviews r ON u.id = r.user_id
WHERE u.role = 'client'
GROUP BY u.id, u.full_name, u.email
HAVING COUNT(DISTINCT b.id) > 0
ORDER BY total_spent DESC;
```

### 6. Отчет: Финансы по категориям

```sql
SELECT 
    c.id,
    c.name as category_name,
    COUNT(DISTINCT t.id) as tours_count,
    AVG(t.price) as average_tour_price,
    COUNT(DISTINCT b.id) as bookings_count,
    SUM(b.total_price) as total_revenue,
    SUM(b.number_of_people) as total_tourists
FROM categories c
LEFT JOIN tours t ON c.id = t.category_id
LEFT JOIN bookings b ON t.id = b.tour_id AND b.status != 'cancelled'
GROUP BY c.id, c.name
ORDER BY total_revenue DESC;
```

## Тестовые данные (seed)

```sql
-- Categories
INSERT INTO categories (name, description) VALUES
('Пляжный отдых', 'Туры на морские курорты'),
('Экскурсионные', 'Культурно-познавательные туры'),
('Горнолыжные', 'Туры на горнолыжные курорты'),
('Экзотические', 'Туры в экзотические страны');

-- Users
INSERT INTO users (email, password_digest, full_name, phone, role) VALUES
('admin@travel.com', '$2a$12$...', 'Администратор Системы', '+380501234567', 'admin'),
('ivan@example.com', '$2a$12$...', 'Иван Петров', '+380501111111', 'client'),
('maria@example.com', '$2a$12$...', 'Мария Сидорова', '+380502222222', 'client');

-- Tours
INSERT INTO tours (title, description, destination, duration_days, price, available_seats, start_date, end_date, category_id) VALUES
('Отдых в Турции', 'Пляжный отдых в Анталии', 'Анталия, Турция', 7, 15000.00, 20, '2024-06-01', '2024-06-08', 1),
('Экскурсия по Европе', 'Посещение 5 столиц', 'Европа', 10, 35000.00, 15, '2024-07-01', '2024-07-11', 2),
('Карпаты зимой', 'Горнолыжный курорт Буковель', 'Буковель, Украина', 5, 8000.00, 30, '2024-12-20', '2024-12-25', 3);

-- Bookings
INSERT INTO bookings (user_id, tour_id, booking_date, number_of_people, total_price, status, notes) VALUES
(2, 1, '2024-05-01 10:00:00', 2, 30000.00, 'confirmed', 'Семейная поездка'),
(3, 2, '2024-05-15 14:30:00', 1, 35000.00, 'pending', NULL);

-- Reviews
INSERT INTO reviews (user_id, tour_id, rating, comment) VALUES
(2, 1, 5, 'Отличный отдых! Все понравилось!');
```

## Миграции Rails

```ruby
# db/migrate/xxx_create_users.rb
class CreateUsers < ActiveRecord::Migration[7.0]
  def change
    create_table :users do |t|
      t.string :email, null: false
      t.string :password_digest, null: false
      t.string :full_name, null: false
      t.string :phone
      t.string :role, default: 'client'
      
      t.timestamps
    end
    
    add_index :users, :email, unique: true
    add_index :users, :role
  end
end

# db/migrate/xxx_create_categories.rb
class CreateCategories < ActiveRecord::Migration[7.0]
  def change
    create_table :categories do |t|
      t.string :name, null: false
      t.text :description
      
      t.timestamps
    end
    
    add_index :categories, :name, unique: true
  end
end

# db/migrate/xxx_create_tours.rb
class CreateTours < ActiveRecord::Migration[7.0]
  def change
    create_table :tours do |t|
      t.string :title, null: false
      t.text :description
      t.string :destination, null: false
      t.integer :duration_days, null: false
      t.decimal :price, precision: 10, scale: 2, null: false
      t.integer :available_seats, null: false
      t.date :start_date, null: false
      t.date :end_date, null: false
      t.references :category, foreign_key: true
      
      t.timestamps
    end
    
    add_index :tours, :destination
    add_index :tours, [:start_date, :end_date]
    add_index :tours, :price
  end
end

# db/migrate/xxx_create_bookings.rb
class CreateBookings < ActiveRecord::Migration[7.0]
  def change
    create_table :bookings do |t|
      t.references :user, null: false, foreign_key: true
      t.references :tour, null: false, foreign_key: true
      t.datetime :booking_date, null: false
      t.integer :number_of_people, null: false
      t.decimal :total_price, precision: 10, scale: 2, null: false
      t.string :status, default: 'pending'
      t.text :notes
      
      t.timestamps
    end
    
    add_index :bookings, :status
    add_index :bookings, :booking_date
  end
end

# db/migrate/xxx_create_reviews.rb
class CreateReviews < ActiveRecord::Migration[7.0]
  def change
    create_table :reviews do |t|
      t.references :user, null: false, foreign_key: true
      t.references :tour, null: false, foreign_key: true
      t.integer :rating, null: false
      t.text :comment
      
      t.timestamps
    end
    
    add_index :reviews, [:user_id, :tour_id], unique: true
    add_index :reviews, :rating
  end
end
```
