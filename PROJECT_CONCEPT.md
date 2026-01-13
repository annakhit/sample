# Концепция проекта: Туристическое агенство "TravelHub"

## 1. Описание предметной области

Веб-приложение для автоматизации работы туристического агенства с возможностью:
- Управления каталогом туров
- Регистрации и авторизации пользователей (клиенты и администраторы)
- Бронирования туров клиентами
- Управления бронированиями администраторами
- Формирования отчетов

## 2. Схема базы данных (3НФ)

### Таблица 1: users
```
id: integer (PK)
email: string (unique, not null)
password_digest: string (not null)
full_name: string (not null)
phone: string
role: enum ['client', 'admin'] (default: 'client')
created_at: datetime
updated_at: datetime
```

### Таблица 2: tours
```
id: integer (PK)
title: string (not null)
description: text
destination: string (not null)
duration_days: integer (not null)
price: decimal(10,2) (not null)
available_seats: integer (not null)
start_date: date (not null)
end_date: date (not null)
category_id: integer (FK -> categories.id)
created_at: datetime
updated_at: datetime
```

### Таблица 3: categories
```
id: integer (PK)
name: string (unique, not null)
description: text
created_at: datetime
updated_at: datetime
```

### Таблица 4: bookings
```
id: integer (PK)
user_id: integer (FK -> users.id, not null)
tour_id: integer (FK -> tours.id, not null)
booking_date: datetime (not null)
number_of_people: integer (not null)
total_price: decimal(10,2) (not null)
status: enum ['pending', 'confirmed', 'cancelled'] (default: 'pending')
notes: text
created_at: datetime
updated_at: datetime
```

### Таблица 5: reviews
```
id: integer (PK)
user_id: integer (FK -> users.id, not null)
tour_id: integer (FK -> tours.id, not null)
rating: integer (1-5, not null)
comment: text
created_at: datetime
updated_at: datetime
```

**Обоснование 3НФ:**
- Все атрибуты атомарны
- Отсутствуют частичные зависимости от составных ключей
- Отсутствуют транзитивные зависимости
- Категории вынесены в отдельную таблицу
- Связи через внешние ключи

## 3. Технический стек

### Backend (Ruby on Rails API)
- Ruby 3.x
- Rails 7.x (API mode)
- SQLite3
- JWT для аутентификации
- Rack-CORS для CORS

### Frontend (React)
- React 18.x
- React Router для навигации
- Axios для HTTP запросов
- Context API для state management
- CSS модули для стилизации

## 4. Архитектура приложения

### Backend структура
```
app/
├── controllers/
│   ├── api/v1/
│   │   ├── auth_controller.rb
│   │   ├── users_controller.rb
│   │   ├── tours_controller.rb
│   │   ├── categories_controller.rb
│   │   ├── bookings_controller.rb
│   │   ├── reviews_controller.rb
│   │   └── reports_controller.rb
├── models/
│   ├── user.rb
│   ├── tour.rb
│   ├── category.rb
│   ├── booking.rb
│   └── review.rb
├── services/
│   ├── auth_service.rb
│   ├── backup_service.rb
│   ├── report_service.rb
│   └── booking_service.rb
├── serializers/
├── validators/
└── exceptions/
```

### Frontend структура
```
src/
├── components/
│   ├── auth/
│   ├── tours/
│   ├── bookings/
│   ├── admin/
│   └── common/
├── pages/
├── services/
├── context/
├── hooks/
└── utils/
```

## 5. Основные требования проекта

### 5.1 CRUD операции
Все 5 сущностей должны иметь полный CRUD:
- Users: Create (регистрация), Read, Update, Delete
- Tours: Create, Read, Update, Delete (админ)
- Categories: Create, Read, Update, Delete (админ)
- Bookings: Create, Read, Update, Delete
- Reviews: Create, Read, Update, Delete

### 5.2 Валидация
**Users:**
- Email: формат email, уникальность
- Password: минимум 8 символов, наличие букв и цифр
- Full_name: обязательное, минимум 2 слова
- Phone: формат телефона

**Tours:**
- Title: обязательное, 5-200 символов
- Price: положительное число
- Available_seats: >= 0
- Duration_days: > 0
- Start_date: не в прошлом
- End_date: > start_date

**Bookings:**
- Number_of_people: > 0, <= available_seats
- Total_price: = tour.price * number_of_people
- Проверка доступности мест

**Reviews:**
- Rating: 1-5
- Один отзыв на тур от пользователя

### 5.3 Три содержательных отчета

**Отчет 1: Статистика по турам**
- Количество бронирований по каждому туру
- Средний рейтинг
- Общая выручка
- Процент заполненности
- Использует: tours, bookings, reviews

**Отчет 2: Анализ клиентов**
- Список клиентов с количеством бронирований
- Общая сумма потраченных средств
- Средний рейтинг оставленных отзывов
- Использует: users, bookings, reviews

**Отчет 3: Финансовый отчет по категориям**
- Выручка по категориям туров
- Количество туров в категории
- Средняя цена тура
- Самые популярные направления
- Использует: categories, tours, bookings

### 5.4 Обработка исключений

**Иерархия кастомных исключений:**
```ruby
class TravelHubError < StandardError; end

class AuthenticationError < TravelHubError; end
  class InvalidCredentialsError < AuthenticationError; end
  class TokenExpiredError < AuthenticationError; end
  class UnauthorizedError < AuthenticationError; end

class ValidationError < TravelHubError; end
  class BookingValidationError < ValidationError; end
  class TourValidationError < ValidationError; end

class ResourceError < TravelHubError; end
  class ResourceNotFoundError < ResourceError; end
  class ResourceConflictError < ResourceError; end

class BackupError < TravelHubError; end
  class BackupFileNotFoundError < BackupError; end
  class BackupRestoreError < BackupError; end
```

### 5.5 Backup и восстановление
- Экспорт всех данных в JSON/YAML
- Импорт данных из файла
- Fallback режим работы с файлом при отключении БД
- Использование паттерна **Strategy** для выбора источника данных (DB/File)

### 5.6 CTRL+Z (Undo функциональность)
- История изменений для критичных операций (создание/отмена бронирований)
- Использование паттерна **Command** для отмены операций
- Хранение последних N операций

### 5.7 Дополнительные паттерны

**Паттерн 1: Factory Method**
- Создание различных типов отчетов
- Генерация разных форматов экспорта (JSON/YAML/CSV)

## 6. API Endpoints

### Authentication
```
POST   /api/v1/auth/register
POST   /api/v1/auth/login
POST   /api/v1/auth/logout
GET    /api/v1/auth/me
```

### Users
```
GET    /api/v1/users
GET    /api/v1/users/:id
PUT    /api/v1/users/:id
DELETE /api/v1/users/:id
```

### Tours
```
GET    /api/v1/tours
GET    /api/v1/tours/:id
POST   /api/v1/tours (admin)
PUT    /api/v1/tours/:id (admin)
DELETE /api/v1/tours/:id (admin)
GET    /api/v1/tours/:id/reviews
```

### Categories
```
GET    /api/v1/categories
GET    /api/v1/categories/:id
POST   /api/v1/categories (admin)
PUT    /api/v1/categories/:id (admin)
DELETE /api/v1/categories/:id (admin)
GET    /api/v1/categories/:id/tours
```

### Bookings
```
GET    /api/v1/bookings
GET    /api/v1/bookings/:id
POST   /api/v1/bookings
PUT    /api/v1/bookings/:id
DELETE /api/v1/bookings/:id
PATCH  /api/v1/bookings/:id/status (admin)
```

### Reviews
```
GET    /api/v1/reviews
GET    /api/v1/reviews/:id
POST   /api/v1/reviews
PUT    /api/v1/reviews/:id
DELETE /api/v1/reviews/:id
```

### Reports
```
GET    /api/v1/reports/tours_statistics
GET    /api/v1/reports/clients_analysis
GET    /api/v1/reports/financial_by_category
```

### Backup
```
POST   /api/v1/backup/export
POST   /api/v1/backup/import
GET    /api/v1/backup/status
```

## 7. Frontend страницы

### Публичные
- Главная страница с каталогом туров
- Страница тура (детали)
- Регистрация
- Вход
- Страница категорий

### Для клиентов
- Личный кабинет
- Мои бронирования
- Создание бронирования
- Мои отзывы

### Для администраторов
- Панель управления
- Управление турами (CRUD)
- Управление категориями (CRUD)
- Управление пользователями
- Управление бронированиями
- Отчеты (3 вкладки)
- Backup/Restore

## 8. Диаграммы

### 8.1 Диаграмма классов (UML)
```
┌─────────────────┐
│     User        │
├─────────────────┤
│ - id            │
│ - email         │
│ - password      │
│ - full_name     │
│ - phone         │
│ - role          │
├─────────────────┤
│ + authenticate()│
│ + admin?()      │
└────────┬────────┘
         │1
         │
         │*
┌────────▼────────┐         ┌─────────────────┐
│    Booking      │*       1│      Tour       │
├─────────────────┤◄────────┤─────────────────┤
│ - id            │         │ - id            │
│ - user_id       │         │ - title         │
│ - tour_id       │         │ - description   │
│ - booking_date  │         │ - destination   │
│ - num_people    │         │ - duration_days │
│ - total_price   │         │ - price         │
│ - status        │         │ - available_seats│
├─────────────────┤         │ - start_date    │
│ + confirm()     │         │ - end_date      │
│ + cancel()      │         │ - category_id   │
│ + calculate()   │         ├─────────────────┤
└─────────────────┘         │ + available?()  │
                            │ + book_seats()  │
         │1                 └────────┬────────┘
         │                           │*
         │*                          │
┌────────▼────────┐                  │1
│     Review      │                  │
├─────────────────┤         ┌────────▼────────┐
│ - id            │         │    Category     │
│ - user_id       │         ├─────────────────┤
│ - tour_id       │         │ - id            │
│ - rating        │         │ - name          │
│ - comment       │         │ - description   │
├─────────────────┤         └─────────────────┘
│ + validate()    │
└─────────────────┘
```

### 8.2 Диаграмма последовательности: Создание бронирования
```
Client    Frontend    API          BookingController    BookingService    Tour    Booking    DB
  │          │         │                   │                   │           │        │        │
  │─Fill form─>│       │                   │                   │           │        │        │
  │          │─POST /bookings─>            │                   │           │        │        │
  │          │         │──create()──────>  │                   │           │        │        │
  │          │         │                   │──book_tour()───>  │           │        │        │
  │          │         │                   │                   │─find()──> │        │        │
  │          │         │                   │                   │           │─query─>│        │
  │          │         │                   │                   │<──tour────│        │        │
  │          │         │                   │                   │──check_availability()        │
  │          │         │                   │                   │──calculate_price()           │
  │          │         │                   │                   │──create()──────────>│        │
  │          │         │                   │                   │                     │─save─>│
  │          │         │                   │                   │──update_seats()──>  │        │
  │          │         │                   │                   │                     │─update>│
  │          │         │                   │<──booking─────────│                     │        │
  │          │         │<──201 Created─────│                   │                     │        │
  │          │<─booking data──             │                   │                     │        │
  │<─success─│         │                   │                   │                     │        │
```

### 8.3 Диаграмма последовательности: Отображение туров с категориями
```
Client    Frontend    API       ToursController    Tour    Category    DB
  │          │         │               │            │         │         │
  │─Open page─>│       │               │            │         │         │
  │          │─GET /tours─>            │            │         │         │
  │          │         │──index()────> │            │         │         │
  │          │         │               │─includes()─>│        │         │
  │          │         │               │            │─query with join──>│
  │          │         │               │            │<─tours with categories─┘
  │          │         │               │<─tours─────│         │         │
  │          │         │<──200 OK──────│            │         │         │
  │          │<─tours with categories──│            │         │         │
  │<─display─│         │               │            │         │         │
```

## 9. Дополнительные фичи

### 9.1 Поиск и фильтрация
- Поиск туров по названию, направлению
- Фильтрация по категории, цене, датам
- Сортировка по цене, рейтингу, дате

### 9.2 Валидация с просмотром по нескольким сущностям
- При создании бронирования: проверка существования тура, пользователя, доступности мест
- При создании отзыва: проверка, что пользователь забронировал этот тур
- При удалении категории: проверка наличия связанных туров

## 10. Git workflow

- Main branch (защищенная)
- Develop branch
- Feature branches: feature/task-name
- Pull requests с code review
- Conventional commits
