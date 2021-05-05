# SQL SERVER
1. Install packages
1. Install `dotnet-ef` **(if you do not have it)**
1. Create class derives from `DbContext`
1. Configure service
1. Add ConnectionString
1. Update database

### Install packages
```sh
dotnet add package Microsoft.EntityFrameworkCore.SqlServer -v 5.0.0 &&
dotnet add package Microsoft.EntityFrameworkCore.Design -v 5.0.0
```
### Install `dotnet-ef` **(if you do not have it)**
Install `dotnet-ef` by typing in your terminal
```sh
dotnet tool install --global dotnet-ef --version 5.0.0
```
### Create class derives from `DbContext`
in `Data/DataContext.cs`
```cs
using Microsoft.EntityFrameworkCore;

namespace server.Data
{
	public class DataContext: DbContext
	{
		public DataContext(DbContextOptions options) : base(options)
		{
			
		}
	}
}
```
### Configure service
in `Startup.cs`
```cs
public void ConfigureServices(IServiceCollection services)
{
    // ...
    services.AddDbContext<DataContext>(options =>
    {
        options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection"));
    });
    // ...
}
```
### Add ConnectionString
in `server/Properties`
```cs
"ConnectionStrings": {
    "DefaultConnection": "Server=127.0.0.1;Database=asp;User Id=sa;Password=CHANGE_ME_PLS"
},
```
### Update database
in `terminal`
```sh
dotnet ef migrations add InitialCreate
dotnet ef database update 
```
