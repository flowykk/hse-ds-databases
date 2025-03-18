# База данных для блогов в Redis

## Описание сущностей

### Пользователь:

```redis
HMSET user:nickname:ddrakhmanov first_name "Danila" last_name "Rakhmanov" date_of_birth "2004-11-26" rating 100 city_from "Moscow" favorite_tags "it,travel"
```

### Пост:

```redis
HMSET post:id:1 title "My First Very Interesting Post" body "Body of my post." creation_date_str "2024-01-01" creation_date 1704067200 tags "it"
```

### Список постов пользователя:

```redis
LPUSH user:nickname:ddrakhmanov:posts 1
```

`1` - идентификатор поста в список постов пользователя.

### Глобальный список постов:

```redis
ZADD global:posts 1696118400 post:id:1
```

`1704067200` - время создания поста.

## Примеры выполнения операций

### Добавление пользователя

```redis
HMSET user:nickname:<nickname> first_name "<First Name>" last_name "<Last Name>" date_of_birth "<Date of Birth>" rating <Rating> city_from "<City>" favorite_tags "<Tags>"
```

### Модификация данных пользователя

```redis
HSET user:nickname:<nickname> <field> <value>
```

### Удаление пользователя

```redis
DEL user:nickname:<nickname>
```

### Добавление поста

Команда создаёт хэш поста, добавляет его идентификатор в список постов пользователя и в глобальный список постов.

```redis
HMSET post:id:<id> title "<Title>" body "<Body>" creation_date_str "<Creation Date>" creation_date <Creation Date in Seconds> tags "<Tags>"
LPUSH user:nickname:<nickname>:posts <id>
ZADD global:posts <Creation Date in Seconds> post:id:<id>
```

### Добавление основного тега к посту

```redis
HSET post:id:<id> tags "<Main Tag>,<Other Tags>"
```

### Удаление тега из поста

```redis
HSET post:id:<id> tags "<Other Tags>"
```

### Удаление поста

```redis
DEL post:id:<id>
```

### Отображение всех постов, отсортированных по дате создания в обратном порядке

```redis
ZREVRANGE global:posts 0 -1
```

### Отображение постов конкретного пользователя, отсортированных по дате создания в обратном порядке

```redis
LRANGE user:nickname:<nickname>:posts 0 -1
```

### Отображение постов по конкретному тегу, отсортированных по дате создания в обратном порядке

```redis
ZREVRANGE tag:<tag>:posts 0 -1
```

## Использование транзакций

Для обеспечения атомарности операций можно использовать транзакции.

```redis
MULTI
HMSET post:id:<id> title "<Title>" body "<Body>" creation_date_str "<Creation Date>" creation_date <Creation Date in Seconds> tags "<Tags>"
LPUSH user:nickname:<nickname>:posts <id>
ZADD global:posts <Creation Date in Seconds> post:id:<id>
EXEC
```

## Использование WATCH

Здесь `WATCH` отслеживает изменения хэша пользователя, и если он изменится до выполнения `EXEC`, транзакция будет отменена.

```redis
WATCH user:nickname:<nickname>
MULTI
HSET user:nickname:<nickname> <field> <value>
EXEC
```

## Дополнительный функционал

#### Подсчёт уникальных посетителей по дням (Bitmap)

Устанавливка бит для идентификатора посетителя в bitmap за указанный день.

```redis
SETBIT visitors:day:<day> <visitor_id> 1
```

#### Подсчёт уникальных посетителей по дням (HyperLogLog)

Добавленин идентификатор посетителя в HyperLogLog за указанный день.

```redis
PFADD visitors:day:<day> <visitor_id>
```

#### Получение дней, в которые конкретный пользователь посещал блог (Bitmap)

Подсчет количества установленных битов в bitmap за указанный день.

```redis
BITCOUNT visitors:day:<day>
```

#### Подсчёт общего количества уникальных посетителей за три дня (HyperLogLog)

Объединение данных HyperLogLog за три дня и подсчёт общего количества уникальных посетителей.

```redis
PFMERGE visitors:three_days visitors:day:<day1> visitors:day:<day2> visitors:day:<day3>
PFCARD visitors:three_days
```
