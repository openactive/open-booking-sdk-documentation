# Day 6: Orders feed

## **Objective for Day 6**

Create an Orders RPDE feed, respecting temporary authentication credentials.

### Rationale

The table used to store Orders should have been populated in [Day 5](day-5-b-and-delete-order.md). Day 6 exposes these Orders as an RPDE feed.

## **Step 1 - Create Orders RPDE Feed Generator** 

The `StoreBookingEngine` handles the serialisation and parameter validation that would usually be required when implementing an RPDE feed. All that is required to implement the Orders feed is to implement a single method within a new class that inherits from the abstract`IRPDEOrdersFeedIncrementingUniqueChangeNumber` or `IRPDEOrdersFeedModifiedTimestampAndIDString` classes, which are very similar to the `IOpportunityDataRPDEFeedGenerator` class from Day 2.

Within `BookingEngineSettings` within `EngineConfig.cs`  the `OrderFeedGenerator` setting configures which generator is called by the controller.

```csharp
OrderFeedGenerator = new AcmeOrdersFeedRPDEGenerator()
```

Two implementations of `OrdersRPDEFeedGenerator` are available depending on your prefered [RPDE Ordering Strategy](https://www.w3.org/2017/08/realtime-paged-data-exchange/#ordering-strategies):

* `OrdersRPDEFeedIncrementingUniqueChangeNumber`
* `OrdersRPDEFeedModifiedTimestampAndID`

Any of the above would require the same `GetRPDEItems` method be implemented, as below:

```csharp
public class AcmeOrdersFeedRPDEGenerator : OrdersRPDEFeedModifiedTimestampAndID
{
    protected override List<RpdeItem> GetRPDEItems(string clientId, long? afterTimestamp, string afterId)
    {
        ...
    }
}
```

The appropriate query for the database for your chosen [RPDE Ordering Strategy](https://www.w3.org/2017/08/realtime-paged-data-exchange/#ordering-strategies) must be used, and the ordering of the items returned from your overridden method is automatically validated to ensure consistency with the specification.

Within the mapping of your data to the OpenActive model, there are a few helper methods available from the base class to construct IDs specific for the feed:

<table>
  <thead>
    <tr>
      <th style="text-align:left">Method</th>
      <th style="text-align:left">Example</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><code>RenderOpportunityId</code>
      </td>
      <td style="text-align:left">
        <p><code>Id = this.RenderOpportunityId(new SessionOpportunity</code>
        </p>
        <p><code>{</code>
        </p>
        <p><code>    OpportunityType = OpportunityType.SessionSeries,</code>
        </p>
        <p><code>    SessionSeriesId = @class.Id</code>
        </p>
        <p><code>}),</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>RenderOfferId</code>
      </td>
      <td style="text-align:left">
        <p><code>Id = this.RenderOfferId(new SessionOpportunity</code>
        </p>
        <p><code>{</code>
        </p>
        <p><code>    OfferOpportunityType = OpportunityType.SessionSeries,</code>
        </p>
        <p><code>    SessionSeriesId = @class.Id,</code>
        </p>
        <p><code>    OfferId = 0</code>
        </p>
        <p><code>}),</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><code>RenderSellerId</code>
        </p>
        <p></p>
        <p>(for Multiple Sellers)</p>
      </td>
      <td style="text-align:left">
        <p><code>Id = this.RenderSellerId(new SellerIdComponents</code>
        </p>
        <p><code>{</code>
        </p>
        <p><code>    SellerIdLong = seller.Id</code>
        </p>
        <p><code>}),</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>RenderSingleSellerId</code>
        <br />
        <br />(for Single Seller)</td>
      <td style="text-align:left"><code>Id = this.RenderSingleSellerId(),</code>
      </td>
    </tr>
  </tbody>
</table>\*\*\*\*

**Implement one more complex response:**

* **openActiveEngine.outputOrderFeedPage\(modified, id\)**

**Implement endpoint to test customer notice \(testCustomerNotice\)**

* **\(Note it may not yet be possible to set a customer notice from inside the booking system\)**

**Test for customer notice should pass**   


**Implement endpoint to test provider-side cancellation \(testCommandTriggerProviderCancellation\)**

* **\(Note it may not yet be possible for provider to cancel from inside the booking system\)**

**Test for provider side cancellation should pass**  


