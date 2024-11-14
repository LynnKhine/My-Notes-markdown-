# Csharp connecting with the database

There is two methods:
- [1. DB First](#db-first) and
- [2. Code First](#code-first)

# DB First

You need to configure all these three ways
- Connection String
- DB context(DBset)

### Connection String
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=your_server_name;Database=your_database_name;User Id=your_username;Password=your_password;Trusted_Connection=False;MultipleActiveResultSets=true;"
  }
}
```

### Add builder configuration in Program.cs

You need to paste this above `builder.Services.AddControllers();`

```csharp
// Register DbContext with MSSQL connection string (db context)
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection"))); (edited)
```

### Create DBContext (DBset)

Create a new file under new folder and this file is to keep entity names

```csharp
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    // Define your tables here as DbSet<T>
    public DbSet<YourEntity> YourEntities { get; set; }

    // You can override OnModelCreating if you need additional configuration
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        // Additional configuration here if needed
    }
}
```

# Code First

