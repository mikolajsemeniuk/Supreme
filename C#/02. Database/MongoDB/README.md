# MongoDB
* [Run container](#run-container)
* [Add driver](#add-driver)
* [Add configuration](#add-configuration)
* [Add service](#add-service)
* [Create service](#create-service)
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
[go top](#mongodb)
### Create service
in `Services/TodoService.cs`
```cs
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using MongoDB.Driver;
using mongooo.Entities;
using mongooo.Interfaces;

namespace mongooo.Services
{
    public class TodoService : ITodoService
    {
        private const string collectionName = "Todos";
        private readonly IMongoCollection<Todo> dbCollection;
        private readonly FilterDefinitionBuilder<Todo> filterBuilder = Builders<Todo>.Filter;

        public TodoService(IMongoDatabase database) =>
            dbCollection = database.GetCollection<Todo>(collectionName);

        public async Task<IEnumerable<Todo>> GetTodosAsync() =>
            await dbCollection.Find(filterBuilder.Empty).ToListAsync();

        public async Task<Todo> GetTodoAsync(Guid id) =>
            await dbCollection.Find(filterBuilder.Eq(todo => todo.Id, id)).FirstOrDefaultAsync();

        public async Task<Todo> AddTodoAsync()
        {
            var todo = new Todo
            {
                Title = "hi",
                Description = "there",
                IsDone = true
            };
            await dbCollection.InsertOneAsync(todo);
            return todo;
        }

        public async Task<Todo> SetTodoAsync(Guid id)
        {
            var todo = await dbCollection.Find(filterBuilder.Eq(todo => todo.Id, id)).FirstOrDefaultAsync();
            todo.Title = "heh";
            todo.Description = "heheheh";
            todo.IsDone = false;
            await dbCollection.ReplaceOneAsync(filterBuilder.Eq(todo => todo.Id, id), todo);
            return todo;
        }

        public async Task RemoveTodoAsync(Guid id) =>
            await dbCollection.DeleteOneAsync(filterBuilder.Eq(todo => todo.Id, id));
    }
}
```
