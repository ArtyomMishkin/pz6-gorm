# Практическое задание 6

## ЭФМО-02-25 Мишкин Артём Дмитриевич 09.12.2025
---
# Информация о проекте
REST API на Go + GORM + PostgreSQL для управления заметками с поддержкой связей 1:N (User→Notes) и M:N (Note↔Tags). Автоматические миграции таблиц, создание/получение заметок с подгрузкой автора и тегов через Preload.

## Цели занятия
- Освоить ORM GORM: модели, AutoMigrate, связи 1:N и M:N
- Настроить PostgreSQL с GORM driver
- Реализовать REST: /users, /notes (POST+GET с Preload)
- Работать с JSON, тегами gorm:"many2many:note_tags;"

## Файловая структура проекта:
pz6-gorm/
├── cmd/server/main.go
├── internal/db/postgres.go
├── internal/models/models.go
├── internal/http/
│ ├── router.go
│ └── handlers.go
└── go.mod

## ВАЖНОЕ ПРИМЕЧАНИЕ
Порт: **8080** (стандартный)

## Примеры запросов:

### /health

<img width="974" height="369" alt="image" src="https://github.com/user-attachments/assets/7e90ace9-4eb5-45d8-9b13-21100afb1307" />

### Добавление пользователя

<img width="974" height="37" alt="image" src="https://github.com/user-attachments/assets/226af652-8c50-47c5-93cb-a6bb1ad75a44" />

### Добавление задачи

<img width="974" height="223" alt="image" src="https://github.com/user-attachments/assets/f1d9c35c-efa0-44a7-923e-14778b5a97ac" />

### Вызов задачи через Notes

<img width="974" height="196" alt="image" src="https://github.com/user-attachments/assets/8d3fc376-6aff-4dfd-8046-a7625deab03c" />

### База данных(через psql \d)

<img width="456" height="183" alt="image" src="https://github.com/user-attachments/assets/f29e1dbd-80bd-4bf0-aaf5-158c00dfad98" />

## Запуск
$env:DB_DSN="host=127.0.0.1 user=postgres password=postgres dbname=pz6_gorm port=5432 sslmode=disable"
go run ./cmd/server

<img width="788" height="117" alt="image" src="https://github.com/user-attachments/assets/1913f053-b2ea-4196-8b06-a56e0dfc36bb" />

## Проблемы и решения

**GORM M:N не мигрировал** — `unsupported data type: &[]`.
**Решение:** `gorm:"many2many:note_tags;"`

## Контрольные вопросы

### 1. Что такое ORM и зачем она нужна?
**ORM** сопоставляет структуры Go с таблицами БД, позволяя писать `db.Create(&User{})` вместо `INSERT INTO users...`.  
**Плюсы:** ускорение разработки, защита от SQL-инъекций, AutoMigrate, кросс-СУБД.  
**Минусы:** медленнее сырого SQL, сложные запросы всё равно требуют JOIN'ов.

### 2. Связи 1:N и M:N в GORM
**1:N (User→Notes):** `User.Notes []Note` + `Note.UserID uint`  
**M:N (Note↔Tags):** `Note.Tags []Tag \`gorm:"many2many:note_tags;"\`` + `Tag.Notes []Note \`gorm:"many2many:note_tags;"\``

### 3. AutoMigrate
Создаёт/обновляет таблицы по структурам: `db.AutoMigrate(&User{}, &Note{}, &Tag{})`.  
**Недостатки:** не удаляет поля, не меняет типы — для этого нужны миграции.

### 4. Preload vs Find/First
`db.First(&note, 1)` — только заметка.  
`db.Preload("User").Preload("Tags").First(&note, 1)` — заметка + автор + теги одним запросом.  
**Preload** для связанных данных (экономит N+1 запросы).

### 5. Обработка unique constraint
if err := db.Create(&user).Error; err != nil {
writeErr(w, 409, "email already exists") // duplicate key violation
}

GORM возвращает `gorm.ErrDuplicateEntry` или текст ошибки БД.

## Коды ответа
- **201 Created** — создание User/Note
- **200 OK** — GET с Preload  
- **400 Bad Request** — валидация
- **404 Not Found** — ресурс не найден
- **409 Conflict** — unique violation


