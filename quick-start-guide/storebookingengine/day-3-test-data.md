# Day 3: Test data

## **Objective for Day 3**

Allow the test suite to create events within the open feeds.

### Rationale

In order to allow the test suite to fully test your implementation, it must be permitted to use two test endpoints to create events and delete events within your database. Note these endpoints must not be available in your application in a production environment.

## Step 1 - Inspect Test Endpoints

These endpoints are shown below, and are already configured in your controller from Day 1:

```csharp
// POST api/openbooking/test-interface/datasets/uat-ci/opportunities
[HttpPost("test-interface/datasets/{testDatasetIdentifier}/opportunities")]
public IActionResult TestInterfaceDatasetInsert([FromServices] IBookingEngine bookingEngine, string testDatasetIdentifier, [FromBody] string @event)
{
    try
    {
        return bookingEngine.InsertTestOpportunity(testDatasetIdentifier, @event).GetContentResult();
    }
    catch (OpenBookingException obe)
    {
        return obe.ErrorResponseContent.GetContentResult();
    }
}

// DELETE api/openbooking/test-interface/datasets/uat-ci
[HttpDelete("test-interface/datasets/{testDatasetIdentifier}")]
public IActionResult TestInterfaceDatasetDelete([FromServices] IBookingEngine bookingEngine, string testDatasetIdentifier)
{
    try
    {
        return bookingEngine.DeleteTestDataset(testDatasetIdentifier).GetContentResult();
    }
    catch (OpenBookingException obe)
    {
        return obe.ErrorResponseContent.GetContentResult();
    }
}

// POST api/openbooking/test-interface/actions
[HttpPost("test-interface/actions")]
public IActionResult TestInterfaceAction([FromServices] IBookingEngine bookingEngine, [FromBody] string action)
{
    try
    {
        return bookingEngine.TriggerTestAction(action).GetContentResult();
    }
    catch (OpenBookingException obe)
    {
        return obe.ErrorResponseContent.GetContentResult();
    }
}
```

Note at this stage these endpoints are not authenticated, and should not be left unsecured in production.

## Step 2 - Configure Opportunity Stores

The `StoreBookingEngine` handles the orchestration of the booking across various implementations of `OpportunityStore` , where a store is defined for a number of related opportunity types that use a common `IBookableIdComponents`.

Implementations of `OpportunityStore` are therefore configured to be routed from a number of  related opportunity types, and each bookable opportunity type within your booking system must match at least one implementation of `OpportunityStore`.

Within `StoreBookingEngineSettings` within `EngineConfig.cs`  the `OpportunityStoreRouting` setting configures the routing to the different `OpportunityStore`implementations from the `CreateTestData` and `DeleteTestData` methods being called in the controller.

```csharp
// List of _bookable_ opportunity types and which store to route to for each
OpportunityStoreRouting = new Dictionary<IOpportunityStore, List<OpportunityType>> {
    {
        new SessionStore(), new List<OpportunityType> { OpportunityType.ScheduledSession }
    }
},
```

## Step 3 - Implement Test Interface

For this day of the guide, in each store all that is required is to implement `CreateOpportunityWithinTestDataset` and `DeleteTestDataset` within an implementation of the abstract `OpportunityStore` class. The other methods can simply throw `NotImplementedException`.

`CreateOpportunityWithinTestDataset` must create the opportunity that meets the [`TestOpportunityBookable`](https://openactive.io/test-interface#TestOpportunityBookable) criteria as defined in the [Open Booking API Test Interface](https://openactive.io/test-interface/), or throw a `NotSupportedException` exception if the requested opportunity type is not supported. The opportunity must be linked to the string value of `testDatasetIdentifier`, to allow it to be deleted by a call to `DeleteTestDataset`.

`DeleteTestDataset` must delete all opportunities that match the `testDatasetIdentifier` provided.

Note that the names of the opportunities created within the booking system may be prefixed by the booking system to make them recognisable within the UI, as the opportunities are referenced by `@id` within the test suite.

## Step 4 - Run Test Suite

The [test-interface](https://github.com/openactive/openactive-test-suite/blob/master/packages/openactive-integration-tests/test/features/core/test-interface/README.md) feature within the `openactive-integration-tests` test suite should pass if the implementations of `OpportunityStore` have been implemented successfully, and if the created opportunity correctly appears within the appropriate feed.

Run this test in isolation as follows:

```text
npm start -- --runInBand test/features/core/test-interface/
```

