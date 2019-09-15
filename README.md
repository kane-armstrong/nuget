# Event Bus

Provides abstractions for publishing events to external consumers (named "integration events" here, signifying an event 
that originates from one process and is handled in one or more others). 

## Example Usage

In Startup.cs:

````
public void ConfigureServices(IServiceCollection services)
{
	services.AddInMemorySubscriptionsManager();
}

public virtual void Configure(IApplicationBuilder app)
{
	var eventBus = app.ApplicationServices.GetRequiredService<IEventBus>();
	eventBus.Subscribe<MyIntegrationEvent, MyIntegrationEventHandler>();
}
````

# Event Bus - Service Bus

Implements the Event Bus abstractions with Azure Service Bus topics.

## Example Usage

In Startup.cs:

````
public void ConfigureServices(IServiceCollection services)
{
    services.Configure<EventBusConnectionOptions>(Configuration.GetSection(nameof(EventBusConnectionOptions)));
    services.Configure<EventBusOptions>(Configuration.GetSection(nameof(EventBusOptions)));

	services.AddServiceBusEventBus();
    services.AddDefaultServiceBusConnectionManager()
}

public virtual void Configure(IApplicationBuilder app)
{

	var eventBus = app.ApplicationServices.GetRequiredService<IEventBus>();
	eventBus.Subscribe<MyIntegrationEvent, MyIntegrationEventHandler>();
}
````

# Event Bus - Integration Events

Persisted store for recording outgoing integration events, using Entity Framework. This adds resilience and provides
an option for building management APIs or interfaces, e.g. if you need to display a history of events, their statuses,
etc.

## Example Usage

In Startup.cs:

````
public void ConfigureServices(IServiceCollection services)
{
    services.AddIntegrationEventLogContext(options =>
    {
        options.ConfigureDbContext = db =>
        {
            db.UseSqlServer(configuration.GetAccountsIntegrationEventsDbConnectionString(),
                sql =>
                {
                    sql.MigrationsAssembly(typeof(IntegrationEventLogContext).Assembly.FullName);
                });
        };
        options.DefaultSchema = "accounts";
    });
}
````