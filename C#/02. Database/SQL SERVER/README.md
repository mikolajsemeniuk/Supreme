# SQL SERVER
1. Install packages
1. Install `dotnet-ef` **(if you do not have it)**
1. Create Class derives from DbContext
1. Configure service
1. Add ConnectionString
1. Update database

# Install packages
```sh
dotnet add package Microsoft.EntityFrameworkCore.SqlServer -v 5.0.0 &&
dotnet add package Microsoft.EntityFrameworkCore.Design -v 5.0.0
```
### Install `dotnet-ef` **(if you do not have it)**
Install `dotnet-ef` by typing in your terminal
```sh
dotnet tool install --global dotnet-ef --version 5.0.0
```
