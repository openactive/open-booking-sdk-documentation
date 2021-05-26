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
</table>

## Step 2: Implement Test Interface

The [OpenActive Test Interface Actions Endpoint](https://openactive.io/test-interface/#actions-endpoint) allows the test suite to test behaviours that are usually triggered by the Seller.

To support seller requested cancellation and customer notification tests, implement the appropriate Actions with the following cases within the `TriggerTestAction` method in the `OrderStore.cs`:

```csharp
case SellerRequestedCancellationSimulateAction _:
    ...
    break;
case CustomerNoticeSimulateAction _:
    ...
    break;
```

## Step 3: Run Test Suite

The [order-deletion](https://github.com/openactive/openactive-test-suite/blob/master/packages/openactive-integration-tests/test/features/core/order-deletion/README.md), [seller-requested-cancellation](https://github.com/openactive/openactive-test-suite/blob/master/packages/openactive-integration-tests/test/features/cancellation/seller-requested-cancellation/README.md), and [customer-notice-notifications](https://github.com/openactive/openactive-test-suite/blob/master/packages/openactive-integration-tests/test/features/notifications/customer-notice-notifications/README.md) \(if implemented\) features within the `openactive-integration-tests` test suite should pass.

Note that headers are used in the tests to provide a temporary `clientId` in place of authentication credentials.

Run these tests in isolation as follows:

```text
npm start -- --runInBand test/features/core/order-deletion/ test/features/cancellation/seller-requested-cancellation/ test/features/notifications/customer-notice-notifications/
```

