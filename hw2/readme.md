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

## Основные операции

#### Добавление пользователя
```redis
HMSET user:nickname:<nickname> first_name "<First Name>" last_name "<Last Name>" date_of_birth "<Date of Birth>" rating <Rating> city_from "<City>" favorite_tags "<Tags>"
```
- **Объяснение**: устанавливает данные пользователя в хэш.

#### Модификация данных пользователя
```redis
HSET user:nickname:<nickname> <field> <value>
```
- **Объяснение**: изменяет значение указанного поля в хэше пользователя.

#### Удаление пользователя
```redis
DEL user:nickname:<nickname>
```
- **Объяснение**: удаляет хэш пользователя.

#### Добавление поста
```redis
HMSET post:id:<id> title "<Title>" body "<Body>" creation_date_str "<Creation Date>" creation_date <Creation Date in Seconds> tags "<Tags>"
LPUSH user:nickname:<nickname>:posts <id>
ZADD global:posts <Creation Date in Seconds> post:id:<id>
```
- **Объяснение**: создаёт хэш поста, добавляет его идентификатор в список постов пользователя и в глобальный список постов.

#### Добавление основного тега к посту
```redis
HSET post:id:<id> tags "<Main Tag>,<Other Tags>"
```
- **Объяснение**: изменяет список тегов поста, добавляя основной тег.

#### Удаление тега из поста
```redis
HSET post:id:<id> tags "<Other Tags>"
```
- **Объяснение**: изменяет список тегов поста, удаляя указанный тег.

#### Удаление поста
```redis
DEL post:id:<id>
```
- **Объяснение**: удаляет хэш поста.

#### Отображение всех постов, отсортированных по дате создания в обратном порядке
```redis
ZREVRANGE global:posts 0 -1
```
- **Объяснение**: возвращает идентификаторы постов из глобального списка, отсортированные в обратном порядке по дате создания.

#### Отображение постов конкретного пользователя, отсортированных по дате создания в обратном порядке
```redis
LRANGE user:nickname:<nickname>:posts 0 -1
```
- **Объяснение**: возвращает идентификаторы постов пользователя из его списка постов.

#### Отображение постов по конкретному тегу, отсортированных по дате создания в обратном порядке
```redis
ZREVRANGE tag:<tag>:posts 0 -1
```
- **Объяснение**: возвращает идентификаторы постов с указанным тегом, отсортированные в обратном порядке по дате создания.

### Транзакции

Для обеспечения атомарности операций можно использовать транзакции. Например, для добавления поста:
```redis
MULTI
HMSET post:id:<id> title "<Title>" body "<Body>" creation_date_str "<Creation Date>" creation_date <Creation Date in Seconds> tags "<Tags>"
LPUSH user:nickname:<nickname>:posts <id>
ZADD global:posts <Creation Date in Seconds> post:id:<id>
EXEC
```
- **Объяснение**: команды внутри `MULTI` и `EXEC` выполняются атомарно, что гарантирует целостность данных.

### Использование WATCH

Команда `WATCH` позволяет отслеживать изменения переменных перед выполнением транзакции. Если переменная изменится, транзакция будет отменена.
```redis
WATCH user:nickname:<nickname>
MULTI
HSET user:nickname:<nickname> <field> <value>
EXEC
```
- **Объяснение**: `WATCH` отслеживает изменения хэша пользователя, и если он изменится до выполнения `EXEC`, транзакция будет отменена.

### Дополнительные функции

#### Подсчёт уникальных посетителей по дням (Bitmap)
```redis
SETBIT visitors:day:<day> <visitor_id> 1
```
- **Объяснение**: устанавливает бит для идентификатора посетителя в bitmap за указанный день.

#### Подсчёт уникальных посетителей по дням (HyperLogLog)
```redis
PFADD visitors:day:<day> <visitor_id>
```
- **Объяснение**: добавляет идентификатор посетителя в HyperLogLog за указанный день.

#### Получение дней, в которые конкретный пользователь посещал блог (Bitmap)
```redis
BITCOUNT visitors:day:<day>
```
- **Объяснение**: подсчитывает количество установленных битов в bitmap за указанный день.

#### Подсчёт общего количества уникальных посетителей за три дня (HyperLogLog)
```redis
PFMERGE visitors:three_days visitors:day:<day1> visitors:day:<day2> visitors:day:<day3>
PFCARD visitors:three_days
```
- **Объяснение**: объединяет HyperLogLog за три дня и подсчитывает общее количество уникальных посетителей.
