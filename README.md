# Домашнее задание к занятию «Базы данных»

**ФИО:** [Сидоренко Алексей Романович]

---

## Задание 1

### Описание таблиц базы данных

На основе предоставленного отчёта данные были нормализованы до третьей нормальной формы (3НФ) для устранения избыточности. Выделено 7 таблиц:

1. **`department_types`** (Типы подразделений)
   - Хранит справочник типов подразделений (например: "Отдел", "Группа", "Департамент").
   - `id`: `SERIAL` (PRIMARY KEY)
   - `type_name`: `VARCHAR(50)` (NOT NULL, UNIQUE)

2. **`departments`** (Структурные подразделения)
   - Хранит названия конкретных подразделений и ссылку на их тип.
   - `id`: `SERIAL` (PRIMARY KEY)
   - `name`: `VARCHAR(150)` (NOT NULL, UNIQUE)
   - `type_id`: `INT` (NOT NULL, FOREIGN KEY -> `department_types.id`)

3. **`positions`** (Должности)
   - Справочник должностей сотрудников (например: "ведущий QA инженер", "специалист").
   - `id`: `SERIAL` (PRIMARY KEY)
   - `title`: `VARCHAR(100)` (NOT NULL, UNIQUE)

4. **`branches`** (Филиалы)
   - Хранит адреса филиалов, разбитые на логические компоненты для удобства поиска и фильтрации.
   - `id`: `SERIAL` (PRIMARY KEY)
   - `region`: `VARCHAR(100)` (NOT NULL)
   - `city`: `VARCHAR(100)` (NOT NULL)
   - `address_line`: `VARCHAR(255)` (NOT NULL)

5. **`projects`** (Проекты)
   - Справочник проектов, на которые могут быть назначены сотрудники.
   - `id`: `SERIAL` (PRIMARY KEY)
   - `name`: `VARCHAR(200)` (NOT NULL, UNIQUE)

6. **`employees`** (Сотрудники)
   - Основная таблица, хранящая персональные данные, оклад, дату найма и внешние ключи к справочникам.
   - `id`: `SERIAL` (PRIMARY KEY)
   - `last_name`: `VARCHAR(100)` (NOT NULL)
   - `first_name`: `VARCHAR(100)` (NOT NULL)
   - `middle_name`: `VARCHAR(100)` (NULL)
   - `position_id`: `INT` (NOT NULL, FOREIGN KEY -> `positions.id`)
   - `department_id`: `INT` (NOT NULL, FOREIGN KEY -> `departments.id`)
   - `branch_id`: `INT` (NOT NULL, FOREIGN KEY -> `branches.id`)
   - `hire_date`: `DATE` (NOT NULL)
   - `salary`: `NUMERIC(12, 2)` (NOT NULL)

7. **`employee_projects`** (Назначения сотрудников на проекты)
   - Промежуточная таблица для реализации связи "многие-ко-многим" между сотрудниками и проектами (один сотрудник может работать над несколькими проектами, и на одном проекте может быть несколько сотрудников).
   - `id`: `SERIAL` (PRIMARY KEY)
   - `employee_id`: `INT` (NOT NULL, FOREIGN KEY -> `employees.id` ON DELETE CASCADE)
   - `project_id`: `INT` (NOT NULL, FOREIGN KEY -> `projects.id` ON DELETE CASCADE)
   - Ограничение: `UNIQUE (employee_id, project_id)`

### Типы связей между таблицами
- `department_types` **1 : N** `departments` (Один тип может включать много подразделений)
- `departments` **1 : N** `employees` (Одно подразделение включает много сотрудников)
- `positions` **1 : N** `employees` (Одна должность может быть у многих сотрудников)
- `branches` **1 : N** `employees` (В одном филиале работает много сотрудников)
- `employees` **N : M** `projects` (Реализуется через таблицу `employee_projects`)

### Схема модели данных

Ниже представлена схема в формате **Mermaid**. 

```mermaid
erDiagram
    DEPARTMENT_TYPES {
        int id PK
        varchar type_name
    }
    DEPARTMENTS {
        int id PK
        varchar name
        int type_id FK
    }
    POSITIONS {
        int id PK
        varchar title
    }
    BRANCHES {
        int id PK
        varchar region
        varchar city
        varchar address_line
    }
    PROJECTS {
        int id PK
        varchar name
    }
    EMPLOYEES {
        int id PK
        varchar last_name
        varchar first_name
        varchar middle_name
        int position_id FK
        int department_id FK
        int branch_id FK
        date hire_date
        numeric salary
    }
    EMPLOYEE_PROJECTS {
        int id PK
        int employee_id FK
        int project_id FK
    }

    DEPARTMENT_TYPES ||--o{ DEPARTMENTS : "включает"
    DEPARTMENTS ||--o{ EMPLOYEES : "employs"
    POSITIONS ||--o{ EMPLOYEES : "holds"
    BRANCHES ||--o{ EMPLOYEES : "located_at"
    EMPLOYEES ||--o{ EMPLOYEE_PROJECTS : "assigned_to"
    PROJECTS ||--o{ EMPLOYEE_PROJECTS : "includes"
