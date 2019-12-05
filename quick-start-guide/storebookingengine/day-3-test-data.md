# Day 3: Test data

## **Objective for Day 3**

Allow the test suite to create events within the open feeds.

### Rationale

In order to allow the test suite to fully test your implementation, it must be permitted to use two test endpoints to create events and delete within your database. Note these endpoints must not be available in your application in a production environment.

## Step 1 - Inspect Test Endpoints

These endpoints are shown below, and are already configured in your controller from Day 1:

```csharp
// POST api/openbooking/test-interface/scheduledsession
[HttpPost("test-interface/{type}")]
public IActionResult Post([FromServices] IBookingEngine bookingEngine, string type, [FromBody] string @event)
{
    try
    {
        return bookingEngine.CreateTestData("<client-credential>", type, @event).GetContentResult();
    }
    catch (OpenBookingException obe)
    {
        return obe.ErrorResponseContent.GetContentResult();
    }
}

// DELETE api/openbooking/test-interface/scheduledsession/{name}
[HttpDelete("test-interface/{type}/{name}")]
public IActionResult Delete([FromServices] IBookingEngine bookingEngine, string type, string name)
{
    try
    {
        return bookingEngine.DeleteTestData("<client-credential>", type, name).GetContentResult();
    }
    catch (OpenBookingException obe)
    {
        return obe.ErrorResponseContent.GetContentResult();
    }
}
```

Note at this stage these endpoints are not authenticated

## Step 2 - Configure Opportunity Stores

The `StoreBookingEngine` handles the orchestration of the booking across various implementations of `OpportunityStore` , where a store is defined for a number of related opportunity types that use a common `IBookableIdComponents`.

Implementations of `OpportunityStore` are therefore configured to be routed from a number of  related opportunity types, and each bookable opportunity type within your booking system must match at least one implementation of `OpportunityStore`.

Within `StoreBookingEngineSettings` within `Startup.cs` or `ServiceConfig.cs` the `OpenBookingStoreRouting` setting configures the routing to the different `OpportunityStore`implementations from the `CreateTestData` and `DeleteTestData` methods being called in the controller.

```csharp
// List of _bookable_ opportunity types and which store to route to for each
OpenBookingStoreRouting = new Dictionary<IOpportunityStore, List<OpportunityType>> {
    {
        new SessionStore(), new List<OpportunityType> { OpportunityType.ScheduledSession }
    }
},
```

## Step 3 - Implement Test Interface

For this day of the guide, in each store all that is required is to implement `CreateTestDataItem` and `DeleteTestDataItem` within an implementation of the abstract `OpportunityStore` class. The other methods can simply throw `NotImplementedException`.

`CreateTestDataItem` must create the opportunity that is provided, or return a 500 error if the requested opportunity type is not supported. 

`DeleteTestDataItem` must delete all opportunities that match the name provided.

Note that the names of the items stored within the booking system may be prefixed by the test interface, where such a prefix is applied to both the event stored by `CreateTestDataItem` and event matched by `DeleteTestDataItem`.

{% hint style="warning" %}
Note that the current test suite implementation does not support prefixes, however these can be easily added on request if this is of interest.
{% endhint %}

## Step 4 - Run Test Suite

The "Create test event" test within the `openactive-integration-tests` test suite should pass if the implementations of `OpportunityStore` have been implemented successfully, and if the created Event correctly appears within the appropriate feed.

