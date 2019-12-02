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

## Step 2 - Configure Stores

## Step 3 - Implement Stores

* **Implement endpoint to add test data \(duplicated for each data type - e.g. sessions, facilities\)**
* **Implement endpoint to remove test data \(duplicated for each data type - e.g. sessions, facilities\)**
* **These are repeated for each data type - e.g. sessions, facilities**

## Step 4 - Test

* * **“Test data exists in correct state” test should pass after test data is added**
  * **“Test data does not exist” test should pass after test data is removed**

