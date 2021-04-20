# Day 7: Cancellation

## **Objective for Day 7**

Implement customer requested cancellation

### Rationale

With the steps from the previous days in place, customer requested cancellation should be straightforward to implement.

## Step **1**: Implement cancellation within the OrderStore

Implement the method `CustomerCancelOrderItems` within the  `OrderStore` implementation created on [Day 5](day-5-b-and-delete-order.md).

Simply check that the Order `clientId` and `uuid` match a record in your database, and then set the status of each `OrderItem` referenced by `orderItemIds` to `CustomerCancelled`. Note that this must also touch the underlying `Order` such that it appears as updated in the RPDE feed.

```csharp
public override bool CustomerCancelOrderItems(OrderIdComponents orderId, SellerIdComponents sellerId, OrderIdTemplate orderIdTemplate, List<OrderIdComponents> orderItemIds)
{

}
```

Note that the `OrderStore` handles customer requested cancellation, rather than the `OpportunityStore`.

{% hint style="info" %}
**Feedback requested:** An option could also be available to manage this through the `OpportunityStore` using the transaction pattern similar to that used for leasing and booking in Day 5.  Would this be more useful, or too heavy-weight?
{% endhint %}

## Step 2: Run Test Suite

The [customer-requested-cancellation](https://github.com/openactive/openactive-test-suite/blob/master/packages/openactive-integration-tests/test/features/cancellation/customer-requested-cancellation/README.md) feature within the `openactive-integration-tests` test suite should pass.

Run this test in isolation as follows:

```text
npm start -- --runInBand test/features/cancellation/customer-requested-cancellation/
```

