# SQL SERVER
1. Create container
1. Install packages
1. Install `dotnet-ef` **(if you do not have it)**
1. Create class derives from `DbContext`
1. Configure service
1. Add ConnectionString
1. Update database
### Create container
in `terminal`
```sh
# IF YOU DO NOT HAVE IT
sudo docker pull mcr.microsoft.com/mssql/server:2019-latest
   
# Password is "Passw0rd!" 
sudo docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=Passw0rd'!'" \
   -p 1433:1433 --name sql1 -h sql1 \
   -d mcr.microsoft.com/mssql/server:2019-latest
```
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
	public class DataContext : DbContext
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
    "DefaultConnection": "Server=127.0.0.1;Database=app;User Id=sa;Password=Passw0rd!"
},
```
### Update database
in `terminal`
```sh
dotnet ef migrations add InitialCreate
dotnet ef database update 
```
## Resources
* [docker](https://docs.microsoft.com/en-us/sql/linux/quickstart-install-connect-docker?view=sql-server-ver15&pivots=cs1-bash)
