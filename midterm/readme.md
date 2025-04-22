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
CREATE TABLE Dim_Lesson (
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
    FOREIGN KEY (student_id) REFERENCES Dim_Student(student_id),
    FOREIGN KEY (course_id) REFERENCES Dim_Course(course_id),
    FOREIGN KEY (module_id) REFERENCES Dim_Module(module_id),
    FOREIGN KEY (lesson_id) REFERENCES Dim_Lesson(lesson_id),
    FOREIGN KEY (element_id) REFERENCES Dim_Element(element_id),
    FOREIGN KEY (topic_id) REFERENCES Dim_Topic(topic_id),
    FOREIGN KEY (element_type_id) REFERENCES Dim_ElementType(type_id)
);
```

## Учёт просмотра уроков/модулей/модулей

### Просмотр уроков

```sql
CREATE TABLE StudentLessonStatus (
    student_id BIGINT NOT NULL,
    lesson_id BIGINT NOT NULL,
    is_completed BOOLEAN NOT NULL,
    completed_at DATETIME,
    PRIMARY KEY (student_id, lesson_id),
    FOREIGN KEY (student_id) REFERENCES Student(student_id),
    FOREIGN KEY (lesson_id) REFERENCES Lesson(lesson_id)
);
```

### Просмотр модулей

```sql
CREATE TABLE StudentModuleStatus (
    student_id BIGINT NOT NULL,
    module_id BIGINT NOT NULL,
    is_completed BOOLEAN NOT NULL,
    completed_at DATETIME,
    PRIMARY KEY (student_id, module_id),
    FOREIGN KEY (student_id) REFERENCES Student(student_id),
    FOREIGN KEY (module_id) REFERENCES Module(module_id)
);
```

### Просмотр курсов

```sql
CREATE TABLE StudentCourseStatus (
    student_id BIGINT NOT NULL,
    course_id BIGINT NOT NULL,
    is_completed BOOLEAN NOT NULL,
    completed_at DATETIME,
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES Student(student_id),
    FOREIGN KEY (course_id) REFERENCES Course(course_id)
);
```

## Откуда взять интересующие численные данные/параметры

### Cколько студентов зарегистрировано на курс/модуль/урок

```sql
SELECT course_id, COUNT(DISTINCT student_id) AS student_count
FROM StudentProgress
GROUP BY course_id;

SELECT module_id, COUNT(DISTINCT student_id) AS student_count
FROM StudentProgress
GROUP BY module_id;

SELECT lesson_id, COUNT(DISTINCT student_id) AS student_count
FROM StudentProgress
GROUP BY lesson_id;
```

### Cколько суммарно/минимально/максимально затрачено времени на курс/модуль/урок

```sql
```

### С какой средней/минимальной/максимальной оценкой

```sql
```

### Набор тем

```sql
```

### Период времени (год/месяц/неделя/день) 

```sql
```

### Время регистрации (последняя неделя/месяц/год или конкретный месяц/год)

```sql
```

### Набор уровней сложности

```sql
```

### Набор типов учебных элементов (видео/текст/изображение/аудио/тест/форма) 

```sql

```

# Задание 2



# Задание 3



# Задание 4


