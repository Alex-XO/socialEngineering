#### Таблица: `Users` (Пользователи)
Хранит данные о сотрудниках и администраторах системы.

| Поле          | Тип          | Описание                             |
|---------------|--------------|--------------------------------------|
| `Id`          | INT          | Уникальный идентификатор пользователя (PK). |
| `Name`        | NVARCHAR(100)| Имя пользователя.                   |
| `Email`       | NVARCHAR(100)| Email пользователя.                 |
| `Role`        | NVARCHAR(50) | Роль (Admin, Employee).             |
| `PasswordHash`| NVARCHAR(MAX)| Хэш пароля для аутентификации.      |
| `CreatedAt`   | DATETIME     | Дата создания пользователя.         |

---

#### Таблица: `Scenarios` (Сценарии)
Хранит описание сценариев атак.

| Поле          | Тип          | Описание                              |
|---------------|--------------|---------------------------------------|
| `Id`          | INT          | Уникальный идентификатор сценария (PK). |
| `Title`       | NVARCHAR(100)| Название сценария.                   |
| `Description` | NVARCHAR(MAX)| Описание сценария (в чем его суть).  |
| `Complexity`  | INT          | Уровень сложности сценария (1-5).    |
| `CreatedAt`   | DATETIME     | Дата создания сценария.              |

---

#### Таблица: `Tests` (Тесты)
Хранит информацию о прохождении тестов сотрудниками.

| Поле          | Тип          | Описание                             |
|---------------|--------------|--------------------------------------|
| `Id`          | INT          | Уникальный идентификатор теста (PK). |
| `UserId`      | INT          | Ссылка на сотрудника (FK к `Users.Id`). |
| `ScenarioId`  | INT          | Ссылка на сценарий (FK к `Scenarios.Id`). |
| `StartedAt`   | DATETIME     | Время начала теста.                 |
| `CompletedAt` | DATETIME     | Время завершения теста.             |
| `IsPassed`    | BIT          | Пройден ли тест (true/false).       |

---

#### Таблица: `Logs` (Логи действий)
Хранит действия сотрудников в рамках теста.

| Поле          | Тип          | Описание                             |
|---------------|--------------|---------------------------------------|
| `Id`          | INT          | Уникальный идентификатор записи (PK).|
| `TestId`      | INT          | Ссылка на тест (FK к `Tests.Id`).    |
| `Action`      | NVARCHAR(255)| Описание действия (например, "Кликнул на фишинговую ссылку"). |
| `CreatedAt`   | DATETIME     | Дата и время действия.              |

---

#### Пример связей между таблицами
- Один пользователь (`Users`) может проходить множество тестов (`Tests`).
- Каждый тест связан с одним сценарием (`Scenarios`).
- Логи действий (`Logs`) связаны с конкретным тестом.

---

### Пример SQL для создания таблиц

#### Таблица `Users`:
```sql
CREATE TABLE Users (
    Id INT PRIMARY KEY IDENTITY(1,1),
    Name NVARCHAR(100) NOT NULL,
    Email NVARCHAR(100) NOT NULL UNIQUE,
    Role NVARCHAR(50) NOT NULL,
    PasswordHash NVARCHAR(MAX) NOT NULL,
    CreatedAt DATETIME DEFAULT GETDATE()
);
```

#### Таблица `Scenarios`:
```sql
CREATE TABLE Scenarios (
    Id INT PRIMARY KEY IDENTITY(1,1),
    Title NVARCHAR(100) NOT NULL,
    Description NVARCHAR(MAX) NOT NULL,
    Complexity INT NOT NULL CHECK (Complexity BETWEEN 1 AND 5),
    CreatedAt DATETIME DEFAULT GETDATE()
);
```

#### Таблица `Tests`:
```sql
CREATE TABLE Tests (
    Id INT PRIMARY KEY IDENTITY(1,1),
    UserId INT NOT NULL,
    ScenarioId INT NOT NULL,
    StartedAt DATETIME NOT NULL,
    CompletedAt DATETIME,
    IsPassed BIT NOT NULL,
    FOREIGN KEY (UserId) REFERENCES Users(Id),
    FOREIGN KEY (ScenarioId) REFERENCES Scenarios(Id)
);
```

#### Таблица `Logs`:
```sql
CREATE TABLE Logs (
    Id INT PRIMARY KEY IDENTITY(1,1),
    TestId INT NOT NULL,
    Action NVARCHAR(255) NOT NULL,
    CreatedAt DATETIME DEFAULT GETDATE(),
    FOREIGN KEY (TestId) REFERENCES Tests(Id)
);
```

---

### Пример данных для заполнения
#### Таблица `Scenarios`:
| Id | Title                        | Description                       | Complexity | CreatedAt           |
|----|------------------------------|-----------------------------------|------------|---------------------|
| 1  | Phishing Email Simulation    | Employee receives a fake email.  | 3          | 2024-11-25 10:00:00 |
| 2  | Fake IT Support Call         | Employee receives a fake call.   | 4          | 2024-11-25 10:00:00 |

#### Таблица `Users`:
| Id | Name          | Email               | Role       | PasswordHash  | CreatedAt           |
|----|---------------|---------------------|------------|---------------|---------------------|
| 1  | John Doe      | john.doe@company.com| Employee   | [hash]        | 2024-11-25 10:00:00 |
| 2  | Jane Smith    | jane.smith@admin.com| Admin      | [hash]        | 2024-11-25 10:00:00 |
