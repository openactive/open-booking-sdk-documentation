# Day 1: Fake Implementation

## **Objective for Day 1**

Create a fully working implementation of the [Open Booking API](https://www.openactive.io/open-booking-api/EditorsDraft/) by using [FakeDatabase](https://www.nuget.org/packages/OpenActive.FakeDatabase.NET/) within your application.

### Rationale

Having a fully working implementation allows you to easily see what needs to be achieved, and also provides an easy way to ensure that the test suite and libraries are behaving as expected within your environment.

## **Step 1 - Copy files**

Copy the files within the `Feeds`, `Helpers`, `IdComponents`, `Settings`, and `Stores` directories from the [example project](https://github.com/openactive/OpenActive.Server.NET/tree/master/Examples/BookingSystem.AspNetCore) into your application, and add the dependencies [OpenActive.Server.NET](https://www.nuget.org/packages/OpenActive.Server.NET/) and [OpenActive.FakeDatabase.NET](https://www.nuget.org/packages/OpenActive.FakeDatabase.NET/). FakeDatabase.NET is an in-memory fake database that is not persisted, and will allow you to create a quick working Open Booking API.

Inspect the `Controllers` and copy the files \(or the contents of the files\) as appropriate for your application.

Note you will be creating the following endpoints \(as per the [Open Booking API specification](https://www.openactive.io/open-booking-api/EditorsDraft/#paths-and-verbs)\):

* Dataset Site
* Open Data RPDE feeds
* OrderQuote Creation \(C1\)
* OrderQuote Creation \(C2\)
* OrderQuote Deletion
* Order Creation \(B\)
* Order Deletion
* Order Cancellation
* Orders RPDE Feed
* Order Status

Additionally you will be creating three endpoints for use with the [OpenActive Test Suite](https://github.com/openactive/openactive-test-suite/) that implement the [Open Booking API Test Interface](https://openactive.io/test-interface/) \(not for use in production\):

* Delete Test Dataset
* Create Test Opportunity
* Execute Action

A further endpoint is required to meet the [recommendations outlined for authentication](day-8-authentication.md), however this can be added as part of Day 8 as it has it is not a dependency of Days 1-7: 

* Dynamic Client Update

## Step 2 - Copy configuration

Inspect the `Startup.cs` \(.NET Core\) or `ServiceConfig.cs` \(.NET Framework\) and copy the `services.AddSingleton<IBookingEngine>(...)` configuration into your own `Startup.cs` or `ServiceConfig.cs`.

The initial objective is to get a working version of the Open Booking API using entirely fake data. So when copying these files do not modify the configuration values at this stage - simply use them as-is.

## Step 2 - Controller endpoint bindings

The `ResponseContent` class provides a .NET version agnostic representation of HTTP responses from the Booking Engine. Two example helper methods are provided to be used with your version of .NET. See `ResponseContentHelper.cs`in the example projects.

### .NET Core

The following extension method can be used to return an `Mvc.ContentResult`.

```bash
public static Microsoft.AspNetCore.Mvc.ContentResult GetContentResult(this OpenActive.Server.NET.OpenBookingHelper.ResponseContent response)
{
    return new Microsoft.AspNetCore.Mvc.ContentResult
    {
        StatusCode = (int)response.StatusCode,
        Content = response.Content,
        ContentType = response.ContentType
    };
}
```

### .NET Framework

The following extension method can be used to return an `HttpResponseMessage`.

```bash
public static HttpResponseMessage GetContentResult(this OpenActive.Server.NET.OpenBookingHelper.ResponseContent response)
{
    var resp = new HttpResponseMessage
    {
        Content = response.Content == null ? null : new StringContent(response.Content),
        StatusCode = response.StatusCode
    };
    resp.Content.Headers.ContentType = MediaTypeHeaderValue.Parse(response.ContentType);
    return resp;
}
```

## **Step 3 - Input Formatter**

The Booking Engine accepts input JSON as `string` in order to fully control deserialisation. In order to allow the web framework to capture the body of the request as a `string`, for the [Open Booking API's media type](https://www.openactive.io/open-booking-api/EditorsDraft/#media-types), an `InputFormatter` is required.  See `OpenBookingInputFormatter.cs`in the example projects.

### .NET Core

```csharp
services
    .AddMvc()
    .AddMvcOptions(options => options.InputFormatters.Insert(0, new OpenBookingInputFormatter()))
```

```csharp
public class OpenBookingInputFormatter : InputFormatter
{
    public OpenBookingInputFormatter()
    {
        this.SupportedMediaTypes.Add(OpenActiveMediaTypes.OpenBooking.Version1);
    }

    public override async Task<InputFormatterResult> ReadRequestBodyAsync(InputFormatterContext context)
    {
        var request = context.HttpContext.Request;
        using (var reader = new StreamReader(request.Body))
        {
            var content = await reader.ReadToEndAsync();
            return await InputFormatterResult.SuccessAsync(content);
        }
    }

    protected override bool CanReadType(Type type)
    {
        return type == typeof(string);
    }
}
```

### .NET Framework

```csharp
config.Formatters.Add(new OpenBookingInputFormatter());
```

```csharp
public class OpenBookingInputFormatter : MediaTypeFormatter
{
    public OpenBookingInputFormatter()
    {
        this.SupportedMediaTypes.Add(MediaTypeHeaderValue.Parse(OpenActiveMediaTypes.OpenBooking.Version1));
    }

    public override Task<object> ReadFromStreamAsync(Type type, Stream readStream, HttpContent content, IFormatterLogger formatterLogger)
    {
        var taskCompletionSource = new TaskCompletionSource<object>();
        try
        {
            var memoryStream = new MemoryStream();
            readStream.CopyTo(memoryStream);
            var s = System.Text.Encoding.UTF8.GetString(memoryStream.ToArray());
            taskCompletionSource.SetResult(s);
        }
        catch (Exception e)
        {
            taskCompletionSource.SetException(e);
        }
        return taskCompletionSource.Task;
    }

    public override bool CanReadType(Type type)
    {
        return type == typeof(string);
    }

    public override bool CanWriteType(Type type)
    {
        return false;
    }
}
```

## **Step 4 - Authentication Helper**

The Booking Engine does not handle authentication, and instead accepts claims from an authentication layer in front of it \(this will be covered in more detail in [Day 8](day-8-authentication.md)\). An AuthenticationHelper is provided for .NET Core and .NET Framework that extracts OAuth 2.0 claims from access tokens, and also provides simple header alternatives to facilitate development and testing.

Note that these test header alternatives are not secure and **must not** be used in production, however they will be used for Day 1-7 of this guide.

### .NET Core

.NET Core implementations can leverage the authentication middleware, using the provided helpers:

```csharp
(string clientId, Uri sellerId) = User.GetAccessTokenOpenBookingClaims();
return bookingEngine.ProcessCheckpoint1(clientId, sellerId, uuid, orderQuote).GetContentResult();                
```

To speed development, the following allows test headers to be used for accessing the API, and can be set up to allow development to proceed until Day 8 of this tutorial.

{% code title="Startup.cs" %}
```csharp
public void ConfigureServices(IServiceCollection services)
{
    // DO NOT USE THIS IN PRODUCTION.
    // This passes test API headers straight through to claims, and provides no security whatsoever.
    services.AddAuthentication(options =>
    {
        options.DefaultAuthenticateScheme = TestHeaderAuthenticationOptions.DefaultScheme;
        options.DefaultChallengeScheme = TestHeaderAuthenticationOptions.DefaultScheme;

    })
    .AddTestHeaderAuthenticationSupport(options => { });
    services.AddAuthorization(options =>
    {
        // No authorization checks are performed, this just ensures that the required claims are supplied
        options.AddPolicy(OpenActiveScopes.OpenBooking, policy => {
            policy.RequireClaim(OpenActiveCustomClaimNames.ClientId);
            policy.RequireClaim(OpenActiveCustomClaimNames.SellerId);
        });
        options.AddPolicy(OpenActiveScopes.OrdersFeed, policy => policy.RequireClaim(OpenActiveCustomClaimNames.ClientId));
    });
}
```
{% endcode %}

{% code title="TestHeaderAuthentication.cs" %}
```csharp
/**
* DO NOT USE THIS FILE IN PRODUCTION. This approach is for testing only, and provides no security whatsoever.
*/

public class TestHeaderAuthenticationOptions : AuthenticationSchemeOptions
{
    public const string DefaultScheme = "Test Headers";
    public string Scheme => DefaultScheme;
    public string AuthenticationType = DefaultScheme;
}

public static class AuthenticationBuilderExtensions
{
    public static AuthenticationBuilder AddTestHeaderAuthenticationSupport(this AuthenticationBuilder authenticationBuilder, Action<TestHeaderAuthenticationOptions> options)
    {
        return authenticationBuilder.AddScheme<TestHeaderAuthenticationOptions, TestHeaderAuthenticationHandler>(TestHeaderAuthenticationOptions.DefaultScheme, options);
    }
}

public class TestHeaderAuthenticationHandler : AuthenticationHandler<TestHeaderAuthenticationOptions>
{
    public TestHeaderAuthenticationHandler(
        IOptionsMonitor<TestHeaderAuthenticationOptions> options,
        ILoggerFactory logger,
        UrlEncoder encoder,
        ISystemClock clock) : base(options, logger, encoder, clock)
    {
    }

    protected override async Task<AuthenticateResult> HandleAuthenticateAsync()
    {
        // Get the claims from headers if they exist
        Request.Headers.TryGetValue(AuthenticationTestHeaders.ClientId, out var testClientId);
        Request.Headers.TryGetValue(AuthenticationTestHeaders.SellerId, out var testSellerId);
        var clientId = testClientId.FirstOrDefault();
        var sellerId = testSellerId.FirstOrDefault();

        // This just passes the test headers through to the claims - it does not provide any security.
        var claims = new List<Claim>();
        if (clientId != null) claims.Add(new Claim(OpenActiveCustomClaimNames.ClientId, clientId));
        if (sellerId != null) claims.Add(new Claim(OpenActiveCustomClaimNames.SellerId, sellerId));

        var identity = new ClaimsIdentity(claims, Options.AuthenticationType);
        var identities = new List<ClaimsIdentity> { identity };
        var principal = new ClaimsPrincipal(identities);
        var ticket = new AuthenticationTicket(principal, Options.Scheme);

        // No checks are made, so this always succeeds. It's just setting the claims if they exist.
        return AuthenticateResult.Success(ticket);
    }

    protected override async Task HandleChallengeAsync(AuthenticationProperties properties)
    {

        Response.StatusCode = 401;
        await Response.WriteAsync(OpenActiveSerializer.Serialize(new InvalidAPITokenError()));
    }

    protected override async Task HandleForbiddenAsync(AuthenticationProperties properties)
    {
        Response.StatusCode = 403;
        await Response.WriteAsync(OpenActiveSerializer.Serialize(new UnauthenticatedError()));
    }
}
```
{% endcode %}

### .NET Framework

```csharp
(string clientId, Uri sellerId) = AuthenticationHelper.GetIdsFromAuth(Request, User);
return _bookingEngine.ProcessCheckpoint1(clientId, sellerId, uuid, orderQuote).GetContentResult();
```

```csharp
public static class AuthenticationHelper
{
    private static (string clientId, Uri sellerId) GetIdsFromAuth(HttpRequestMessage request, IPrincipal principal, bool requireSellerId)
    {
        // NOT FOR PRODUCTION USE: Please remove this block in production
        IEnumerable<string> testSellerId = null;
        if (request.Headers.TryGetValues(AuthenticationTestHeaders.ClientId, out IEnumerable<string> testClientId)
            && testClientId.Count() == 1
            && (!requireSellerId || (request.Headers.TryGetValues(AuthenticationTestHeaders.SellerId, out testSellerId) && testSellerId.FirstOrDefault().ParseUrlOrNull() != null))
            )
        {
            return (testClientId.FirstOrDefault(), testSellerId?.FirstOrDefault().ParseUrlOrNull());
        }

        // For production use: Get Ids from JWT
        var claimsPrincipal = principal as ClaimsPrincipal;
        var clientId = claimsPrincipal.GetClientId();
        var sellerId = claimsPrincipal.GetSellerId().ParseUrlOrNull();
        if (clientId != null && (sellerId != null || !requireSellerId))
        {
            return (clientId, sellerId);
        }
        else
        {
            throw new OpenBookingException(new InvalidAPITokenError());
        }
    }

    public static (string clientId, Uri sellerId) GetIdsFromAuth(HttpRequestMessage request, IPrincipal principal)
    {
        return GetIdsFromAuth(request, principal, true);
    }

    public static string GetClientIdFromAuth(HttpRequestMessage request, IPrincipal principal)
    {
        return GetIdsFromAuth(request, principal, false).clientId;
    }
}
```

## **Step 5 - Run Application**

If you run your application and navigate to the dataset site endpoint \(e.g. [https://localhost:44307/openactive](https://localhost:44307/openactive)\), you should find the following page:

![Dataset Site Default Configuration](../../.gitbook/assets/screenshot-2019-11-26-at-15.51.35.png)

Clicking on the ScheduledSessions feed should return JSON in the following format:

![ScheduledSessions feed from FakeDatabase](../../.gitbook/assets/screenshot-2019-11-26-at-15.52.44.png)

{% hint style="warning" %}
If some links appear broken, try updating `BaseUrl` within `Startup.cs` or `ServiceConfig.cs` with the correct port number for your local test environment.
{% endhint %}

Navigating to the Orders feed \(e.g. [https://localhost:44307/api/openbooking/orders-rpde](https://localhost:44307/api/openbooking/orders-rpde)\) should return the following JSON result \(an empty RPDE page\):

![Empty Orders feed](../../.gitbook/assets/screenshot-2019-11-26-at-15.54.59.png)

## **Step 6 - Run Test Suite**

The Booking Engine includes full support for the [Open Booking API Test Interface](https://openactive.io/test-interface/), so running the [OpenActive Test Suite](https://github.com/openactive/openactive-test-suite/) in 'Controlled' mode is recommended.

Follow the instructions below to set up the OpenActive Test Suite:

{% embed url="https://developer.openactive.io/open-booking-api/test-suite" %}

You will need [Node.js](https://nodejs.org/en/) version 14 or above installed to do this - which can installed with the Visual Studio installer.

In Steps 7 and 8, the header configuration can use the default values in order to work with the Booking Engine, except that the Seller `@id` must be replaced with a valid Seller `@id` from your booking system.

In Step 9, your Dataset Site is automatically created and configured by the Booking Engine, so simply update the value of `datasetSiteUrl` based on the port number and path used by your .NET application when running:

{% code title="./config/dev.json \(extract\)" %}
```javascript
  "datasetSiteUrl": "https://localhost:44307/openactive"
```
{% endcode %}

At this stage we are still using [FakeDatabase](https://www.nuget.org/packages/OpenActive.FakeDatabase.NET/), so all tests for whichever 'implemented' features you have configured should pass. Assuming configuration values have not been changed from the copied files, issues with the tests at this stage will very likely be due to the controller or other application configuration.

