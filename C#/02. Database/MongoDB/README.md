# MongoDB
* [Run container](#run-container)
* [Add driver](#add-driver)
* [Add configuration](#add-configuration)
* [Add service](#add-service)
* 
### Run container
in `terminal`
```sh
docker run -d --rm \
    --name mongo \
    -p 27017:27017 \
    -v mongodbdata:/data/db \
    mongo
```
[go top](#mongodb)
### Add driver
in `terminal`
```sh
dotnet add package MongoDB.Driver
```
[go top](#mongodb)
### Add configuration
in `appsettings.Development.json`
```json
{
  "MongoDbSettings": {
    "Host": "localhost",
    "Port": "27017",
    "Collection": "app"
  },
  // ...
}
```
[go top](#mongodb)
### Add service
in `Startup.cs`
```cs
// ...

namespace mongooo
{
    public class Startup
    {
        // ...
        public void ConfigureServices(IServiceCollection services)
        {
            // ...
            // ADD THIS
            services.AddSingleton(serviceProvider => 
            {
                var mongoClient = new MongoClient($"mongodb://{Configuration["MongoDbSettings:Host"]}:{Configuration["MongoDbSettings:Port"]}");
                return mongoClient.GetDatabase(Configuration["MongoDbSettings:Collection"]);
            });
            // ...
        }
        // ...
    }
}
```
