# Migration

Set your project up for the DB migration.

## Prerequisites

0. Have a properly designed OnModelCreating() method in your application DbContext class

## Steps

1. Install Microsoft.EntityFrameworkCore.Design package (mine is installed in the API project)
 ```bash
dotnet add package Microsoft.EntityFrameworkCore.Design
 ```

2. Create a migration using next command:
```bash
 dotnet ef migrations add InitialCreate --project TalentHub.UserService.Infrastructure --startup-project TalentHub.UserService.Api
 ```

3. To perform an automigration every application run add the next code snippet to the application configuration class (`Program.cs`):
```cs
using (var scope = app.Services.CreateScope())
{
    var dbContext = scope.ServiceProvider.GetRequiredService<UserDbContext>();
    var logger = scope.ServiceProvider.GetRequiredService<ILogger<Program>>();
    
    logger.LogInformation("Applying migrations...");
    dbContext.Database.Migrate();
    logger.LogInformation("Migrations applied successfully.");
}
```

*Note:* Make sure to disable this code if you need to run the application locally


# Container deployment

## Prerequisites

1. Install Podman or Docker on your computer

## Steps

2. Place a `Dockerfile` in the root of the startup project of the solution

3. Build and publish every application you'd like to deploy using next command:
```bash
podman build --no-cache -t talenthub.userservice.api:dev -f TalentHub.UserService.Api/Dockerfile .
```

4. Copy the`podman-compose.yml` script to your machine

5. Run all containers built previously in the directory with the `podman-compose.yml` by next command:
```bash
podman-compose up
```

Congratulations! All containers are up to work! 


# Nota bene! 

Remember to set the `podman-compose.yml` properly. 
If your application has some local settings - override them using the environment variables. 

## Example configuration

1. The following example redefines `ApplicationConfiguration:ConnectionString`, `RabbitMqConfiguration:Host` and `RabbitMqConfiguration:QueueName`
```yaml
  userservice:
    image: localhost/talenthub.userservice.api:dev
    container_name: userservice
    ports:
      - "8080:8080"
    environment:
      ASPNETCORE_ENVIRONMENT: "Development"
      ApplicationConfiguration__ConnectionString: "Host=userdb;Database=UserServiceDb;Username=user;Password=password"
      RabbitMqConfiguration__Host: "rabbitmq"
      RabbitMqConfiguration__QueueName: "lotusbyte.talenthub.notification.req"
    depends_on:
      - userdb
      - rabbitmq
    networks:
      - talenthub-net
```

2. Remember to add the following line to your `Program.cs` to apply the environment settings you set up before (should be placed before the configuration binding):
```cs
 builder.Configuration.AddEnvironmentVariables();
```

3. Keep in mind the ports you're setting up: to access your application's web UI outside the container, use localhost with the port specified in the `.yml` file, e.g.

in .yml 
```yaml
     ports:
      - 8082:80
```

in the browser 
```bash
localhost:8082
```

4. shut your containers down when they're no longer needed using
```bash
podman-compose down
```
