# UML Диаграммы проекта

## 1. Диаграмма классов (Class Diagram)

```
┌─────────────────────────────────────────────────────────────────────┐
│                              User                                   │
├─────────────────────────────────────────────────────────────────────┤
│ - id: Integer                                                       │
│ - email: String                                                     │
│ - password_digest: String                                           │
│ - full_name: String                                                 │
│ - phone: String                                                     │
│ - role: Enum ['client', 'admin']                                    │
│ - created_at: DateTime                                              │
│ - updated_at: DateTime                                              │
├─────────────────────────────────────────────────────────────────────┤
│ + authenticate(password: String): Boolean                           │
│ + admin?(): Boolean                                                 │
│ + client?(): Boolean                                                │
│ + bookings(): Array<Booking>                                        │
│ + reviews(): Array<Review>                                          │
└────────────────────┬────────────────────────────────────────────────┘
                     │
                     │ 1
                     │
                     │ has_many
                     │
                     │ *
┌────────────────────▼────────────────────────────────────────────────┐
│                           Booking                                   │
├─────────────────────────────────────────────────────────────────────┤
│ - id: Integer                                                       │
│ - user_id: Integer (FK)                                             │
│ - tour_id: Integer (FK)                                             │
│ - booking_date: DateTime                                            │
│ - number_of_people: Integer                                         │
│ - total_price: Decimal                                              │
│ - status: Enum ['pending', 'confirmed', 'cancelled']                │
│ - notes: Text                                                       │
│ - created_at: DateTime                                              │
│ - updated_at: DateTime                                              │
├─────────────────────────────────────────────────────────────────────┤
│ + confirm(): Boolean                                                │
│ + cancel(): Boolean                                                 │
│ + calculate_total_price(): Decimal                                  │
│ + can_be_cancelled?(): Boolean                                      │
│ + pending?(): Boolean                                               │
│ + confirmed?(): Boolean                                             │
│ + cancelled?(): Boolean                                             │
└────────────────────┬────────────────────────────────────────────────┘
                     │
                     │ *
                     │
                     │ belongs_to
                     │
                     │ 1
┌────────────────────▼────────────────────────────────────────────────┐
│                             Tour                                    │
├─────────────────────────────────────────────────────────────────────┤
│ - id: Integer                                                       │
│ - title: String                                                     │
│ - description: Text                                                 │
│ - destination: String                                               │
│ - duration_days: Integer                                            │
│ - price: Decimal                                                    │
│ - available_seats: Integer                                          │
│ - start_date: Date                                                  │
│ - end_date: Date                                                    │
│ - category_id: Integer (FK)                                         │
│ - created_at: DateTime                                              │
│ - updated_at: DateTime                                              │
├─────────────────────────────────────────────────────────────────────┤
│ + available?(): Boolean                                             │
│ + book_seats(count: Integer): Boolean                               │
│ + release_seats(count: Integer): Boolean                            │
│ + average_rating(): Float                                           │
│ + total_bookings(): Integer                                         │
│ + revenue(): Decimal                                                │
│ + occupancy_rate(): Float                                           │
│ + bookings(): Array<Booking>                                        │
│ + reviews(): Array<Review>                                          │
│ + category(): Category                                              │
└────────────────────┬────────────────────────────────────────────────┘
                     │
                     │ *
                     │
                     │ belongs_to
                     │
                     │ 1
┌────────────────────▼────────────────────────────────────────────────┐
│                          Category                                   │
├─────────────────────────────────────────────────────────────────────┤
│ - id: Integer                                                       │
│ - name: String                                                      │
│ - description: Text                                                 │
│ - created_at: DateTime                                              │
│ - updated_at: DateTime                                              │
├─────────────────────────────────────────────────────────────────────┤
│ + tours(): Array<Tour>                                              │
│ + tours_count(): Integer                                            │
│ + total_revenue(): Decimal                                          │
│ + average_tour_price(): Decimal                                     │
└─────────────────────────────────────────────────────────────────────┘


┌─────────────────────────────────────────────────────────────────────┐
│                            Review                                   │
├─────────────────────────────────────────────────────────────────────┤
│ - id: Integer                                                       │
│ - user_id: Integer (FK)                                             │
│ - tour_id: Integer (FK)                                             │
│ - rating: Integer (1-5)                                             │
│ - comment: Text                                                     │
│ - created_at: DateTime                                              │
│ - updated_at: DateTime                                              │
├─────────────────────────────────────────────────────────────────────┤
│ + user(): User                                                      │
│ + tour(): Tour                                                      │
│ + validate_user_booked_tour(): Boolean                              │
└─────────────────────────────────────────────────────────────────────┘
         │                              │
         │ *                            │ *
         │                              │
         │ belongs_to                   │ belongs_to
         │                              │
         │ 1                            │ 1
         └──────────► User              └──────────► Tour
```

## 2. Диаграмма последовательности: Создание бронирования (Sequence Diagram)

```
Клиент    Frontend    API Gateway    AuthController    BookingsController    BookingService    Tour    Booking    Database
  │          │             │                │                   │                   │           │        │          │
  │──Выбор тура──>│        │                │                   │                   │           │        │          │
  │          │             │                │                   │                   │           │        │          │
  │──Заполнение формы─>│   │                │                   │                   │           │        │          │
  │          │             │                │                   │                   │           │        │          │
  │          │─POST /api/v1/bookings────────>│                  │                   │           │        │          │
  │          │             │                │                   │                   │           │        │          │
  │          │             │                │──authenticate_request!──>             │           │        │          │
  │          │             │                │                   │                   │           │        │          │
  │          │             │                │<──current_user────│                   │           │        │          │
  │          │             │                │                   │                   │           │        │          │
  │          │             │                │──create(params)──>│                   │           │        │          │
  │          │             │                │                   │                   │           │        │          │
  │          │             │                │                   │──create_booking(user, tour_id, params)──>        │
  │          │             │                │                   │                   │           │        │          │
  │          │             │                │                   │                   │──find(tour_id)──>  │          │
  │          │             │                │                   │                   │           │        │          │
  │          │             │                │                   │                   │           │─SELECT─>│         │
  │          │             │                │                   │                   │           │        │          │
  │          │             │                │                   │                   │<──tour────│        │          │
  │          │             │                │                   │                   │           │        │          │
  │          │             │                │                   │                   │──available?(num_people)       │
  │          │             │                │                   │                   │           │        │          │
  │          │             │                │                   │                   │<──true────│        │          │
  │          │             │                │                   │                   │           │        │          │
  │          │             │                │                   │                   │──calculate_total_price()      │
  │          │             │                │                   │                   │           │        │          │
  │          │             │                │                   │                   │──BEGIN TRANSACTION──>         │
  │          │             │                │                   │                   │           │        │          │
  │          │             │                │                   │                   │──new(attributes)──>│          │
  │          │             │                │                   │                   │           │        │          │
  │          │             │                │                   │                   │           │──save──>│         │
  │          │             │                │                   │                   │           │        │          │
  │          │             │                │                   │                   │           │        │─INSERT─>│
  │          │             │                │                   │                   │           │        │          │
  │          │             │                │                   │                   │──book_seats(num_people)       │
  │          │             │                │                   │                   │           │        │          │
  │          │             │                │                   │                   │           │─update─>│         │
  │          │             │                │                   │                   │           │        │          │
  │          │             │                │                   │                   │           │        │─UPDATE─>│
  │          │             │                │                   │                   │           │        │          │
  │          │             │                │                   │                   │──COMMIT TRANSACTION──>        │
  │          │             │                │                   │                   │           │        │          │
  │          │             │                │                   │                   │──notify_observers(:created)   │
  │          │             │                │                   │                   │           │        │          │
  │          │             │                │                   │<──booking─────────│           │        │          │
  │          │             │                │                   │                   │           │        │          │
  │          │             │                │<──201 Created─────│                   │           │        │          │
  │          │             │                │   {booking_data}  │                   │           │        │          │
  │          │             │                │                   │                   │           │        │          │
  │          │<────────────────201 Created──│                   │                   │           │        │          │
  │          │             │   {booking_data}                   │                   │           │        │          │
  │          │             │                │                   │                   │           │        │          │
  │<─Успешно─│             │                │                   │                   │           │        │          │
  │  создано │             │                │                   │                   │           │        │          │
```

## 3. Диаграмма последовательности: Отображение туров с категориями

```
Клиент    Frontend    API Gateway    ToursController    Tour Model    Category Model    Database
  │          │             │                │                │               │              │
  │──Открыть каталог─>│    │                │                │               │              │
  │          │             │                │                │               │              │
  │          │─GET /api/v1/tours?category_id=1&sort=price────>│               │              │
  │          │             │                │                │               │              │
  │          │             │                │──index()──────>│               │              │
  │          │             │                │                │               │              │
  │          │             │                │──Tour.includes(:category)─────>│              │
  │          │             │                │                │               │              │
  │          │             │                │                │──SQL JOIN────────────────────>│
  │          │             │                │                │               │              │
  │          │             │                │                │               │  SELECT tours.*, categories.*
  │          │             │                │                │               │  FROM tours
  │          │             │                │                │               │  LEFT JOIN categories
  │          │             │                │                │               │  ON tours.category_id = categories.id
  │          │             │                │                │               │  WHERE category_id = 1
  │          │             │                │                │               │  ORDER BY price
  │          │             │                │                │               │              │
  │          │             │                │                │<──tours with categories──────│
  │          │             │                │                │               │              │
  │          │             │                │<──tours────────│               │              │
  │          │             │                │                │               │              │
  │          │             │                │──serialize(tours)              │              │
  │          │             │                │                │               │              │
  │          │             │<──200 OK───────│                │               │              │
  │          │             │   [{tour_data with category}]   │               │              │
  │          │             │                │                │               │              │
  │          │<────────────────200 OK────────│                │               │              │
  │          │             │   [{tour_data}]│                │               │              │
  │          │             │                │                │               │              │
  │          │──Рендер карточек туров        │                │               │              │
  │          │   с категориями               │                │               │              │
  │          │             │                │                │               │              │
  │<─Отображение─│         │                │                │               │              │
```

## 4. Диаграмма последовательности: Аутентификация пользователя

```
Клиент    LoginPage    AuthService    API    AuthController    User Model    AuthService    Database
  │          │             │           │           │                │              │            │
  │──Ввод данных─>│        │           │           │                │              │            │
  │          │             │           │           │                │              │            │
  │──Submit──>│            │           │           │                │              │            │
  │          │             │           │           │                │              │            │
  │          │─login(email, password)─>│           │                │              │            │
  │          │             │           │           │                │              │            │
  │          │             │─POST /api/v1/auth/login─>              │              │            │
  │          │             │           │           │                │              │            │
  │          │             │           │           │──login()──────>│              │            │
  │          │             │           │           │                │              │            │
  │          │             │           │           │──User.find_by(email)──────────>│           │
  │          │             │           │           │                │              │            │
  │          │             │           │           │                │──SELECT──────────────────>│
  │          │             │           │           │                │              │            │
  │          │             │           │           │                │<──user───────────────────│
  │          │             │           │           │                │              │            │
  │          │             │           │           │<──user─────────│              │            │
  │          │             │           │           │                │              │            │
  │          │             │           │           │──user.authenticate(password)──>│           │
  │          │             │           │           │                │              │            │
  │          │             │           │           │<──true─────────│              │            │
  │          │             │           │           │                │              │            │
  │          │             │           │           │──AuthService.encode(user.id)──>│           │
  │          │             │           │           │                │              │            │
  │          │             │           │           │                │              │──JWT.encode()
  │          │             │           │           │                │              │            │
  │          │             │           │           │<──token────────│              │            │
  │          │             │           │           │                │              │            │
  │          │             │           │<──200 OK──│                │              │            │
  │          │             │           │   {token, user}            │              │            │
  │          │             │           │           │                │              │            │
  │          │             │<──response─│           │                │              │            │
  │          │             │           │           │                │              │            │
  │          │             │──save token to localStorage            │              │            │
  │          │             │                       │                │              │            │
  │          │<─success────│           │           │                │              │            │
  │          │             │           │           │                │              │            │
  │<─redirect to dashboard─│           │           │                │              │            │
```

## 5. Диаграмма компонентов (Component Diagram)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           Frontend (React)                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐         │
│  │   Auth Module    │  │   Tours Module   │  │  Bookings Module │         │
│  ├──────────────────┤  ├──────────────────┤  ├──────────────────┤         │
│  │ - LoginPage      │  │ - ToursPage      │  │ - MyBookingsPage │         │
│  │ - RegisterPage   │  │ - TourDetailPage │  │ - BookingForm    │         │
│  │ - AuthContext    │  │ - TourCard       │  │ - BookingCard    │         │
│  │ - ProtectedRoute │  │ - TourFilters    │  └──────────────────┘         │
│  └──────────────────┘  └──────────────────┘                                │
│                                                                             │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐         │
│  │  Reviews Module  │  │   Admin Module   │  │  Common Module   │         │
│  ├──────────────────┤  ├──────────────────┤  ├──────────────────┤         │
│  │ - ReviewList     │  │ - AdminDashboard │  │ - Layout         │         │
│  │ - ReviewForm     │  │ - AdminToursPage │  │ - Navigation     │         │
│  │ - StarRating     │  │ - ReportsPage    │  │ - UndoButton     │         │
│  └──────────────────┘  │ - BackupPage     │  └──────────────────┘         │
│                        └──────────────────┘                                │
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────┐         │
│  │                    API Services Layer                         │         │
│  ├───────────────────────────────────────────────────────────────┤         │
│  │ authService │ tourService │ bookingService │ reviewService    │         │
│  │ categoryService │ reportService │ backupService │ undoService │         │
│  └───────────────────────────────────────────────────────────────┘         │
│                                                                             │
└──────────────────────────────────┬──────────────────────────────────────────┘
                                   │
                                   │ HTTP/REST API
                                   │ JSON
                                   │
┌──────────────────────────────────▼──────────────────────────────────────────┐
│                        Backend (Ruby on Rails API)                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────┐         │
│  │                      Controllers Layer                        │         │
│  ├───────────────────────────────────────────────────────────────┤         │
│  │ AuthController │ UsersController │ ToursController            │         │
│  │ CategoriesController │ BookingsController │ ReviewsController │         │
│  │ ReportsController │ BackupController │ UndoController         │         │
│  └───────────────────────────────────────────────────────────────┘         │
│                                   │                                         │
│  ┌───────────────────────────────▼───────────────────────────────┐         │
│  │                      Services Layer                            │         │
│  ├────────────────────────────────────────────────────────────────┤         │
│  │ AuthService │ BookingService │ ReportService                  │         │
│  │ BackupService │ NotificationService                            │         │
│  └────────────────────────────────────────────────────────────────┘         │
│                                   │                                         │
│  ┌───────────────────────────────▼───────────────────────────────┐         │
│  │                       Models Layer                             │         │
│  ├────────────────────────────────────────────────────────────────┤         │
│  │ User │ Tour │ Category │ Booking │ Review                      │         │
│  └────────────────────────────────────────────────────────────────┘         │
│                                   │                                         │
│  ┌───────────────────────────────▼───────────────────────────────┐         │
│  │                    Design Patterns Layer                       │         │
│  ├────────────────────────────────────────────────────────────────┤         │
│  │ Strategy (DataSource) │ Command (Undo) │ Observer (Notifications)       │
│  │ Factory (Reports) │ Singleton (Config)                         │         │
│  └────────────────────────────────────────────────────────────────┘         │
│                                                                             │
└──────────────────────────────────┬──────────────────────────────────────────┘
                                   │
                                   │ ActiveRecord ORM
                                   │
┌──────────────────────────────────▼──────────────────────────────────────────┐
│                         SQLite3 Database                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│  users │ tours │ categories │ bookings │ reviews                            │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 6. Диаграмма развертывания (Deployment Diagram)

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Client Browser                               │
├─────────────────────────────────────────────────────────────────────┤
│                      React Application                              │
│                   (JavaScript, HTML, CSS)                            │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             │ HTTPS
                             │
┌────────────────────────────▼────────────────────────────────────────┐
│                      Vercel / Netlify                               │
├─────────────────────────────────────────────────────────────────────┤
│                   Static File Hosting                               │
│                   CDN Distribution                                  │
└─────────────────────────────────────────────────────────────────────┘


┌─────────────────────────────────────────────────────────────────────┐
│                    Heroku / Railway / Render                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌───────────────────────────────────────────────────────┐         │
│  │            Rails API Server (Puma)                    │         │
│  ├───────────────────────────────────────────────────────┤         │
│  │  - Controllers                                        │         │
│  │  - Services                                           │         │
│  │  - Models                                             │         │
│  │  - Background Jobs (Sidekiq)                          │         │
│  └───────────────────────────────────────────────────────┘         │
│                             │                                       │
│                             │                                       │
│  ┌──────────────────────────▼────────────────────────────┐         │
│  │              SQLite3 Database                      │         │
│  ├───────────────────────────────────────────────────────┤         │
│  │  - users                                              │         │
│  │  - tours                                              │         │
│  │  - categories                                         │         │
│  │  - bookings                                           │         │
│  │  - reviews                                            │         │
│  └───────────────────────────────────────────────────────┘         │
│                             │                                       │
│  ┌──────────────────────────▼────────────────────────────┐         │
│  │              Redis (Optional)                         │         │
│  ├───────────────────────────────────────────────────────┤         │
│  │  - Session Storage                                    │         │
│  │  - Command History (Undo)                             │         │
│  │  - Cache                                              │         │
│  └───────────────────────────────────────────────────────┘         │
│                             │                                       │
│  ┌──────────────────────────▼────────────────────────────┐         │
│  │         File Storage (S3 / Local)                     │         │
│  ├───────────────────────────────────────────────────────┤         │
│  │  - Backup Files (JSON/YAML)                           │         │
│  │  - Exports                                            │         │
│  └───────────────────────────────────────────────────────┘         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## 7. Диаграмма паттернов проектирования

### Strategy Pattern (Источник данных)

```
┌─────────────────────────────────────────────────────────────┐
│                   DataSourceStrategy                        │
│                     <<interface>>                           │
├─────────────────────────────────────────────────────────────┤
│ + find_all(model: Class): Array                             │
│ + find_by_id(model: Class, id: Integer): Object             │
│ + create(model: Class, attributes: Hash): Object            │
│ + update(record: Object, attributes: Hash): Boolean         │
│ + destroy(record: Object): Boolean                          │
└────────────────────┬────────────────────────────────────────┘
                     │
         ┌───────────┴───────────┐
         │                       │
┌────────▼────────┐    ┌─────────▼────────┐
│ DatabaseSource  │    │   FileSource     │
├─────────────────┤    ├──────────────────┤
│ + find_all()    │    │ + find_all()     │
│ + find_by_id()  │    │ + find_by_id()   │
│ + create()      │    │ + create()       │
│ + update()      │    │ + update()       │
│ + destroy()     │    │ + destroy()      │
│                 │    │                  │
│ Uses:           │    │ Uses:            │
│ ActiveRecord    │    │ JSON/YAML files  │
└─────────────────┘    └──────────────────┘
         │                       │
         └───────────┬───────────┘
                     │
         ┌───────────▼───────────┐
         │ DataSourceContext     │
         ├───────────────────────┤
         │ - strategy: Strategy  │
         ├───────────────────────┤
         │ + initialize(strategy)│
         │ + switch_strategy()   │
         │ + execute_query()     │
         └───────────────────────┘
```

### Command Pattern (Undo)

```
┌─────────────────────────────────────────────────────────────┐
│                        Command                              │
│                     <<interface>>                           │
├─────────────────────────────────────────────────────────────┤
│ + execute(): void                                           │
│ + undo(): void                                              │
└────────────────────┬────────────────────────────────────────┘
                     │
         ┌───────────┴───────────┬───────────────────┐
         │                       │                   │
┌────────▼────────┐    ┌─────────▼────────┐  ┌──────▼──────────┐
│CreateBooking    │    │CancelBooking     │  │UpdateTour       │
│Command          │    │Command           │  │Command          │
├─────────────────┤    ├──────────────────┤  ├─────────────────┤
│ - booking_params│    │ - booking        │  │ - tour          │
│ - booking       │    │                  │  │ - old_attrs     │
├─────────────────┤    ├──────────────────┤  ├─────────────────┤
│ + execute()     │    │ + execute()      │  │ + execute()     │
│   create booking│    │   cancel booking │  │   update tour   │
│ + undo()        │    │ + undo()         │  │ + undo()        │
│   cancel booking│    │   restore booking│  │   restore attrs │
└─────────────────┘    └──────────────────┘  └─────────────────┘
                                │
                                │
                    ┌───────────▼───────────┐
                    │   CommandHistory      │
                    ├───────────────────────┤
                    │ - history: Array      │
                    │ - max_size: Integer   │
                    ├───────────────────────┤
                    │ + execute_command()   │
                    │ + undo_last()         │
                    │ + clear_history()     │
                    │ + get_history()       │
                    └───────────────────────┘
```

### Observer Pattern (Уведомления)

```
┌─────────────────────────────────────────────────────────────┐
│                        Subject                              │
│                      (Booking)                              │
├─────────────────────────────────────────────────────────────┤
│ - observers: Array<Observer>                                │
├─────────────────────────────────────────────────────────────┤
│ + attach_observer(observer: Observer): void                 │
│ + detach_observer(observer: Observer): void                 │
│ + notify_observers(event: Symbol): void                     │
└────────────────────┬────────────────────────────────────────┘
                     │
                     │ notifies
                     │
┌────────────────────▼────────────────────────────────────────┐
│                      Observer                               │
│                    <<interface>>                            │
├─────────────────────────────────────────────────────────────┤
│ + update(subject: Subject, event: Symbol): void             │
└────────────────────┬────────────────────────────────────────┘
                     │
         ┌───────────┴───────────┬───────────────────┐
         │                       │                   │
┌────────▼────────┐    ┌─────────▼────────┐  ┌──────▼──────────┐
│EmailNotifier    │    │LoggerObserver    │  │AdminNotifier    │
├─────────────────┤    ├──────────────────┤  ├─────────────────┤
│ + update()      │    │ + update()       │  │ + update()      │
│   send email    │    │   log event      │  │   notify admin  │
└─────────────────┘    └──────────────────┘  └─────────────────┘
```

### Factory Method Pattern (Отчеты)

```
┌─────────────────────────────────────────────────────────────┐
│                     ReportFactory                           │
├─────────────────────────────────────────────────────────────┤
│ + create_report(type: Symbol, params: Hash): Report         │
└────────────────────┬────────────────────────────────────────┘
                     │
                     │ creates
                     │
┌────────────────────▼────────────────────────────────────────┐
│                        Report                               │
│                     <<abstract>>                            │
├─────────────────────────────────────────────────────────────┤
│ - params: Hash                                              │
├─────────────────────────────────────────────────────────────┤
│ + generate(): Hash                                          │
│ + export(format: Symbol): String                            │
│ + to_json(): String                                         │
│ + to_csv(): String                                          │
└────────────────────┬────────────────────────────────────────┘
                     │
         ┌───────────┴───────────┬───────────────────┐
         │                       │                   │
┌────────▼────────┐    ┌─────────▼────────┐  ┌──────▼──────────┐
│ToursStatistics  │    │ClientsAnalysis   │  │FinancialBy      │
│Report           │    │Report            │  │CategoryReport   │
├─────────────────┤    ├──────────────────┤  ├─────────────────┤
│ + generate()    │    │ + generate()     │  │ + generate()    │
│   tours stats   │    │   clients data   │  │   financial data│
│ + export()      │    │ + export()       │  │ + export()      │
└─────────────────┘    └──────────────────┘  └─────────────────┘
```

## 8. Диаграмма состояний для Booking (State Diagram)

```
                    ┌──────────────┐
                    │   [START]    │
                    └──────┬───────┘
                           │
                           │ create_booking()
                           │
                    ┌──────▼───────┐
              ┌─────│   PENDING    │─────┐
              │     └──────┬───────┘     │
              │            │             │
              │            │ confirm()   │ cancel()
              │            │             │
              │     ┌──────▼───────┐     │
              │     │  CONFIRMED   │     │
              │     └──────────────┘     │
              │                          │
              │ cancel()                 │
              │                          │
              │     ┌──────────────┐     │
              └────►│  CANCELLED   │◄────┘
                    └──────┬───────┘
                           │
                           │ [END]
                           ▼

Transitions:
- PENDING → CONFIRMED: admin confirms booking
- PENDING → CANCELLED: user or admin cancels
- CONFIRMED → CANCELLED: admin cancels (with reason)

Business Rules:
- PENDING: booking created, awaiting confirmation
- CONFIRMED: booking confirmed, seats reserved
- CANCELLED: booking cancelled, seats released
```

## 9. Диаграмма активности: Процесс бронирования (Activity Diagram)

```
                    [START]
                       │
                       ▼
              ┌────────────────┐
              │ Выбрать тур    │
              └────────┬───────┘
                       │
                       ▼
              ┌────────────────┐
              │ Проверить      │
              │ авторизацию    │
              └────────┬───────┘
                       │
                ┌──────┴──────┐
                │             │
         [Не авторизован]  [Авторизован]
                │             │
                ▼             ▼
      ┌─────────────┐  ┌──────────────┐
      │ Redirect to │  │ Заполнить    │
      │ Login       │  │ форму        │
      └─────────────┘  └──────┬───────┘
                              │
                              ▼
                     ┌─────────────────┐
                     │ Проверить       │
                     │ доступность мест│
                     └────────┬────────┘
                              │
                       ┌──────┴──────┐
                       │             │
                [Нет мест]      [Есть места]
                       │             │
                       ▼             ▼
              ┌────────────┐  ┌──────────────┐
              │ Показать   │  │ Рассчитать   │
              │ ошибку     │  │ цену         │
              └────────────┘  └──────┬───────┘
                                     │
                                     ▼
                            ┌─────────────────┐
                            │ Создать         │
                            │ бронирование    │
                            │ (транзакция)    │
                            └────────┬────────┘
                                     │
                                     ▼
                            ┌─────────────────┐
                            │ Уменьшить       │
                            │ available_seats │
                            └────────┬────────┘
                                     │
                                     ▼
                            ┌─────────────────┐
                            │ Отправить       │
                            │ уведомления     │
                            └────────┬────────┘
                                     │
                                     ▼
                            ┌─────────────────┐
                            │ Показать        │
                            │ подтверждение   │
                            └────────┬────────┘
                                     │
                                     ▼
                                  [END]
```

## 10. Use Case диаграмма

```
                                    Travel Hub System

┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│                                                                         │
│  ┌──────────────┐        ┌──────────────┐        ┌──────────────┐     │
│  │ Регистрация  │        │ Просмотр     │        │ Поиск и      │     │
│  │              │        │ каталога     │        │ фильтрация   │     │
│  └──────────────┘        └──────────────┘        └──────────────┘     │
│         │                        │                        │            │
│         │                        │                        │            │
│  ┌──────▼──────┐        ┌────────▼────────┐      ┌────────▼──────┐   │
│  │ Авторизация │        │ Просмотр деталей│      │ Создание      │   │
│  │             │        │ тура            │      │ бронирования  │   │
│  └──────────────┘       └─────────────────┘      └───────────────┘   │
│         │                                                 │            │
│         │                                                 │            │
│         └─────────────────────┬─────────────────────────┘            │
│                               │                                       │
│                               │                                       │
└───────────────────────────────┼───────────────────────────────────────┘
                                │
                                │
                    ┌───────────┴───────────┐
                    │                       │
              ┌─────▼─────┐          ┌──────▼──────┐
              │  Клиент   │          │    Админ    │
              │  (Client) │          │   (Admin)   │
              └───────────┘          └──────┬──────┘
                    │                       │
                    │                       │
┌───────────────────┼───────────────────────┼───────────────────────────┐
│                   │                       │                           │
│  ┌────────────────▼──────┐      ┌─────────▼────────┐                 │
│  │ Управление            │      │ Управление       │                 │
│  │ своими бронированиями │      │ всеми турами     │                 │
│  └───────────────────────┘      └──────────────────┘                 │
│                                                                        │
│  ┌───────────────────────┐      ┌──────────────────┐                 │
│  │ Оставить отзыв        │      │ Управление       │                 │
│  │                       │      │ категориями      │                 │
│  └───────────────────────┘      └──────────────────┘                 │
│                                                                        │
│  ┌───────────────────────┐      ┌──────────────────┐                 │
│  │ Просмотр своих        │      │ Управление       │                 │
│  │ отзывов               │      │ бронированиями   │                 │
│  └───────────────────────┘      └──────────────────┘                 │
│                                                                        │
│                                  ┌──────────────────┐                 │
│                                  │ Просмотр отчетов │                 │
│                                  └──────────────────┘                 │
│                                                                        │
│                                  ┌──────────────────┐                 │
│                                  │ Backup/Restore   │                 │
│                                  └──────────────────┘                 │
│                                                                        │
└────────────────────────────────────────────────────────────────────────┘
```
