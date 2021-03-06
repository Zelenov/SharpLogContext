# SharpLogContext
![SharpLogContext Icon][SharpLogContext.icon]
</br>
Stores log data

| Name | Package | Description |
| ------------ | ----------- | ----------- |
| Main Package | [SharpLogContext][SharpLogContext.nuget] | Stores log data throught async calls and threads |
| ASP.NetCore | [SharpLogContext.NetCore][SharpLogContext.NetCore.nuget] | Stores log data throught async web request |
| MassTransit | [SharpLogContext.MassTransit][SharpLogContext.MassTransit.nuget] | Stores log data throught consumer message processing |


## Getting job done

Add `app.UseLogContext()` in the beginning of request pipeline in `Startup.cs`
```csharp
public void Configure(IApplicationBuilder app)
{
    app.UseLogContext();
    ...
}
```

Add log items anywhere in your code

```csharp
public class FooController : ControllerBase
{
    [HttpGet]
    public void Get()
    {
        LogContext.Current.AttachValue("foo", true);
    }
}
```

Access your log items anywhere you want. Unlike logging scopes, sharing values only with the underlying calls, `LogContext` items can be accessed from the outside, even in the middleware.
```csharp
public class FooMiddleware
{
    private readonly RequestDelegate _next;
    public FooMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task Invoke(HttpContext context)
    {
        await _next(context);
        var foo = LogContext.Current.GetValue("foo");
    }
}
```

## Log it!
Extract `LogContext` items when you need to log them
```csharp
using Microsoft.Extensions.Logging;
...

private readonly ILogger _logger;

...
var logContextValues = LogContext.Current.GetValues();
_logger.Log(logLevel: LogLevel.Info, eventId: 1, state: logContextValues, exception: null,
                formatter: (_, __) => "Log message");
```

## Print your log items pretty 
### NLog
`NLog.config` example
```xml
<?xml version="1.0" encoding="utf-8"?>
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <targets>
    <target name="mainLogFile" xsi:type="File"
      fileName="${basedir}/logs/${shortdate}/${logger}.json">
      <layout xsi:type="JsonLayout">
        <attribute layout="${event-properties:foo}" name="foo"/>
      </layout>
    </target>
  </targets>
  <rules>
    <logger minlevel="Trace" name="*" writeTo="mainLogFile"/>
  </rules>
</nlog>
```

## Scopes
Determine scope for particular log items, if you don't want them to be visible from the outer space.

```csharp
LogContext.Current.AttachValue("1", 1);
using (LogContext.Current.CreateScope("2", 2))
{
    Console.WriteLine(LogContext.Current.GetValues());
    //Returns {"1", 1}, {"2", 2}
}
Console.WriteLine(LogContext.Current.GetValues());
//Returns {"1", 1}
```

# SharpLogContext.MassTransit
Stores log data throughout MassTransit consumer message processing

Call `.AddLogContext()` on your `IBusControl`

```csharp
var bus = Bus.Factory.CreateUsingInMemory(...);
bus.AddLogContext();
await bus.StartAsync();
```
or on any `IBusFactoryConfigurator`
```csharp
services.AddMassTransit(x =>
{
  x.UsingInMemory((context, cfg) =>
  {
    cfg.ConfigureEndpoints(context);
    cfg.AddLogContext();
  });
}
```

And start using `LogContext`
```csharp
class FooConsumer : IConsumer<Foo>
{
    public async Task Consume(ConsumeContext<Foo> context)
    {
        LogContext.Current.AttachValue("foo", 1);
    }
}
```

[SharpLogContext.icon]: icon_small.png "SharpLogContext Icon"
[SharpLogContext.nuget]: https://www.nuget.org/packages/SharpLogContext
[SharpLogContext.NetCore.nuget]: https://www.nuget.org/packages/SharpLogContext.NetCore
[SharpLogContext.MassTransit.nuget]: https://www.nuget.org/packages/SharpLogContext.MassTransit