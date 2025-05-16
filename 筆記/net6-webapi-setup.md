# .NET 6+ Web API Setup

> **Version**: 2025-05-16

---

## 1. Program.cs 範例

```csharp
var builder = WebApplication.CreateBuilder(args);

// 1. Configuration / Logging
builder.Host.ConfigureAppConfiguration((ctx, cfg) =>
{
    cfg.AddJsonFile("appsettings.Local.json", optional: true, reloadOnChange: true);
})
.ConfigureLogging((ctx, log) =>
{
    log.ClearProviders();
    log.AddConsole();
    log.AddDebug();
});

// 2. Services Registration
builder.Services
    .AddControllers()
    .AddEndpointsApiExplorer()
    .AddSwaggerGen()
    .AddCors(options =>
    {
        options.AddPolicy("AllowAll", policy =>
            policy.AllowAnyOrigin()
                  .AllowAnyMethod()
                  .AllowAnyHeader());
    })
    // Authentication & Authorization
    .AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer           = true,
            ValidateAudience         = true,
            ValidateLifetime         = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer              = builder.Configuration["Jwt:Issuer"],
            ValidAudience            = builder.Configuration["Jwt:Audience"],
            IssuerSigningKey         = new SymmetricSecurityKey(
                                          Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]))
        };
    })
    .Services.AddAuthorization();

// 3. EF Core with Stored Procedures
builder.Services.AddDbContext<MyDbContext>(opts =>
    opts.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();

// 4. Middleware Pipeline
app.UseHttpsRedirection();

app.UseCors("AllowAll");

app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();
app.Run();
```

---

## 2. appsettings.json 範例

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=YOUR_SERVER;Database=YOUR_DB;User Id=YOUR_USER;Password=YOUR_PASSWORD;"
  },
  "Jwt": {
    "Issuer": "YourIssuer",
    "Audience": "YourAudience",
    "Key": "YourSuperSecretKey123456!"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  }
}
```

### 使用 Stored Procedure

在 `MyDbContext` 中：

```csharp
public class MyDbContext : DbContext
{
    public MyDbContext(DbContextOptions<MyDbContext> options)
        : base(options) { }

    public DbSet<MyEntity> MyEntities { get; set; }

    public async Task<List<MyEntity>> GetEntitiesByProcAsync(int param)
    {
        return await MyEntities
            .FromSqlRaw("EXEC dbo.MyStoredProcedure @Param = {0}", param)
            .ToListAsync();
    }
}
```

---

## 3. `app.UseAuthentication()` 與 `app.UseAuthorization()`

- **app.UseAuthentication()**  
  - 解析並驗證請求中的認證資訊（如 `Authorization: Bearer {token}`）。  
  - 如果 JWT 有效，將 ClaimsPrincipal 塞入 `HttpContext.User`。  
  - 若無或驗證失敗，不會自動回應 401，但後續 `[Authorize]` 會拒絕存取。

- **app.UseAuthorization()**  
  - 根據已設定的授權策略、角色、Policy 來決定是否允許當前 `HttpContext.User` 訪問資源。  
  - 在中介軟體管線中需放在 `UseAuthentication` 之後、`MapControllers` 之前。

### JWT 範例流程

1. **Client 請求**  
   ```
   GET /api/values
   Authorization: Bearer eyJhbGciOiJIUzI1Ni...
   ```

2. **UseAuthentication**  
   - 解碼 JWT → 驗證簽章、Issuer、Audience、Expire。  
   - 若成功，User.Identity.IsAuthenticated = true。

3. **UseAuthorization**  
   - `[Authorize]` 標記的 Controller/Action 會檢查 `User` 是否符合要求。  
   - 不符合 → 403 Forbidden，或 401 Unauthorized。

4. **Controller 執行**  
   ```csharp
   [Authorize]
   [ApiController]
   [Route("api/[controller]")]
   public class ValuesController : ControllerBase
   {
       [HttpGet]
       public IActionResult Get()
       {
           var userId = User.FindFirstValue(ClaimTypes.NameIdentifier);
           return Ok(new { Data = "Protected data", User = userId });
       }
   }
   ```
```