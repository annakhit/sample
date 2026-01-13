# Распределение задач между студентами

## Принципы распределения

1. **Равномерность**: Каждый студент получает примерно равный объем работы
2. **Баланс Backend/Frontend**: Каждый работает и с Ruby on Rails, и с React
3. **Логическая связность**: Задачи одного студента логически связаны
4. **Независимость**: Студенты могут работать параллельно с минимальными конфликтами
5. **Полнота**: Все требования проекта покрыты

---

## СТУДЕНТ 1: Аутентификация и пользователи

### Backend задачи (Ruby on Rails)

#### Задача 1.1: Настройка проекта и модель User

- [ ] Создать Rails API проект
- [ ] Настроить SQLite3
- [ ] Настроить CORS
- [ ] Создать миграцию для таблицы `users`
- [ ] Создать модель `User` с валидациями:
  - Email: формат, уникальность
  - Password: минимум 8 символов, bcrypt
  - Full_name: обязательное, минимум 2 слова
  - Phone: формат телефона
  - Role: enum ['client', 'admin']
- [ ] Добавить метод `authenticate` для проверки пароля
- [ ] Добавить метод `admin?` для проверки роли

**Паттерны проектирования:**
- **Ruby**: Active Record (ORM паттерн), Template Method (для валидаций)
- **Применение**: Использование встроенных валидаций Rails как примера Template Method

**Файлы:**
- `db/migrate/xxx_create_users.rb`
- `app/models/user.rb`

#### Задача 1.2: Аутентификация через JWT

- [ ] Установить gem `jwt`
- [ ] Создать `AuthService` для работы с JWT:
  - Метод `encode(user_id)` - создание токена
  - Метод `decode(token)` - декодирование токена
  - Обработка истекших токенов
- [ ] Создать `AuthController` с endpoints:
  - `POST /api/v1/auth/register` - регистрация
  - `POST /api/v1/auth/login` - вход
  - `GET /api/v1/auth/me` - текущий пользователь
- [ ] Создать `AuthenticationConcern` для проверки токена:
  - Метод `authenticate_request!`
  - Метод `current_user`
  - Метод `require_admin!`
- [ ] Создать иерархию исключений для аутентификации:
  ```ruby
  class AuthenticationError < StandardError; end
  class InvalidCredentialsError < AuthenticationError; end
  class TokenExpiredError < AuthenticationError; end
  class UnauthorizedError < AuthenticationError; end
  ```
- [ ] Обработать исключения в `ApplicationController`

**Паттерны проектирования:**
- **Ruby**: Service Object (AuthService), Concern (модуль для контроллеров), Exception Hierarchy
- **Применение**: AuthService инкапсулирует логику JWT, Concern для переиспользования кода в контроллерах

**Файлы:**
- `app/services/auth_service.rb`
- `app/controllers/api/v1/auth_controller.rb`
- `app/controllers/concerns/authentication_concern.rb`
- `app/exceptions/authentication_errors.rb`

#### Задача 1.3: CRUD для Users (админ)

- [ ] Создать `UsersController` с endpoints:
  - `GET /api/v1/users` - список (только админ)
  - `GET /api/v1/users/:id` - один пользователь
  - `PUT /api/v1/users/:id` - обновление (сам или админ)
  - `DELETE /api/v1/users/:id` - удаление (только админ)
- [ ] Добавить проверки прав доступа
- [ ] Создать `UserSerializer` для JSON ответов

**Паттерны проектирования:**
- **Ruby**: Serializer (для представления данных), Decorator (через Concern для авторизации)
- **Применение**: UserSerializer для форматирования JSON, использование before_action для проверки прав

**Файлы:**
- `app/controllers/api/v1/users_controller.rb`
- `app/serializers/user_serializer.rb`

### Frontend задачи (React)

#### Задача 1.4: Настройка React проекта и аутентификация

- [ ] Создать React проект с Vite
- [ ] Установить зависимости: axios, react-router-dom
- [ ] Настроить CSS модули
- [ ] Создать `AuthContext` для управления состоянием аутентификации:
  - State: user, token, loading
  - Methods: login, register, logout, checkAuth
- [ ] Создать `api/authService.js` для HTTP запросов:
  - `register(userData)`
  - `login(credentials)`
  - `getCurrentUser()`
- [ ] Настроить axios interceptor для добавления токена
- [ ] Создать `ProtectedRoute` компонент
- [ ] Создать `AdminRoute` компонент

**Паттерны проектирования:**
- **React**: Context API (для глобального состояния), Higher-Order Component (для защищенных маршрутов), Service Layer (authService)
- **Применение**: AuthContext как Provider паттерн, ProtectedRoute как HOC для защиты маршрутов

**Файлы:**
- `src/context/AuthContext.jsx`
- `src/services/api/authService.js`
- `src/services/api/axios.js`
- `src/components/auth/ProtectedRoute.jsx`
- `src/components/auth/AdminRoute.jsx`

#### Задача 1.5: Страницы регистрации и входа

- [ ] Создать компонент `LoginPage`:
  - Форма с email и password
  - Валидация на клиенте
  - Обработка ошибок
  - Редирект после входа
- [ ] Создать компонент `RegisterPage`:
  - Форма с всеми полями
  - Валидация (email формат, пароль 8+ символов, телефон)
  - Подтверждение пароля
  - Обработка ошибок
  - Редирект после регистрации
- [ ] Создать компонент `UserProfile`:
  - Отображение данных пользователя
  - Форма редактирования профиля
  - Кнопка выхода
- [ ] Стилизация с CSS модулями (современный дизайн)
- [ ] Адаптивная верстка

**Паттерны проектирования:**
- **React**: Controlled Components (для форм), Custom Hooks (для валидации), Composition (разделение на компоненты)
- **Применение**: Формы как контролируемые компоненты, переиспользуемые компоненты форм

**Файлы:**
- `src/pages/auth/LoginPage.jsx`
- `src/pages/auth/LoginPage.module.css`
- `src/pages/auth/RegisterPage.jsx`
- `src/pages/auth/RegisterPage.module.css`
- `src/pages/user/UserProfile.jsx`
- `src/pages/user/UserProfile.module.css`
- `src/components/auth/LoginForm.jsx`
- `src/components/auth/LoginForm.module.css`
- `src/components/auth/RegisterForm.jsx`
- `src/components/auth/RegisterForm.module.css`

---

## СТУДЕНТ 2: Категории и туры

### Backend задачи (Ruby on Rails)

#### Задача 2.1: Модели Category и Tour

- [ ] Создать миграцию для таблицы `categories`
- [ ] Создать модель `Category` с валидациями:
  - Name: обязательное, уникальное, 2-50 символов
  - Description: опциональное
- [ ] Создать миграцию для таблицы `tours`
- [ ] Создать модель `Tour` с валидациями:
  - Title: обязательное, 5-200 символов
  - Price: положительное число
  - Available_seats: >= 0
  - Duration_days: > 0
  - Start_date: не в прошлом
  - End_date: > start_date
  - Category: обязательная связь
- [ ] Настроить ассоциации:
  - `Category has_many :tours`
  - `Tour belongs_to :category`
- [ ] Добавить методы в Tour:
  - `available?` - проверка доступности
  - `book_seats(count)` - бронирование мест
  - `release_seats(count)` - освобождение мест

**Паттерны проектирования:**
- **Ruby**: Active Record Associations (One-to-Many), Query Object (для сложных запросов), Scope (для фильтрации)
- **Применение**: Использование has_many/belongs_to, создание scope для популярных запросов (например, available_tours)

**Файлы:**
- `db/migrate/xxx_create_categories.rb`
- `db/migrate/xxx_create_tours.rb`
- `app/models/category.rb`
- `app/models/tour.rb`

#### Задача 2.2: CRUD для Categories

- [ ] Создать `CategoriesController` с endpoints:
  - `GET /api/v1/categories` - список всех категорий
  - `GET /api/v1/categories/:id` - одна категория
  - `POST /api/v1/categories` - создание (только админ)
  - `PUT /api/v1/categories/:id` - обновление (только админ)
  - `DELETE /api/v1/categories/:id` - удаление (только админ)
  - `GET /api/v1/categories/:id/tours` - туры категории
- [ ] Добавить валидацию при удалении (проверка связанных туров)
- [ ] Создать `CategorySerializer`

**Паттерны проектирования:**
- **Ruby**: RESTful Controller, Serializer, Before Action (для авторизации)
- **Применение**: Стандартный REST API, использование before_action для DRY кода

**Файлы:**
- `app/controllers/api/v1/categories_controller.rb`
- `app/serializers/category_serializer.rb`

#### Задача 2.3: CRUD для Tours

- [ ] Создать `ToursController` с endpoints:
  - `GET /api/v1/tours` - список с фильтрацией и поиском
  - `GET /api/v1/tours/:id` - один тур с категорией
  - `POST /api/v1/tours` - создание (только админ)
  - `PUT /api/v1/tours/:id` - обновление (только админ)
  - `DELETE /api/v1/tours/:id` - удаление (только админ)
- [ ] Реализовать фильтрацию:
  - По категории
  - По диапазону цен
  - По датам
  - Поиск по названию и направлению
- [ ] Реализовать сортировку (цена, дата, рейтинг)
- [ ] Использовать eager loading для категорий (`includes(:category)`)
- [ ] Создать `TourSerializer` с вложенной категорией
- [ ] Создать исключения для туров:
  ```ruby
  class TourValidationError < ValidationError; end
  class TourNotAvailableError < TourValidationError; end
  ```

**Паттерны проектирования:**
- **Ruby**: Query Object (для сложной фильтрации), Eager Loading (N+1 оптимизация), Builder (для построения запросов)
- **Применение**: Создание отдельного класса TourQuery для фильтрации, использование includes для оптимизации

**Файлы:**
- `app/controllers/api/v1/tours_controller.rb`
- `app/serializers/tour_serializer.rb`
- `app/exceptions/tour_errors.rb`

### Frontend задачи (React)

#### Задача 2.4: Сервисы и компоненты для категорий и туров

- [ ] Создать `api/categoryService.js`:
  - `getCategories()`
  - `getCategory(id)`
  - `createCategory(data)` (админ)
  - `updateCategory(id, data)` (админ)
  - `deleteCategory(id)` (админ)
- [ ] Создать `api/tourService.js`:
  - `getTours(filters)`
  - `getTour(id)`
  - `createTour(data)` (админ)
  - `updateTour(id, data)` (админ)
  - `deleteTour(id)` (админ)
- [ ] Создать `TourCard` компонент для отображения тура
- [ ] Создать `CategoryBadge` компонент
- [ ] Создать `SearchBar` компонент
- [ ] Создать `FilterPanel` компонент

**Паттерны проектирования:**
- **React**: Service Layer (API сервисы), Presentational Components (TourCard, CategoryBadge), Container/Presenter
- **Применение**: Разделение логики API и UI компонентов, переиспользуемые презентационные компоненты

**Файлы:**
- `src/services/api/categoryService.js`
- `src/services/api/tourService.js`
- `src/components/tours/TourCard.jsx`
- `src/components/tours/TourCard.module.css`
- `src/components/categories/CategoryBadge.jsx`
- `src/components/categories/CategoryBadge.module.css`
- `src/components/common/SearchBar.jsx`
- `src/components/common/SearchBar.module.css`
- `src/components/common/FilterPanel.jsx`
- `src/components/common/FilterPanel.module.css`

#### Задача 2.5: Страницы каталога и деталей тура

- [ ] Создать `ToursPage` (главная страница):
  - Отображение списка туров в виде карточек
  - Поиск по названию
  - Фильтрация по категории, цене, датам
  - Сортировка
  - Пагинация
  - Адаптивная сетка (grid)
- [ ] Создать `TourDetailPage`:
  - Полная информация о туре
  - Отображение категории
  - Кнопка "Забронировать"
  - Список отзывов (интеграция со Студентом 4)
  - Breadcrumbs навигация
- [ ] Создать `CategoriesPage`:
  - Список всех категорий
  - Количество туров в каждой
  - Переход к турам категории
- [ ] Стилизация с CSS модулями
- [ ] Loading states и error handling

**Паттерны проектирования:**
- **React**: Custom Hooks (для фильтрации, пагинации), Compound Components (для сложных UI), State Management
- **Применение**: useFilter, usePagination хуки, композиция компонентов для гибкости

**Файлы:**
- `src/pages/tours/ToursPage.jsx`
- `src/pages/tours/ToursPage.module.css`
- `src/pages/tours/TourDetailPage.jsx`
- `src/pages/tours/TourDetailPage.module.css`
- `src/pages/categories/CategoriesPage.jsx`
- `src/pages/categories/CategoriesPage.module.css`
- `src/components/tours/TourList.jsx`
- `src/components/tours/TourList.module.css`
- `src/components/tours/TourFilters.jsx`
- `src/components/tours/TourFilters.module.css`

#### Задача 2.6: Админ панель для туров и категорий

- [ ] Создать `AdminToursPage`:
  - Таблица всех туров
  - Кнопки редактирования и удаления
  - Кнопка создания нового тура
  - Поиск и фильтрация
- [ ] Создать `TourFormPage` (создание/редактирование):
  - Форма со всеми полями
  - Выбор категории из dropdown
  - Валидация
  - Обработка ошибок
- [ ] Создать `AdminCategoriesPage`:
  - Список категорий
  - CRUD операции
  - Модальное окно для создания/редактирования
- [ ] Подтверждение удаления (modal)
- [ ] Toast уведомления

**Паттерны проектирования:**
- **React**: Modal Pattern (для диалогов), Form Validation Pattern, Toast/Notification Pattern
- **Применение**: Переиспользуемый Modal компонент, централизованная система уведомлений

**Файлы:**
- `src/pages/admin/AdminToursPage.jsx`
- `src/pages/admin/AdminToursPage.module.css`
- `src/pages/admin/TourFormPage.jsx`
- `src/pages/admin/TourFormPage.module.css`
- `src/pages/admin/AdminCategoriesPage.jsx`
- `src/pages/admin/AdminCategoriesPage.module.css`
- `src/components/admin/TourTable.jsx`
- `src/components/admin/TourTable.module.css`
- `src/components/admin/CategoryModal.jsx`
- `src/components/admin/CategoryModal.module.css`
- `src/components/common/ConfirmModal.jsx`
- `src/components/common/ConfirmModal.module.css`

---

## СТУДЕНТ 3: Бронирования и отчеты

### Backend задачи (Ruby on Rails)

#### Задача 3.1: Модель Booking и сервис бронирования

- [ ] Создать миграцию для таблицы `bookings`
- [ ] Создать модель `Booking` с валидациями:
  - User и Tour: обязательные связи
  - Number_of_people: > 0
  - Total_price: положительное число
  - Status: enum ['pending', 'confirmed', 'cancelled']
  - Кастомная валидация: number_of_people <= tour.available_seats
- [ ] Настроить ассоциации:
  - `Booking belongs_to :user`
  - `Booking belongs_to :tour`
  - `User has_many :bookings`
  - `Tour has_many :bookings`
- [ ] Создать `BookingService` с методами:
  - `create_booking(user, tour, params)`:
    - Проверка доступности мест
    - Расчет total_price
    - Создание бронирования в транзакции
    - Уменьшение available_seats
  - `cancel_booking(booking)`:
    - Изменение статуса
    - Возврат мест
  - `confirm_booking(booking)` (админ)
- [ ] Создать исключения:
  ```ruby
  class BookingValidationError < ValidationError; end
  class InsufficientSeatsError < BookingValidationError; end
  class BookingNotCancellableError < BookingValidationError; end
  ```

**Паттерны проектирования:**
- **Ruby**: Service Object (BookingService), Transaction Script (для атомарных операций), State Machine (для статусов)
- **Применение**: BookingService инкапсулирует бизнес-логику, использование транзакций для целостности данных

**Файлы:**
- `db/migrate/xxx_create_bookings.rb`
- `app/models/booking.rb`
- `app/services/booking_service.rb`
- `app/exceptions/booking_errors.rb`

#### Задача 3.2: CRUD для Bookings

- [ ] Создать `BookingsController` с endpoints:
  - `GET /api/v1/bookings` - список (свои для клиента, все для админа)
  - `GET /api/v1/bookings/:id` - одно бронирование
  - `POST /api/v1/bookings` - создание
  - `PUT /api/v1/bookings/:id` - обновление (ограниченное)
  - `DELETE /api/v1/bookings/:id` - отмена бронирования
  - `PATCH /api/v1/bookings/:id/status` - изменение статуса (только админ)
- [ ] Использовать `BookingService` для создания/отмены
- [ ] Добавить проверки прав доступа
- [ ] Eager loading для user и tour
- [ ] Создать `BookingSerializer` с вложенными user и tour

**Паттерны проектирования:**
- **Ruby**: Policy Object (для авторизации), Eager Loading, Serializer with Associations
- **Применение**: Отдельный класс для проверки прав доступа, оптимизация запросов

**Файлы:**
- `app/controllers/api/v1/bookings_controller.rb`
- `app/serializers/booking_serializer.rb`

#### Задача 3.3: Три отчета

- [ ] Создать `ReportService` с методами:
  - `tours_statistics`:
    - Количество бронирований по турам
    - Средний рейтинг (join с reviews)
    - Общая выручка
    - Процент заполненности
    - SQL: JOIN tours, bookings, reviews + GROUP BY
  - `clients_analysis`:
    - Список клиентов с количеством бронирований
    - Общая сумма потраченных средств
    - Средний рейтинг отзывов
    - SQL: JOIN users, bookings, reviews + GROUP BY
  - `financial_by_category`:
    - Выручка по категориям
    - Количество туров в категории
    - Средняя цена тура
    - Самые популярные направления
    - SQL: JOIN categories, tours, bookings + GROUP BY
- [ ] Создать `ReportsController` с endpoints:
  - `GET /api/v1/reports/tours_statistics`
  - `GET /api/v1/reports/clients_analysis`
  - `GET /api/v1/reports/financial_by_category`
- [ ] Все отчеты доступны только админам
- [ ] Добавить параметры фильтрации (даты, категории)
- [ ] Оптимизировать SQL запросы

**Паттерны проектирования:**
- **Ruby**: Query Object (для сложных отчетов), Value Object (для результатов отчетов), Repository (для доступа к данным)
- **Применение**: Отдельные классы для каждого типа отчета, инкапсуляция SQL логики

**Файлы:**
- `app/services/report_service.rb`
- `app/controllers/api/v1/reports_controller.rb`

### Frontend задачи (React)

#### Задача 3.4: Бронирования для клиентов

- [ ] Создать `api/bookingService.js`:
  - `getBookings()`
  - `getBooking(id)`
  - `createBooking(data)`
  - `cancelBooking(id)`
- [ ] Создать `BookingFormModal` компонент:
  - Выбор количества человек
  - Расчет итоговой цены
  - Поле для заметок
  - Валидация (не больше доступных мест)
  - Подтверждение бронирования
- [ ] Создать `MyBookingsPage`:
  - Список бронирований пользователя
  - Фильтрация по статусу
  - Отображение деталей тура
  - Кнопка отмены (для pending)
  - Статус badges (pending/confirmed/cancelled)
- [ ] Создать `BookingDetailPage`:
  - Полная информация о бронировании
  - Детали тура
  - Кнопка отмены
- [ ] Интеграция с `TourDetailPage` (кнопка "Забронировать")

**Паттерны проектирования:**
- **React**: Modal Pattern, Custom Hooks (useBooking), Computed Properties (для расчета цены), Status Badge Pattern
- **Применение**: Переиспользуемый Modal, хук для управления бронированиями, динамический расчет цены

**Файлы:**
- `src/services/api/bookingService.js`
- `src/pages/bookings/MyBookingsPage.jsx`
- `src/pages/bookings/MyBookingsPage.module.css`
- `src/pages/bookings/BookingDetailPage.jsx`
- `src/pages/bookings/BookingDetailPage.module.css`
- `src/components/bookings/BookingFormModal.jsx`
- `src/components/bookings/BookingFormModal.module.css`
- `src/components/bookings/BookingCard.jsx`
- `src/components/bookings/BookingCard.module.css`

#### Задача 3.5: Управление бронированиями (админ)

- [ ] Создать `AdminBookingsPage`:
  - Таблица всех бронирований
  - Фильтрация по статусу, дате, пользователю
  - Поиск по имени клиента или туру
  - Кнопки изменения статуса
  - Отображение клиента и тура
- [ ] Создать `BookingStatusModal`:
  - Изменение статуса бронирования
  - Подтверждение действия
- [ ] Цветовая индикация статусов
- [ ] Экспорт списка бронирований (CSV)

**Паттерны проектирования:**
- **React**: Table Component Pattern, Filter Pattern, Export Pattern
- **Применение**: Переиспользуемый компонент таблицы, система фильтров, экспорт данных

**Файлы:**
- `src/pages/admin/AdminBookingsPage.jsx`
- `src/pages/admin/AdminBookingsPage.module.css`
- `src/components/admin/BookingTable.jsx`
- `src/components/admin/BookingTable.module.css`
- `src/components/admin/BookingStatusModal.jsx`
- `src/components/admin/BookingStatusModal.module.css`

#### Задача 3.6: Страница отчетов (админ)

- [ ] Создать `api/reportService.js`:
  - `getToursStatistics(filters)`
  - `getClientsAnalysis(filters)`
  - `getFinancialByCategory(filters)`
- [ ] Создать `ReportsPage` с тремя вкладками:
  - Вкладка 1: Статистика по турам
  - Вкладка 2: Анализ клиентов
  - Вкладка 3: Финансовый отчет по категориям
- [ ] Для каждой вкладки:
  - Таблица с данными
  - Фильтры (даты, категории)
  - Сортировка
  - Итоговые суммы
  - Экспорт в CSV
- [ ] Loading states
- [ ] Адаптивный дизайн таблиц

**Паттерны проектирования:**
- **React**: Tabs Pattern, Data Table Pattern, Custom Hooks (useReport), Export Hook
- **Применение**: Компонент вкладок, переиспользуемая таблица данных, хуки для работы с отчетами

**Файлы:**
- `src/services/api/reportService.js`
- `src/pages/admin/ReportsPage.jsx`
- `src/pages/admin/ReportsPage.module.css`
- `src/components/reports/ToursStatisticsTab.jsx`
- `src/components/reports/ToursStatisticsTab.module.css`
- `src/components/reports/ClientsAnalysisTab.jsx`
- `src/components/reports/ClientsAnalysisTab.module.css`
- `src/components/reports/FinancialByCategoryTab.jsx`
- `src/components/reports/FinancialByCategoryTab.module.css`
- `src/components/common/DataTable.jsx`
- `src/components/common/DataTable.module.css`

---

## СТУДЕНТ 4: Отзывы, Backup/Restore, паттерны и дополнительные фичи

### Backend задачи (Ruby on Rails)

#### Задача 4.1: Модель Review и CRUD

- [ ] Создать миграцию для таблицы `reviews`
- [ ] Создать модель `Review` с валидациями:
  - User и Tour: обязательные связи
  - Rating: 1-5
  - Comment: опциональное
  - Уникальность: один отзыв на тур от пользователя
  - Кастомная валидация: пользователь должен иметь бронирование этого тура
- [ ] Настроить ассоциации:
  - `Review belongs_to :user`
  - `Review belongs_to :tour`
  - `User has_many :reviews`
  - `Tour has_many :reviews`
- [ ] Добавить метод в Tour: `average_rating`
- [ ] Создать `ReviewsController` с endpoints:
  - `GET /api/v1/reviews` - список
  - `GET /api/v1/reviews/:id` - один отзыв
  - `POST /api/v1/reviews` - создание
  - `PUT /api/v1/reviews/:id` - обновление (только свой)
  - `DELETE /api/v1/reviews/:id` - удаление (свой или админ)
  - `GET /api/v1/tours/:id/reviews` - отзывы тура
- [ ] Создать `ReviewSerializer`

**Паттерны проектирования:**
- **Ruby**: Active Record Validations, Callback (для обновления рейтинга), Aggregation (для average_rating)
- **Применение**: Кастомная валидация, использование after_save для обновления среднего рейтинга тура

**Файлы:**
- `db/migrate/xxx_create_reviews.rb`
- `app/models/review.rb`
- `app/controllers/api/v1/reviews_controller.rb`
- `app/serializers/review_serializer.rb`

#### Задача 4.2: Backup и Restore с паттерном Strategy

- [ ] Создать `BackupService` с методами:
  - `export_to_json(filename)` - экспорт всех данных
  - `export_to_yaml(filename)` - экспорт в YAML
  - `import_from_file(filename)` - импорт данных
  - `restore_from_backup(filename)` - восстановление
- [ ] Реализовать паттерн **Strategy** для источника данных:
  ```ruby
  class DataSourceStrategy
    def find_all(model); end
    def find_by_id(model, id); end
    def create(model, attributes); end
    def update(record, attributes); end
    def destroy(record); end
  end
  
  class DatabaseSource < DataSourceStrategy
    # Работа с ActiveRecord
  end
  
  class FileSource < DataSourceStrategy
    # Работа с JSON/YAML файлом
  end
  
  class DataSourceContext
    def initialize(strategy)
      @strategy = strategy
    end
    
    def switch_strategy(strategy)
      @strategy = strategy
    end
  end
  ```
- [ ] Создать `BackupController` с endpoints:
  - `POST /api/v1/backup/export` - экспорт (JSON/YAML)
  - `POST /api/v1/backup/import` - импорт
  - `GET /api/v1/backup/status` - статус бекапа
  - `POST /api/v1/backup/switch_source` - переключение источника
- [ ] Обработка ошибок:
  ```ruby
  class BackupError < TravelHubError; end
  class BackupFileNotFoundError < BackupError; end
  class BackupRestoreError < BackupError; end
  ```
- [ ] Fallback на файл при недоступности БД

**Паттерны проектирования:**
- **Ruby**: **Strategy Pattern** (выбор источника данных), Adapter (для разных форматов), Template Method (для экспорта)
- **Применение**: DataSourceStrategy для переключения между БД и файлами, адаптеры для JSON/YAML

**Файлы:**
- `app/services/backup_service.rb`
- `app/services/data_source_strategy.rb`
- `app/services/database_source.rb`
- `app/services/file_source.rb`
- `app/services/data_source_context.rb`
- `app/controllers/api/v1/backup_controller.rb`
- `app/exceptions/backup_errors.rb`

#### Задача 4.3: CTRL+Z с паттерном Command

- [ ] Реализовать паттерн **Command** для отмены операций:
  ```ruby
  class Command
    def execute; end
    def undo; end
  end
  
  class CreateBookingCommand < Command
    def initialize(booking_params)
      @booking_params = booking_params
    end
    
    def execute
      @booking = BookingService.create_booking(@booking_params)
    end
    
    def undo
      BookingService.cancel_booking(@booking)
    end
  end
  
  class CancelBookingCommand < Command
    def execute
      # Отмена бронирования
    end
    
    def undo
      # Восстановление бронирования
    end
  end
  
  class CommandHistory
    def initialize
      @history = []
    end
    
    def execute_command(command)
      command.execute
      @history << command
    end
    
    def undo_last
      command = @history.pop
      command.undo if command
    end
  end
  ```
- [ ] Создать `UndoController`:
  - `POST /api/v1/undo` - отмена последней операции
  - `GET /api/v1/undo/history` - история операций
- [ ] Хранить историю в сессии
- [ ] Ограничить историю последними 10 операциями

**Паттерны проектирования:**
- **Ruby**: **Command Pattern** (для отмены операций), Memento (для сохранения состояния), History/Stack (для хранения команд)
- **Применение**: Каждая операция как команда с методами execute/undo, стек для истории

**Файлы:**
- `app/commands/command.rb`
- `app/commands/create_booking_command.rb`
- `app/commands/cancel_booking_command.rb`
- `app/commands/command_history.rb`
- `app/controllers/api/v1/undo_controller.rb`

#### Задача 4.4: Паттерн Factory для отчетов

- [ ] Реализовать паттерн **Factory Method** для создания отчетов:
  ```ruby
  class ReportFactory
    def self.create_report(type, params = {})
      case type
      when :tours_statistics
        ToursStatisticsReport.new(params)
      when :clients_analysis
        ClientsAnalysisReport.new(params)
      when :financial_by_category
        FinancialByCategoryReport.new(params)
      when :custom
        CustomReport.new(params)
      end
    end
  end
  
  class Report
    def generate; end
    def export(format); end
  end
  
  class ToursStatisticsReport < Report
    def generate
      # Генерация отчета
    end
    
    def export(format)
      case format
      when :json
        to_json
      when :csv
        to_csv
      end
    end
  end
  ```
- [ ] Добавить экспорт отчетов в разных форматах (JSON, CSV)
- [ ] Обновить `ReportsController` для использования фабрики

**Паттерны проектирования:**
- **Ruby**: **Factory Method Pattern** (создание отчетов), Template Method (для экспорта), Polymorphism (разные типы отчетов)
- **Применение**: ReportFactory для создания нужного типа отчета, общий интерфейс для всех отчетов

**Файлы:**
- `app/factories/report_factory.rb`
- `app/reports/report.rb`
- `app/reports/tours_statistics_report.rb`
- `app/reports/clients_analysis_report.rb`
- `app/reports/financial_by_category_report.rb`

### Frontend задачи (React)

#### Задача 4.5: Отзывы на турах

- [ ] Создать `api/reviewService.js`:
  - `getReviews(tourId)`
  - `createReview(data)`
  - `updateReview(id, data)`
  - `deleteReview(id)`
- [ ] Создать `ReviewList` компонент:
  - Список отзывов с рейтингом
  - Отображение автора и даты
  - Кнопки редактирования/удаления (для своих)
- [ ] Создать `ReviewForm` компонент:
  - Выбор рейтинга (звездочки)
  - Текстовое поле для комментария
  - Валидация
- [ ] Интегрировать в `TourDetailPage`:
  - Средний рейтинг тура
  - Список отзывов
  - Форма добавления отзыва (если забронировал)
- [ ] Создать `MyReviewsPage` для пользователя

**Паттерны проектирования:**
- **React**: Compound Components (ReviewList + ReviewForm), Controlled Components (для формы), Rating Component Pattern
- **Применение**: Композиция компонентов отзывов, переиспользуемый компонент рейтинга

**Файлы:**
- `src/services/api/reviewService.js`
- `src/components/reviews/ReviewList.jsx`
- `src/components/reviews/ReviewList.module.css`
- `src/components/reviews/ReviewForm.jsx`
- `src/components/reviews/ReviewForm.module.css`
- `src/components/reviews/ReviewCard.jsx`
- `src/components/reviews/ReviewCard.module.css`
- `src/components/reviews/StarRating.jsx`
- `src/components/reviews/StarRating.module.css`
- `src/pages/user/MyReviewsPage.jsx`
- `src/pages/user/MyReviewsPage.module.css`

#### Задача 4.6: Backup/Restore интерфейс (админ)

- [ ] Создать `api/backupService.js`:
  - `exportBackup(format)`
  - `importBackup(file)`
  - `getBackupStatus()`
  - `switchDataSource(source)`
- [ ] Создать `BackupPage` (админ):
  - Кнопка экспорта (выбор формата: JSON/YAML)
  - Загрузка файла для импорта
  - Отображение статуса бекапа
  - Переключение источника данных (DB/File)
  - История бекапов
- [ ] Индикатор текущего источника данных
- [ ] Подтверждение опасных операций
- [ ] Progress bar для импорта/экспорта

**Паттерны проектирования:**
- **React**: File Upload Pattern, Progress Indicator Pattern, Confirmation Dialog Pattern
- **Применение**: Компонент загрузки файлов, индикатор прогресса, модальные окна подтверждения

**Файлы:**
- `src/services/api/backupService.js`
- `src/pages/admin/BackupPage.jsx`
- `src/pages/admin/BackupPage.module.css`
- `src/components/admin/BackupPanel.jsx`
- `src/components/admin/BackupPanel.module.css`
- `src/components/admin/DataSourceSwitch.jsx`
- `src/components/admin/DataSourceSwitch.module.css`

#### Задача 4.7: Undo функциональность

- [ ] Создать `api/undoService.js`:
  - `undoLastAction()`
  - `getUndoHistory()`
- [ ] Создать `UndoButton` компонент:
  - Кнопка Ctrl+Z в header
  - Показ последней операции
  - Disabled если нет истории
- [ ] Добавить keyboard shortcut (Ctrl+Z)
- [ ] История операций в dropdown

**Паттерны проектирования:**
- **React**: Custom Hook (useUndo, useKeyboardShortcut), Command Pattern (на клиенте), Event Handler Pattern
- **Применение**: Хук для управления undo, обработка клавиатурных событий

**Файлы:**
- `src/services/api/undoService.js`
- `src/components/common/UndoButton.jsx`
- `src/components/common/UndoButton.module.css`
- `src/hooks/useUndo.js`
- `src/hooks/useKeyboardShortcut.js`

#### Задача 4.8: Навигация и общий layout

- [ ] Создать `Layout` компонент:
  - Header с навигацией
  - Sidebar (для админа)
  - Footer
  - Breadcrumbs
- [ ] Создать `Navigation` компонент:
  - Меню для гостей (Туры, Категории, Вход, Регистрация)
  - Меню для клиентов (+ Мои бронирования, Профиль)
  - Меню для админов (+ Админ панель)
  - Undo кнопка
  - Индикатор пользователя
- [ ] Настроить роутинг в `App.jsx`:
  - Публичные маршруты
  - Защищенные маршруты (ProtectedRoute)
  - Админ маршруты (AdminRoute)
- [ ] Создать `NotFound` страницу
- [ ] Адаптивное меню (burger menu для мобильных)

**Паттерны проектирования:**
- **React**: Layout Pattern, Render Props (для роутинга), Responsive Design Pattern, Breadcrumb Pattern
- **Применение**: Общий Layout для всех страниц, адаптивная навигация, динамические breadcrumbs

**Файлы:**
- `src/components/layout/Layout.jsx`
- `src/components/layout/Layout.module.css`
- `src/components/layout/Header.jsx`
- `src/components/layout/Header.module.css`
- `src/components/layout/Navigation.jsx`
- `src/components/layout/Navigation.module.css`
- `src/components/layout/Sidebar.jsx`
- `src/components/layout/Sidebar.module.css`
- `src/components/layout/Footer.jsx`
- `src/components/layout/Footer.module.css`
- `src/App.jsx`
- `src/pages/NotFound.jsx`
- `src/pages/NotFound.module.css`

---

## Общая сводка по студентам

| Студент | Основные задачи |
|---------|-----------------|
| **Студент 1** | Аутентификация, JWT, User CRUD, формы входа/регистрации |
| **Студент 2** | Category, Tour модели и CRUD, каталог, админ панель туров |
| **Студент 3** | Booking модель и сервис, 3 отчета, бронирования, отчеты UI |
| **Студент 4** | Reviews, Backup/Restore, CTRL+Z, паттерны, навигация |

---

## Используемые паттерны проектирования

### Backend (Ruby on Rails)

1. **Active Record Pattern** - встроенный ORM Rails
2. **Service Object Pattern** - AuthService, BookingService, ReportService, BackupService
3. **Strategy Pattern** ⭐ - DataSourceStrategy (DB vs File)
4. **Command Pattern** ⭐ - CreateBookingCommand, CancelBookingCommand
5. **Factory Method Pattern** ⭐ - ReportFactory
6. **Serializer Pattern** - для JSON представления
7. **Concern Pattern** - AuthenticationConcern
8. **Query Object Pattern** - для сложных запросов
9. **Exception Hierarchy** - кастомные исключения
10. **Template Method** - в валидациях и callbacks
11. **Repository Pattern** - для доступа к данным
12. **Transaction Script** - для атомарных операций

### Frontend (React)

1. **Context API Pattern** - AuthContext для глобального состояния
2. **Higher-Order Component (HOC)** - ProtectedRoute, AdminRoute
3. **Service Layer Pattern** - API сервисы
4. **Presentational/Container Pattern** - разделение логики и UI
5. **Custom Hooks Pattern** - useUndo, useKeyboardShortcut, useFilter
6. **Compound Components** - сложные UI компоненты
7. **Controlled Components** - формы
8. **Modal Pattern** - диалоговые окна
9. **Layout Pattern** - общий layout
10. **Render Props** - для гибкости компонентов
11. **Composition Pattern** - композиция компонентов
12. **Observer Pattern** - через useEffect и state

---

## Критерии приемки

### Для каждого студента:
- [ ] Все миграции созданы и работают
- [ ] Все модели с валидациями
- [ ] Все API endpoints реализованы
- [ ] Все frontend компоненты работают
- [ ] Код соответствует стандартам (Rubocop, ESLint)
- [ ] Документация в README
- [ ] Pull requests с описанием изменений

### Общие требования:
- [ ] Все CRUD операции для 5 сущностей
- [ ] Валидация всех полей
- [ ] 3 содержательных отчета
- [ ] Backup/Restore в JSON/YAML
- [ ] Fallback на файл при отключении БД
- [ ] CTRL+Z функциональность
- [ ] 3 паттерна (Strategy, Command, Factory)
- [ ] Обработка исключений с иерархией
- [ ] Диаграммы классов и последовательностей
- [ ] Схема БД в 3НФ

---

## Полезные ресурсы

### Backend (Ruby on Rails)
- [Rails Guides](https://guides.rubyonrails.org/)
- [JWT Ruby](https://github.com/jwt/ruby-jwt)
- [Design Patterns in Ruby](https://refactoring.guru/design-patterns/ruby)

### Frontend (React)
- [React Documentation](https://react.dev/)
- [React Router](https://reactrouter.com/)
- [CSS Modules](https://github.com/css-modules/css-modules)
- [Axios](https://axios-http.com/)

### Паттерны проектирования
- [Refactoring Guru](https://refactoring.guru/design-patterns)
- Strategy Pattern
- Command Pattern
- Factory Method Pattern

### Git
- [Git Flow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)
- [Conventional Commits](https://www.conventionalcommits.org/)
