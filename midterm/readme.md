# Задание 1

## Набор операторов для проектирования такой БД

### Студенты

```sql
CREATE TABLE Student (
    student_id BIGINT PRIMARY KEY,
    registration_date DATE NOT NULL
);
```

### Курсы

```sql
CREATE TABLE Course (
    course_id BIGINT PRIMARY KEY,
    title VARCHAR(255) NOT NULL
);
```

### Модули

```sql
CREATE TABLE Module (
    module_id BIGINT PRIMARY KEY,
    course_id BIGINT NOT NULL,
    title VARCHAR(255),
    FOREIGN KEY (course_id) REFERENCES Course(course_id)
);
```

### Уроки

```sql
CREATE TABLE Lesson (
    lesson_id BIGINT PRIMARY KEY,
    module_id BIGINT NOT NULL,
    topic_id BIGINT NOT NULL,
    title VARCHAR(255),
    FOREIGN KEY (module_id) REFERENCES Module(module_id),
    FOREIGN KEY (topic_id) REFERENCES Topic(topic_id)
);
```

### Учебные элементы 

```sql
CREATE TABLE LearningElement (
    element_id BIGINT PRIMARY KEY,
    lesson_id BIGINT NOT NULL,
    type_id INT NOT NULL,
    difficulty_level INT CHECK (difficulty_level BETWEEN 1 AND 5),
    is_required BOOLEAN NOT NULL,
    metadata JSON, -- данные, которые нужны для видео, иозображений или любых файлов
    FOREIGN KEY (lesson_id) REFERENCES Lesson(lesson_id),
    FOREIGN KEY (type_id) REFERENCES ElementType(type_id)
);
```

### Типы учебных элементов

```sql
CREATE TABLE ElementType (
    type_id INT PRIMARY KEY,
    type_name VARCHAR(50) NOT NULL -- текст, видео, тест и тд
);
```

### Темы

```sql
CREATE TABLE Topic (
    topic_id BIGINT PRIMARY KEY,
    name VARCHAR(255) NOT NULL
);
```

### Прогресс студента и его информация

```sql
CREATE TABLE StudentProgress (
    progress_id BIGINT PRIMARY KEY,
    student_id BIGINT NOT NULL,
    course_id BIGINT NOT NULL,
    module_id BIGINT NOT NULL,
    lesson_id BIGINT NOT NULL,
    element_id BIGINT NOT NULL,
    topic_id BIGINT NOT NULL,
    element_type_id INT NOT NULL,
    difficulty_level INT NOT NULL,
    started_at DATETIME NOT NULL,
    finished_at DATETIME,
    score DECIMAL(5,2),
    is_required BOOLEAN NOT NULL,
    FOREIGN KEY (student_id) REFERENCES Student(student_id),
    FOREIGN KEY (course_id) REFERENCES Course(course_id),
    FOREIGN KEY (module_id) REFERENCES Module(module_id),
    FOREIGN KEY (lesson_id) REFERENCES Lesson(lesson_id),
    FOREIGN KEY (element_id) REFERENCES Element(element_id),
    FOREIGN KEY (topic_id) REFERENCES Topic(topic_id),
    FOREIGN KEY (element_type_id) REFERENCES ElementType(type_id)
);
```

## Откуда взять интересующие численные данные/параметры

### Cколько студентов зарегистрировано на курс/модуль/урок

```sql
-- Для курса
SELECT course_id, COUNT(DISTINCT student_id) AS student_count
FROM StudentProgress
GROUP BY course_id;

-- Для модуля
SELECT module_id, COUNT(DISTINCT student_id) AS student_count
FROM StudentProgress
GROUP BY module_id;

-- Для урока
SELECT lesson_id, COUNT(DISTINCT student_id) AS student_count
FROM StudentProgress
GROUP BY lesson_id;
```

### Cколько суммарно/минимально/максимально затрачено времени на курс/модуль/урок

```sql
SELECT course_id,
       SUM(TIMESTAMPDIFF(SECOND, started_at, finished_at)) AS total_time,
       MIN(TIMESTAMPDIFF(SECOND, started_at, finished_at)) AS min_time,
       MAX(TIMESTAMPDIFF(SECOND, started_at, finished_at)) AS max_time
FROM StudentProgress
WHERE finished_at IS NOT NULL
GROUP BY course_id;

-- Для модуля
SELECT module_id,
       SUM(TIMESTAMPDIFF(SECOND, started_at, finished_at)) AS total_time,
       MIN(TIMESTAMPDIFF(SECOND, started_at, finished_at)) AS min_time,
       MAX(TIMESTAMPDIFF(SECOND, started_at, finished_at)) AS max_time
FROM StudentProgress
WHERE finished_at IS NOT NULL
GROUP BY module_id;

-- Для урока
SELECT lesson_id,
       SUM(TIMESTAMPDIFF(SECOND, started_at, finished_at)) AS total_time,
       MIN(TIMESTAMPDIFF(SECOND, started_at, finished_at)) AS min_time,
       MAX(TIMESTAMPDIFF(SECOND, started_at, finished_at)) AS max_time
FROM StudentProgress
WHERE finished_at IS NOT NULL
GROUP BY lesson_id;
```

### С какой средней/минимальной/максимальной оценкой

```sql
-- Для курса
SELECT course_id,
       AVG(score) AS avg_score,
       MIN(score) AS min_score,
       MAX(score) AS max_score
FROM StudentProgress
WHERE score IS NOT NULL
GROUP BY course_id;

-- Для модуля
SELECT module_id,
       AVG(score) AS avg_score,
       MIN(score) AS min_score,
       MAX(score) AS max_score
FROM StudentProgress
WHERE score IS NOT NULL
GROUP BY module_id;

-- Для уркоа
SELECT lesson_id,
       AVG(score) AS avg_score,
       MIN(score) AS min_score,
       MAX(score) AS max_score
FROM StudentProgress
WHERE score IS NOT NULL
GROUP BY lesson_id;
```

### Набор тем

```sql
SELECT * FROM Topic;
```

### Период времени (год/месяц/неделя/день) 

**Если речь про количество начатых курсов, то информацию можно получить следующими запросами:**

```sql
-- Количество начатых курсов по дням
SELECT DATE(started_at) AS day, COUNT(*) AS views
FROM StudentProgress
GROUP BY DATE(started_at);

-- Количество начатых курсов по месяцам
SELECT YEAR(started_at) AS year, MONTH(started_at) AS month, COUNT(*) AS views
FROM StudentProgress
GROUP BY YEAR(started_at), MONTH(started_at);

-- Количество начатых курсов по неделям
SELECT YEAR(started_at) AS year, WEEK(started_at) AS week, COUNT(*) AS views
FROM StudentProgress
GROUP BY YEAR(started_at), WEEK(started_at);
```

### Время регистрации (последняя неделя/месяц/год или конкретный месяц/год)

```sql
-- За последнюю неделю
SELECT COUNT(*) AS recent_month_students
FROM Student
WHERE registration_date >= CURRENT_DATE - INTERVAL 1 MONTH;

-- За последний месяц
SELECT COUNT(*) AS recent_month_students
FROM Student
WHERE registration_date >= CURRENT_DATE - INTERVAL 1 MONTH;

-- За последний год
SELECT COUNT(*) AS recent_year_students
FROM Student
WHERE registration_date >= CURRENT_DATE - INTERVAL 1 YEAR;
```

### Набор уровней сложности

```sql
-- Все уровни сложностей, которые сейчас есть у каких-либо учебных элементов
SELECT DISTINCT difficulty_level
FROM LearningElement
WHERE difficulty_level IS NOT NULL;
```

### Набор типов учебных элементов (видео/текст/изображение/аудио/тест/форма) 

```sql
SELECT DISTINCT et.type_name
FROM ElementType et
JOIN LearningElement e ON e.element_type_id = et.element_type_id;
```

# Задание 2

### Найти, сколько максимально затрачено времени на урок в курсах сложности больше трех на тему Hadoop с видеоэлементами в зависимости от календарного месяца регистрации на курс

```sql
SELECT 
    l.lesson_id,
    MAX(TIMESTAMPDIFF(SECOND, sp.started_at, sp.finished_at)) AS max_time_spent,
    YEAR(s.registration_date) AS reg_year,
    MONTH(s.registration_date) AS reg_month
FROM StudentProgress sp
JOIN Lesson l ON sp.lesson_id = l.lesson_id
JOIN Topic t ON l.topic_id = t.topic_id
JOIN ElementType et ON sp.element_type_id = et.type_id
JOIN Student s ON sp.student_id = s.student_id
WHERE t.name = 'Hadoop'
  AND sp.difficulty_level > 3
  AND et.type_name = 'видео'
  AND sp.finished_at IS NOT NULL
GROUP BY l.lesson_id, YEAR(s.registration_date), MONTH(s.registration_date);
```

### Найти все темы, на которых училось максимальное количество студентов, зарегистрированных в прошлом году

```sql
WITH StudentsLastYear AS (
    SELECT student_id
    FROM Student
    WHERE YEAR(registration_date) = YEAR(CURDATE()) - 1
),
TopicStudentCounts AS (
    SELECT l.topic_id, COUNT(DISTINCT sp.student_id) AS student_count
    FROM StudentProgress sp
    JOIN Lesson l ON sp.lesson_id = l.lesson_id
    WHERE sp.student_id IN (SELECT student_id FROM StudentsLastYear)
    GROUP BY l.topic_id
),
MaxCount AS (
    SELECT MAX(student_count) AS max_count FROM TopicStudentCounts
)
SELECT t.name
FROM TopicStudentCounts tc
JOIN Topic t ON tc.topic_id = t.topic_id
JOIN MaxCount mc ON tc.student_count = mc.max_count;
```

# Задание 3



# Задание 4


