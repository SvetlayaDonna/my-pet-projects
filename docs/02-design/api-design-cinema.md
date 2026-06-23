# Данные для проектирования API

## 1. Структура хранения данных

### 1.1. ER-диаграмма
[Ссылка на ER-диаграмму](https://dbdiagram.io/d/69b5622778c6c4bc7ae1c15f)

### 1.2. Описание структуры таблиц БД

**Сокращения:** 
* **PK** — primary key (первичный ключ)
* **FK** — foreign key (внешний ключ)
* **unique** — уникальное значение
* **not null** — не может иметь нулевого значения

#### `users` (Пользователи)
| Имя | Тип | Описание поля |
|---|---|---|
| id | integer | Идентификатор (PK) |
| username | varchar | Имя пользователя/Логин (unique) |
| phone | varchar | Телефон |

#### `bookings` (Брони)
| Имя | Тип | Описание поля |
|---|---|---|
| id | integer | Идентификатор (PK) |
| user_id | integer | Идентификатор пользователя (FK), not null |
| sum | decimal(12,2) | Сумма брони по всем билетам |
| status | enum | Статус брони: `'active'`, `'canceled'`, `'expired'`, `'modified'` |
| date_created | timestamp | Дата и время создания брони |
| date_modified | timestamp | Дата и время изменения брони |

#### `halls` (Залы)
| Имя | Тип | Описание поля |
|---|---|---|
| id | integer | Идентификатор (PK) |
| name | varchar | Название зала |

#### `tickets` (Билеты)
| Имя | Тип | Описание поля |
|---|---|---|
| id | integer | Идентификатор (PK) |
| booking_id | integer | Идентификатор брони (FK), not null |
| session_id | integer | Идентификатор сеанса (FK), not null |
| seat_id | integer | Идентификатор места (FK), not null |
| status | enum | Статус билета: `'active'`, `'canceled'`, `'expired'` |

> **Индексы:** `unique_session_seat` — уникальный индекс по полям (`session_id`, `seat_id`)

#### `seats` (Места)
| Имя | Тип | Описание поля |
|---|---|---|
| id | integer | Идентификатор (PK) |
| hall_id | integer | Идентификатор зала (FK), not null |
| row_number | integer | Номер ряда |
| seat_number | integer | Номер места |

> **Индексы:** `unique_place` — уникальный индекс по полям (`hall_id`, `row_number`, `seat_number`)

#### `films` (Фильмы)
| Имя | Тип | Описание поля |
|---|---|---|
| id | integer | Идентификатор (PK) |
| name | varchar | Название фильма |
| genre | varchar | Жанр фильма |
| description | text | Описание фильма |
| age_rating | enum | Ограничение по возрасту: `'0+'`, `'6+'`, `'12+'`, `'16+'` |
| duration | integer | Продолжительность (в мин.) |

#### `sessions` (Сеансы)
| Имя | Тип | Описание поля |
|---|---|---|
| id | integer | Идентификатор (PK) |
| hall_id | integer | Идентификатор зала (FK), not null |
| film_id | integer | Идентификатор фильма (FK), not null |
| start_datetime | timestamp | Дата и время начала сеанса |
| price | decimal(12,2) | Стоимость сеанса |

---

## 2. Кто будет взаимодействовать с API (actors)

* **2.1.** Мобильное приложение (Посетители кинотеатра).
* **2.2.** Веб-приложение (Администратор кинотеатра).

---

## 3. Какие действия будет выполнять API

### 3.1. Пользователь может:
* **3.1.1.** Узнать наличие мест на сеанс: `GET /sessions/{id}/seats`
* **3.1.2.** Забронировать место: `POST /bookings`
* **3.1.3.** Отменить бронь: `PATCH /bookings/{id}`

### 3.2. Администратор кинотеатра может:
* **3.2.1.** Изменить время сеанса: `PATCH /sessions/{id}`

---

## 4. К каким ресурсам будет обращаться API

* `sessions`
* `seats` (подресурс: `/sessions/{id}/seats`)
* `bookings`

### Структуры ответов (Resources Schema)

#### `sessions` (Сеансы)
| Имя | Тип | Описание поля |
|---|---|---|
| id | integer | Идентификатор |
| hall | object | Объект зала: `id` (integer), `name` (string) |
| film | object | Объект фильма: `id` (integer), `name` (string), `genre` (string), `description` (string), `age_rating` (enum), `duration` (integer) |
| start_datetime | string (DateTime) | Дата и время начала сеанса |
| seats | array of objects | Массив мест: `id` (integer), `row_number` (integer), `seat_number` (integer), `status` (enum: `'free'`, `'booked'`) |

#### `bookings` (Брони)
| Имя | Тип | Описание поля |
|---|---|---|
| id | integer | Идентификатор |
| user | object | Объект пользователя: `id` (integer), `username` (string), `phone` (string) |
| tickets | array of objects | Массив билетов: `id` (integer), `session` (object), `seat` (object), `price` (Money), `status` (enum: `'active'`, `'canceled'`, `'used'`) |
| sum | integer | Сумма брони по всем билетам |
| status | enum | Статус брони: `'active'`, `'canceled'`, `'expired'`, `'modified'` |
| date_created | timestamp | Дата и время создания брони |
| date_modified | timestamp | Дата и время изменения брони |

---

## 5. Параметры запросов и ответов

### 5.1. Для пользователя

#### 5.1.1. Узнать наличие мест на сеанс
* **Метод:** `GET /sessions/{id}`
* **Параметры запроса (Path):** `id`
* **Тело запроса:** -
* **Параметры ответа:** `id`, `hall`, `film`, `start_datetime`, `seats` (массив мест: `id`, `status`)

#### 5.1.2. Забронировать место
* **Метод:** `POST /bookings`
* **Параметры запроса:** -
* **Тело запроса:** `session` (id), `seats` (массив id)
* **Параметры ответа:** `id`, `username`, `tickets`, `sum`, `status`, `date_created`

#### 5.1.3. Отменить бронь
* **Метод:** `PATCH /bookings/{id}`
* **Параметры запроса (Path):** `id`
* **Тело запроса:** `status`
* **Параметры ответа:** `id`, `username`, `tickets`, `sum`, `status`, `date_created`, `date_modified`

### 5.2. Для администратора кинотеатра

#### 5.2.1. Изменить время сеанса
* **Метод:** `PATCH /sessions/{id}`
* **Параметры запроса (Path):** `id`
* **Тело запроса:** `start_datetime`
* **Параметры ответа:** `id`, `username`, `start_datetime` 
  > *Примечание: `username` добавлен в ответ, чтобы было видно, кто из трёх администраторов поменял время сеанса.*
