### Общая структура проекта

```
SocialEngineeringTester
│
├── Controllers
│   ├── ScenarioController.cs     # Контроллер для управления сценариями
│   ├── UserController.cs         # Контроллер для управления пользователями
│   └── TestController.cs         # Контроллер для проведения тестов
│
├── Models
│   ├── User.cs                   # Модель пользователя
│   ├── Scenario.cs               # Модель сценария
│   ├── Test.cs                   # Модель теста
│   └── Log.cs                    # Модель лога действий
│
├── DTOs                          # Data Transfer Objects
│   ├── UserDTO.cs                # DTO для передачи данных пользователя
│   ├── ScenarioDTO.cs            # DTO для передачи данных сценария
│   └── TestResultDTO.cs          # DTO для результата теста
│
├── Services
│   ├── Interfaces
│   │   ├── IUserService.cs       # Интерфейс сервиса пользователей
│   │   ├── IScenarioService.cs   # Интерфейс сервиса сценариев
│   │   └── ITestService.cs       # Интерфейс сервиса тестов
│   ├── UserService.cs            # Реализация сервиса пользователей
│   ├── ScenarioService.cs        # Реализация сервиса сценариев
│   └── TestService.cs            # Реализация сервиса тестов
│
├── Data
│   ├── AppDbContext.cs           # Контекст базы данных
│   ├── SeedData.cs               # Данные для инициализации
│   └── Migrations/               # Миграции базы данных
│
├── Helpers
│   ├── JwtHelper.cs              # Генерация и валидация JWT токенов
│   └── PasswordHasher.cs         # Хэширование паролей
│
├── Middleware
│   ├── LoggingMiddleware.cs      # Middleware для логирования запросов
│   └── ErrorHandlingMiddleware.cs# Middleware для обработки ошибок
│
├── Properties
│   └── launchSettings.json       # Настройки запуска проекта
│
├── appsettings.json              # Настройки конфигурации
├── Program.cs                    # Точка входа в приложение
├── Startup.cs                    # Настройка сервисов и middleware (если используется)
└── SocialEngineeringTester.csproj # Файл конфигурации проекта
```

---

### Детализация структуры

#### 1. **Controllers**
Контроллеры отвечают за обработку HTTP-запросов и передачу данных между клиентом и сервером.

- **`ScenarioController.cs`**:
  Управляет сценариями (CRUD операций).
  ```csharp
  [ApiController]
  [Route("api/[controller]")]
  public class ScenarioController : ControllerBase
  {
      private readonly IScenarioService _scenarioService;
      public ScenarioController(IScenarioService scenarioService)
      {
          _scenarioService = scenarioService;
      }

      [HttpGet]
      public IActionResult GetAllScenarios()
      {
          var scenarios = _scenarioService.GetAll();
          return Ok(scenarios);
      }
  }
  ```

- **`UserController.cs`**:
  Управляет пользователями (регистрация, авторизация).
  - Методы:
    - `Register`
    - `Login`
    - `GetUserById`

- **`TestController.cs`**:
  Управляет тестированием сотрудников.
  - Методы:
    - `StartTest`
    - `SubmitResult`
    - `GetTestResults`

---

#### 2. **Models**
Содержит классы, представляющие сущности базы данных.

- **`User.cs`**:
  ```csharp
  public class User
  {
      public int Id { get; set; }
      public string Name { get; set; }
      public string Email { get; set; }
      public string PasswordHash { get; set; }
      public string Role { get; set; } // Admin, Employee
      public DateTime CreatedAt { get; set; }
  }
  ```

- **`Scenario.cs`**:
  ```csharp
  public class Scenario
  {
      public int Id { get; set; }
      public string Title { get; set; }
      public string Description { get; set; }
      public int Complexity { get; set; }
      public DateTime CreatedAt { get; set; }
  }
  ```

- **`Test.cs`**:
  ```csharp
  public class Test
  {
      public int Id { get; set; }
      public int UserId { get; set; }
      public int ScenarioId { get; set; }
      public bool IsPassed { get; set; }
      public DateTime StartedAt { get; set; }
      public DateTime CompletedAt { get; set; }
  }
  ```

---

#### 3. **DTOs**
Используются для передачи данных между клиентом и сервером.

- **`UserDTO.cs`**:
  ```csharp
  public class UserDTO
  {
      public string Name { get; set; }
      public string Email { get; set; }
  }
  ```

- **`ScenarioDTO.cs`**:
  ```csharp
  public class ScenarioDTO
  {
      public string Title { get; set; }
      public string Description { get; set; }
      public int Complexity { get; set; }
  }
  ```

---

#### 4. **Services**
Сервисы содержат бизнес-логику приложения.

- **`IUserService.cs`**:
  Интерфейс для работы с пользователями.
  ```csharp
  public interface IUserService
  {
      User GetUserById(int id);
      void Register(User user);
      string Authenticate(string email, string password);
  }
  ```

- **`ScenarioService.cs`**:
  Логика управления сценариями.
  ```csharp
  public class ScenarioService : IScenarioService
  {
      private readonly AppDbContext _context;
      public ScenarioService(AppDbContext context)
      {
          _context = context;
      }

      public List<Scenario> GetAll()
      {
          return _context.Scenarios.ToList();
      }
  }
  ```

---

#### 5. **Data**
- **`AppDbContext.cs`**:
  Контекст базы данных с DbSet для каждой модели.
  ```csharp
  public class AppDbContext : DbContext
  {
      public DbSet<User> Users { get; set; }
      public DbSet<Scenario> Scenarios { get; set; }
      public DbSet<Test> Tests { get; set; }

      public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }
  }
  ```

---

#### 6. **Helpers**
Вспомогательные классы для работы с безопасностью.

- **`JwtHelper.cs`**:
  Генерация и валидация JWT-токенов.
  ```csharp
  public static class JwtHelper
  {
      public static string GenerateToken(User user, string secretKey)
      {
          var tokenHandler = new JwtSecurityTokenHandler();
          var key = Encoding.ASCII.GetBytes(secretKey);
          var tokenDescriptor = new SecurityTokenDescriptor
          {
              Subject = new ClaimsIdentity(new Claim[]
              {
                  new Claim(ClaimTypes.Name, user.Name),
                  new Claim(ClaimTypes.Role, user.Role)
              }),
              Expires = DateTime.UtcNow.AddHours(2),
              SigningCredentials = new SigningCredentials(new SymmetricSecurityKey(key), SecurityAlgorithms.HmacSha256Signature)
          };
          var token = tokenHandler.CreateToken(tokenDescriptor);
          return tokenHandler.WriteToken(token);
      }
  }
  ```
