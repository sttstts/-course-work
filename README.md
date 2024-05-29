# Типовые запросы
## 1.Получение всех курсов, доступные пользователю:
``` sql
SELECT Courses.course_name, Courses.level, language.title AS language
FROM Courses
JOIN language ON Courses.language_id = language.id;
```

## 2.Получение прогресса пользователя по всем урокам:
``` sql
SELECT Users.user_name, Lessons.lesson_name, Progress.status
FROM Progress
JOIN Users ON Progress.users_id = Users.id
JOIN Lessons ON Progress.lessons_id = Lessons.id
WHERE Users.id = 1;  
```

## 3.Получение всех преподавателей и их курсы:
``` sql
SELECT Instructors.instructor_name, Courses.course_name
FROM Instructors
JOIN Courses_has_Instructors ON Instructors.id = Courses_has_Instructors.instructors_id
JOIN Courses ON Courses_has_Instructors.courses_id = Courses.id;
```

## 4.Получение всех уроков для определенного курса:
``` sql
SELECT Lessons.lesson_name, Lessons.lesson_number
FROM Lessons
WHERE Lessons.courses_id = 1;  
```

## 5.Получение всех курсов на определенном языке:
``` sql
SELECT Courses.course_name, Courses.level
FROM Courses
JOIN language ON Courses.language_id = language.id
WHERE language.id = 1;
```

# Транзакция
## Обновление статуса в таблице прогресс
``` sql
START TRANSACTION;

UPDATE Progress
SET status = 'Completed', completion_date = NOW()
WHERE users_id = 1 AND lessons_id = 1;  

COMMIT;
```

# Локальные переменные, условие, процедуры и обработка событий.
## 1.Обновленеи статуса урока для пользователя
``` sql
DELIMITER //

CREATE PROCEDURE update_progresss(IN user_id INT, IN lesson_id INT)
BEGIN
    DECLARE current_status VARCHAR(20);
    DECLARE new_status VARCHAR(20) DEFAULT 'Completed';
    DECLARE current_times TIMESTAMP DEFAULT CURRENT_TIMESTAMP;

    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Ошибка при обновлении прогресса';
    END;
    
    SELECT status INTO current_status
    FROM Progress
    WHERE users_id = user_id AND lessons_id = lesson_id;

    IF current_status = 'Not Started' OR current_status = 'In Progress' THEN
        UPDATE Progress
        SET status = new_status, completion_date = current_times
        WHERE users_id = user_id AND lessons_id = lesson_id;
    END IF;
END //

DELIMITER ;
```

## 2.Процедура для добавления нового пользователя:
``` sql
DELIMITER //

CREATE PROCEDURE add_user(IN name VARCHAR(255), IN email VARCHAR(255), IN password VARCHAR(255))
BEGIN
    INSERT INTO Users (user_name, Email, password_hash)
    VALUES (name, email, password);
END //

DELIMITER ;
```

# Представлениe
## Представление для получения информации о курсах и преподавателях:
``` sql
CREATE VIEW CourseInstructors AS
SELECT Courses.course_name, Instructors.instructor_name
FROM Courses
JOIN Courses_has_Instructors ON Courses.id = Courses_has_Instructors.courses_id
JOIN Instructors ON Courses_has_Instructors.instructors_id = Instructors.id;
```

# Триггер
## Обновляет дату последнего изменения в таблице Courses
``` sql
DELIMITER //

CREATE TRIGGER add_progress_on_user_insert
AFTER INSERT ON Users
FOR EACH ROW
BEGIN
    DECLARE lesson_id INT;
    DECLARE default_status ENUM('Not Started', 'In Progress', 'Completed');
    
    SET default_status = 'Not Started';

    SELECT id INTO lesson_id
    FROM Lessons
    WHERE lesson_number = 1;
    
    INSERT INTO Progress (users_id, lessons_id, completion_date, status)
    VALUES (NEW.id, lesson_id, NULL, default_status);
END //

DELIMITER ;
```

# Пользовательская функция
## Функция для получения количества уроков в курсе:
``` sql
DELIMITER //

CREATE FUNCTION lesson_count(course_id INT) RETURNS INT
BEGIN
    DECLARE count INT;
    
    SELECT COUNT(*) INTO count
    FROM Lessons
    WHERE courses_id = course_id;
    
    RETURN count;
END //

DELIMITER;
```

# Пользователи
``` sql
CREATE USER 'student_user'@'localhost' IDENTIFIED BY 'password123';
CREATE USER 'instructor_user'@'localhost' IDENTIFIED BY 'password123';
CREATE USER 'admin_user'@'localhost' IDENTIFIED BY 'adminpass';

GRANT SELECT ON mydatabase.* TO 'student_user'@'localhost';

GRANT SELECT, INSERT, UPDATE ON mydatabase.* TO 'instructor_user'@'localhost';

GRANT ALL PRIVILEGES ON mydatabase.* TO 'admin_user'@'localhost';

FLUSH PRIVILEGES;
```
