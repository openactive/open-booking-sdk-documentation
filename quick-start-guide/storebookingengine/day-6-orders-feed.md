# Day 6: Orders feed

## **Objective for Day 6**

Create an Orders RPDE feed, respecting temporary authentication credentials.

### Rationale

The table used to store Orders should have been populated in [Day 5](day-5-b-and-delete-order.md). Day 6 exposes these Orders as an RPDE feed.

## **Step 1: Create Orders RPDE Feed Generator** 

The `StoreBookingEngine` handles the serialisation and parameter validation that would usually be required when implementing an RPDE feed. All that is required to implement the Orders feed is to implement a single method within a new class that inherits from the abstract`OrdersRPDEFeedIncrementingUniqueChangeNumber` or `OrdersRPDEFeedModifiedTimestampAndID` classes, which are very similar to the `IOpportunityDataRPDEFeedGenerator` class from Day 2.

Within `BookingEngineSettings` within `EngineConfig.cs`  the `OrderFeedGenerator` setting configures which generator is called by the controller.

```csharp
OrderFeedGenerator = new AcmeOrdersFeedRPDEGenerator()
```

Two implementations of `OrdersRPDEFeedGenerator` are available depending on your prefered [RPDE Ordering Strategy](https://www.w3.org/2017/08/realtime-paged-data-exchange/#ordering-strategies):

* `OrdersRPDEFeedIncrementingUniqueChangeNumber`
* `OrdersRPDEFeedModifiedTimestampAndID`

Any of the above would require the same `GetRPDEItems` method be implemented, as below, noting that the feed returned should be specific to the specified `clientId`, and that updates to `customerNotice` cause an item to be updated in the RPDE feed.

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
      <td style="text-align:left"><code>RenderOpportunityWithOnlyId</code>
      </td>
      <td style="text-align:left"><code>OrderedItem = RenderOpportunityWithOnlyId(orderItemRow.OpportunityJsonLdType, new Uri(orderItemRow.OpportunityJsonLdId)),</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>RenderOrderId</code>
      </td>
      <td style="text-align:left"><code>Id = this.RenderOrderId(OrderType.Order, uuid),</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>RenderOrderItemId</code>
      </td>
      <td style="text-align:left"><code>Id = this.RenderOrderItemId(OrderType.Order, uuid, orderItemRow.Id),</code>
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
        <p><code>    SellerIdLong = orderItemRow.sellerId</code>
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
</table>## Step 2: Implement Test Interface

{% hint style="warning" %}
**Skip this step:** Note that the StoreBookingEngine does not currently support these test interfaces. These will be added to both the StoreBookingEngine and the Test Suite.
{% endhint %}

Implement the following endpoints of the test interface:

* Seller requested cancellation
* Customer notification

## Step 3: Run Test Suite

Tests should pass for C1, C2, B and Orders Feed

