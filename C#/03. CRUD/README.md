# CRUD - HTTP
* Allow CORS
* Create custom exception page
* Create model
* Modify DbContext
* Create migration and update db
* Create input and payload
* Create interface
* Create repository
* Register service
* Create controller
* [Seed database](#seed-database)
### Allow CORS
in `Startup.cs`
```cs
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // ...
    app.UseHttpsRedirection();

    // JWT
    app.UseCors(options => options
        .AllowAnyHeader()
        .AllowAnyMethod()
        .AllowAnyOrigin());
	
    // cookie	
    app.UseCors(options => options
        .AllowAnyHeader()
        .AllowAnyMethod()
        .WithOrigins("https://localhost:4200")
        .SetIsOriginAllowed(origin => true)
        .AllowCredentials());

    app.UseRouting();

    app.UseAuthorization();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
    });
}
```
### Create custom exception page
in `Exceptions/ApiException.cs`
```cs
namespace server.Exceptions
{
    public class ApiException
    {
        public ApiException(string message, string details = null)
        {
            Message = message;
            Details = details;

        }
        public string Message { get; set; }
        public string Details { get; set; }
    }
}
```
in `Middlewares/ExceptionMiddleware.cs`
```cs
using System;
using System.Net;
using System.Text.Json;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.Logging;
using server.Exceptions;

namespace server.Middlewares
{
    public class ExceptionMiddleware
    {
        public readonly RequestDelegate _next;
        private readonly ILogger<ExceptionMiddleware> _logger;
        public readonly IHostEnvironment _env;

        public ExceptionMiddleware(RequestDelegate next, ILogger<ExceptionMiddleware> logger, IHostEnvironment env)
        {
            _env = env;
            _logger = logger;
            _next = next;
        }

        public async Task InvokeAsync(HttpContext context)
        {
            try
            {
                await _next(context);
            }
            catch (Exception exception)
            {
                string message;
                int statusCode;
                if (exception.Message.Contains("[Code]"))
                {
                    var text = exception.Message.Split("[Code] ");
                    message = text[0];
                    statusCode = int.Parse(text[1]);
                }
                else
                {
                    message = exception.Message;
                    statusCode = (int) HttpStatusCode.BadRequest;
                }

                _logger.LogError(exception, exception.Message);
                context.Response.ContentType = "application/json";
                context.Response.StatusCode = statusCode;

                var response = _env.IsDevelopment()
                    ? new ApiException(message, exception.StackTrace?.ToString())
                    : new ApiException(message);

                var options = new JsonSerializerOptions { PropertyNamingPolicy = JsonNamingPolicy.CamelCase };

                var json = JsonSerializer.Serialize(response, options);

                await context.Response.WriteAsync(json);
            }
        }
    }
}
```
in `Startup.cs`
```cs
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
         //app.UseDeveloperExceptionPage();
         app.UseSwagger();
         app.UseSwaggerUI(c => c.SwaggerEndpoint("/swagger/v1/swagger.json", "test v1"));
    }
    app.UseMiddleware<ExceptionMiddleware>();
    
    app.UseHttpsRedirection();
    // ...
}
```
### Create model
in `Models/Todo.cs`
```cs
using System;

namespace server.Models
{
	public class Todo
    	{
		public int Id { get; set; }
		public string Title { get; set; }
		public string Description { get; set; }
		public DateTime Created { get; set; }
		public DateTime Updated { get; set; }
		public bool IsDone { get; set; }
    	}
}
```
### Modify DbContext
in `Data/DataContext.cs`
```cs
using Microsoft.EntityFrameworkCore;
using server.Models;

namespace server.Data
{
	public class DataContext: DbContext
	{
		public DataContext(DbContextOptions options) : base(options)
		{
		}
        	public DbSet<Todo> Todos { get; set; }
	}
}
```
### Create migration and update db
```sh
dotnet ef database drop &&
dotnet ef migrations add Todos &&
dotnet ef database update 
```
### Create input and payload
in `DTO/TodoInput.cs`
```cs
using System.ComponentModel.DataAnnotations;

namespace server.DTO
{
	public class TodoInput
	{
		[Required(ErrorMessage = "Title is required.")]
		[StringLength(25, MinimumLength = 4, ErrorMessage = "Title must have min length of 4 and max Length of 25")]
		public string Title { get; set; }
		[Required(ErrorMessage = "Description is required.")]
		[StringLength(255, MinimumLength = 8, ErrorMessage = "Description must have min length of 8 and max Length of 255")]
		public string Description { get; set; }
		[Required(ErrorMessage = "IsDone is required.")]
		public bool IsDone { get; set; }
	}
}
```
in `DTO/TodoPayload.cs`
```cs
using System;

namespace server.DTO
{
	public class TodoPayload
	{
		public int Id { get; set; }
		public string Title { get; set; }
		public string Description { get; set; }
		public DateTime Created { get; set; }
		public DateTime Updated { get; set; }
		public bool IsDone { get; set; }
	}
}
```
### Create interface
in `Interfaces/ITodoRepository.cs`
```cs
using System.Collections.Generic;
using System.Threading.Tasks;
using server.DTO;

namespace server.Interfaces
{
    public interface ITodoRepository
    {
        Task<IEnumerable<TodoPayload>> GetTodosAsync();
        Task<TodoPayload> GetTodoAsync(int id);
        Task<TodoPayload> AddTodoAsync(TodoInput input);
        Task<TodoPayload> SetTodoAsync(int id, TodoInput input);
        Task<TodoPayload> RemoveTodoAsync(int id);
        Task CheckIfAllSavedAsync();
    }
}
```
### Create repository
in `Services/TodoRepository.cs`
```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.EntityFrameworkCore;
using server.Data;
using server.DTO;
using server.Interfaces;
using server.Models;

namespace server.Services
{
    public class TodoRepository : ITodoRepository
    {
        private readonly DataContext _context;

        public TodoRepository(DataContext context) =>
            _context = context;

        public async Task<IEnumerable<TodoPayload>> GetTodosAsync() =>
            await _context.Todos
                .Select(todo => new TodoPayload { Id = todo.Id, Title = todo.Title, Description = todo.Description, Created = todo.Created, Updated = todo.Updated, IsDone = todo.IsDone })
                .ToListAsync();

        public async Task<TodoPayload> GetTodoAsync(int id) =>
            await _context.Todos
                .Where(todo => todo.Id == id)
                .Select(todo => new TodoPayload { Id = todo.Id, Title = todo.Title, Description = todo.Description, Created = todo.Created, Updated = todo.Updated, IsDone = todo.IsDone })
                .SingleAsync();

        public async Task<TodoPayload> AddTodoAsync(TodoInput input)
        {
            var todo = new Todo { Title = input.Title, Description = input.Description, IsDone = input.IsDone };
            await _context.Todos.AddAsync(todo);
            await CheckIfAllSavedAsync();
            return new TodoPayload
            { Id = todo.Id, Title = todo.Title, Description = todo.Description, Created = todo.Created, Updated = todo.Updated, IsDone = todo.IsDone };
        }

        public async Task<TodoPayload> SetTodoAsync(int id, TodoInput input)
        {
            var todo = await _context.Todos.FindAsync(id);
            if (todo == null)
                throw new Exception("Could not find an item with id: " + id + "[Code] 404");
            todo.Title = input.Title;
            todo.Description = input.Description;
            todo.IsDone = input.IsDone;
            todo.Updated = DateTime.Now;
            _context.Todos.Update(todo);
            await CheckIfAllSavedAsync();
            return new TodoPayload
            { Id = todo.Id, Title = todo.Title, Description = todo.Description, Created = todo.Created, Updated = todo.Updated, IsDone = todo.IsDone };
        }

        public async Task<TodoPayload> RemoveTodoAsync(int id)
        {
            var todo = await _context.Todos.FindAsync(id);
            if (todo == null)
                throw new Exception("Could not find an item with id: " + id + "[Code] 404");
            _context.Todos.Remove(todo);
            await CheckIfAllSavedAsync();
            return new TodoPayload
            { Id = todo.Id, Title = todo.Title, Description = todo.Description, Created = todo.Created, Updated = todo.Updated, IsDone = todo.IsDone };
        }

        public async Task CheckIfAllSavedAsync()
        {
            if (!(await _context.SaveChangesAsync() > 0))
                throw new Exception("something gone wrong");
        }
    }
}
```
### Register service
in `Startup.cs`
```cs
public void ConfigureServices(IServiceCollection services)
{
    // ...
    services.AddScoped<ITodoRepository, TodoRepository>();
    // ...
}
```
### Create controller
in `Controllers/TodoController.cs`
```cs
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using server.DTO;
using server.Interfaces;

namespace server.Controllers
{
    [ApiController]
    [Route("[controller]")]
    public class TodoController : ControllerBase
    {
        private ITodoRepository _repository;

        public TodoController(ITodoRepository repository) =>
            _repository = repository;

        [HttpGet]
        public async Task<ActionResult<IEnumerable<TodoPayload>>> GetTodos() =>
            Ok(await _repository.GetTodosAsync());

        [HttpPost]
        public async Task<ActionResult<TodoPayload>> AddTodo([FromBody] TodoInput input) =>
            Ok(await _repository.AddTodoAsync(input));

        [HttpGet("{id}")]
        public async Task<ActionResult<TodoPayload>> GetTodo([FromRoute] int id) =>
            Ok(await _repository.GetTodoAsync(id));

        [HttpPut("{id}")]
        public async Task<ActionResult<TodoPayload>> SetTodo([FromRoute] int id, [FromBody] TodoInput input) =>
            Ok(await _repository.SetTodoAsync(id, input));

        [HttpDelete("{id}")]
        public async Task<ActionResult<TodoPayload>> DeleteTodo([FromRoute] int id) =>
            Ok(await _repository.RemoveTodoAsync(id));
    }
}
```
### Seed database
[go back](#crud---http)

in `appsettings.Development.json`
```json
"SeedDatabase": {
    "Enable": true
},
```
in `Program.cs`
```cs
using System;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.Logging;
using server.Data;

namespace server
{
    public class Program
    {
        public static async Task Main(string[] args)
        {
            var host = CreateHostBuilder(args).Build();
            using var scope = host.Services.CreateScope();
            var services = scope.ServiceProvider;
            try
            {
                var context = services.GetRequiredService<DataContext>();
                var config = services.GetRequiredService<IConfiguration>();

                await Seed.InitSeed(config, context);
            }
            catch (Exception ex)
            {
                var logger = host.Services.GetService<ILogger<Program>>();
                logger.LogError($"error: {ex}");
            }
            await host.RunAsync();
        }

        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                });
    }
}
```
in `Data/Seed.cs`
```cs
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Configuration;
using server.Models;

namespace server.Data
{
    public class Seed
    {
        public static async Task InitSeed(IConfiguration config, DataContext context)
        {
            if (!bool.Parse(config["SeedDatabase:Enable"]))
                return;
            await context.Database.EnsureDeletedAsync();
            await context.Database.MigrateAsync();
            await context.Database.EnsureCreatedAsync();
            await context.Todos.AddRangeAsync(GetTodos());
            await context.SaveChangesAsync();
        }
        private static List<Todo> GetTodos() =>
            new List<Todo> 
            { 
                new Todo { Title = "one", Description = "first", IsDone = false },
                new Todo { Title = "two", Description = "second", IsDone = true }
            };
    }
}
```
## Tests
* https://localhost:5001/todo
* https://localhost:5001/todo/{id}
```json
{
	"Title": "docker",
	"Description": "lorem ipsum 2",
	"IsDone": true
}
```
