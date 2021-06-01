# SQL SERVER
1. Set up database
1. Install packages
1. Install `dotnet-ef` **(if you do not have it)**
1. Create class derives from `DbContext`
1. Configure service
1. Add ConnectionString
1. Update database
### Set up database
in `terminal`
```sh
# servername: "localhost" / "127.0.0.1"
# login: "sa"
# Password is "P@ssw0rd" 
sudo docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=P@ssw0rd" \
   -p 1433:1433 --name db \
   -d mcr.microsoft.com/mssql/server:2019-latest
```
or `docker-compose`
```yml
version: '3.4'

services:
    db:
        image: mcr.microsoft.com/mssql/server:2019-latest
        container_name: db
        ports:
          - 1433:1433
        environment:
          - ACCEPT_EULA=Y
          - SA_PASSWORD=P@ssw0rd
        # optional
        # volumes:
        #  - sqlvolume1:/var/opt/mssql
# docker-compose up -d
```
or `terminal`
```sh
echo "version: '3.4'\n\nservices:\n    db:\n        image: mcr.microsoft.com/mssql/server:2019-latest\n        container_name: db\n        ports:\n         - 1433:1433\n        environment:\n         - ACCEPT_EULA=Y\n         - SA_PASSWORD=P@ssw0rd" > docker-compose.yml
docker-compose up -d
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
or `terminal`
```sh
mkdir Data &&
appname='app' &&
echo "using Microsoft.EntityFrameworkCore;\n\nnamespace $appname.Data\n{\n\tpublic class DataContext : DbContext\n\t{\n\t\tpublic DataContext(DbContextOptions options) : base(options)\n\t\t{\n\t\t}\n\t}\n}" >> Data/DataContext.cs
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
    "DefaultConnection": "Server=127.0.0.1,1433;Database=app;User Id=sa;Password=P@ssw0rd"
},
```
### Update database
in `terminal`
```sh
dotnet ef migrations add InitialCreate &&
dotnet ef database update 
```
## Resources
* [docker](https://docs.microsoft.com/en-us/sql/linux/quickstart-install-connect-docker?view=sql-server-ver15&pivots=cs1-bash)
