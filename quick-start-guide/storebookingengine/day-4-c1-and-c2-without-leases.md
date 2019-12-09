# Day 4: C1 and C2 without leases

## Objective for Day 4

Implement C1 and C2 responses without leases.

### Rationale

The C1, C2 and B responses contain opportunity data that describes the bookable opportunities. This step focusses on just returning valid opportunity data `GetOrderItem` and completing the other configuration required for C1 and C2 to return responses as expected.

## Step 1: GetOrderItem

This step will implement the `GetOrderItem` within the implementations of ****`OpportunityStore` created in Day 3.

```csharp
protected override void GetOrderItem(List<OrderItemContext<SessionOpportunity>> orderItemContexts, StoreBookingFlowContext flowContext)
{
    ...
}
```

The objective of `GetOrderItem` is to create a response `OrderItem` for each request `OrderItem` provided.

The `OpportunityStore` methods generally deal with lists of `OrderItemContext`, where each contains a `OrderItem` request object, along with methods to add an `OrderItem` response object. This allows your implementation to process each `OrderItem` individually, and also to optimise database calls where groups of `OrderItem` \(if they are handled by the same `OpportunityStore`\) can be retrieved together.

### OrderItemContext capabilities

Each `OrderItemContext`  contains the following properties that represent the request:

* **`RequestBookableOpportunityOfferId`** - The ID of the `OrderItem`, to be looked up in your database
* **`RequestOrderItem`** - The `OrderItem` from the original request

Each `OrderItemContext` also contains the following methods, which can be used to construct a response:

* **`SetResponseOrderItem`** - Set the provided `OrderItem` as a response
* **`SetResponseOrderItemAsSkeleton`** -  Create a skeleton response `OrderItem` from the request `OrderItem` \(useful for adding errors\)
* **`AddError`** - Adds an error to the response. Note this must be called after a response exists \(i.e. after `SetResponseOrderItem` or `SetResponseOrderItemAsSkeleton`\).
* **`ValidateAttendeeDetails`** - Automatically validates attendee details in the request and sets errors on the response based on the **response** values of `AttendeeDetailsRequired` and `OrderItemIntakeForm` \(it requires these response values to be set before this is called\).

### GetOrderItem implementation requirements

Your implementation of `GetOrderItem` must achieve the following behaviour:

* For each `OrderItemContext` provided in the list, parse the **request** and set a **response**.

Note that for the common case of multi-party bookings the same `RequestBookableOpportunityOfferId` may be found across multiple `OrderItemContext`.

Additionally, it must handle the following error cases using `AddError`:

| Error case | `AddError(...)` |
| :--- | :--- |
| Specified opportunity does not exist | `UnknownOpportunityDetailsError` |
| Specified offer does not exist within that opportunity | `UnknownOfferError` |
| Specified opportunity and offer do exist, but are not bookable for any reason | `UnavailableOpportunityError` |
| There are conflicts between OrderItems provided e.g. if multiple opportunities use the same physical space, and may not be booked together | `OpportunityIsInConflictError` |
| No spaces available for an opportunity | `OpportunityIsFullError` |
| This opportunity is not from the Seller specified by `context.SellerIdComponents` | `OpportunitySellerMismatchError` |

And optionally validate attendee details provided:

* Validate any `attendeeDetails` or `orderItemIntakeFormResponse` provided and use `AddError` to add an `IncompleteAttendeeDetailsError`, `IncompleteIntakeFormError` or `InvalidIntakeFormError`.
* This can be achieved using `ValidateAttendeeDetails()`, with additional validation logic executed afterwards as required.
* This is only required if your booking system supports attendee details, otherwise simply ignore these properties in the request and they will therefore not appear in the response. 

{% hint style="info" %}
Note `ValidateAttendeeDetails` is not yet implemented, pending feedback on the overall Day 1-7 approach.
{% endhint %}

### Helpers available

Note that the helpers available within the `OpportunityDataRPDEFeedGenerator` implementations \(from Day 2\) are also available here:

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
      <td style="text-align:left">
        <p><code>RenderSingleSellerId</code>
        </p>
        <p></p>
        <p>(for Single Seller)</p>
      </td>
      <td style="text-align:left"><code>Id = this.RenderSingleSellerId(),</code>
      </td>
    </tr>
  </tbody>
</table>## Step 2: Supported Fields

Customise the following `StoreBookingEngineSettings` within `EngineConfig.cs` to include only those fields that your booking system supports and stores \(as only supported fields must be reflected back to the Broker\):

* `CustomerPersonSupportedFields`
* `CustomerOrganizationSupportedFields`
* `BrokerSupportedFields`

For example:

```csharp
CustomerPersonSupportedFields = p => new Person {
    Email = p.Email,
    GivenName = p.GivenName,
    FamilyName = p.FamilyName,
    Telephone = p.Telephone
},
```

## **Step 3: Booking Service Details**

Complete the details in the setting below to include information about your booking system:

* `BookingServiceDetails`

For example:

```csharp
BookingServiceDetails = new BookingService
{
    Name = "Acme booking system",
    Url = new Uri("https://example.com"),
    TermsOfService = new List<Terms>
    {
        new PrivacyPolicy
        {
            Url = new Uri("https://example.com/privacy.html")
        }
    }
},
```

## **Step 4: SellerStore**

Implement a new `SellerStore`, with the method `GetSeller` that returns an  `Organization` or `Person` object. This object must include at least `Name` and `TaxMode` properties.

If the Seller is not found, simply return `null` to generate the correct `OpenBookingError`.

Configure the `SellerStore` setting of  `StoreBookingEngineSettings` within `EngineConfig.cs` to use this new implementation of `SellerStore`:

```csharp
SellerStore = new MyCustomSellerStore(),
```

### Booking systems supporting [Multiple Sellers](../design-considerations.md#booking-system-architecture)

For a booking system supporting Multiple Sellers, your implementation of `GetSeller` must return an `Organization` or `Person` that represents the `sellerIdComponents` supplied, using either `SellerIdLong` or `SellerIdString` depending on your `SellerIdTemplate`.

The returned object must include an `Id` property containing a URL representing the `sellerIdComponents` supplied, which can be rendered using `RenderSellerId(...)`.

The following example demonstrates `GetSeller` for Multiple Sellers:

```csharp
protected override ILegalEntity GetSeller(SellerIdComponents sellerIdComponents)
{
    var seller = FakeBookingSystem.Database.Sellers.SingleOrDefault(x => x.Id == sellerIdComponents.SellerIdLong);
    if (seller != null)
    {
        return seller.IsIndividual ? (ILegalEntity)new Person
        {
            Id = this.RenderSellerId(new SellerIdComponents { SellerIdLong = seller.Id }),
            Name = seller.Name,
            TaxMode = TaxMode.TaxGross
        } : (ILegalEntity)new Organization
        {
            Id = this.RenderSellerId(new SellerIdComponents { SellerIdLong = seller.Id }),
            Name = seller.Name,
            TaxMode = TaxMode.TaxGross
        };
    }
    else
    {
        return null;
    }
}
```

### Booking systems supporting a [Single Sellers](../design-considerations.md#booking-system-architecture)

For a Single Seller booking system, your implementation of `GetSeller` must return an `Organization` or `Person` that represents the Single Seller for which your booking system is configured.

In Single Seller mode both `SellerIdLong` and `SellerIdString` properties of `sellerIdComponents` will be `null`, and the data for your Seller should instead be retrieved from your booking system configuration table. 

`RenderSingleSellerId()` is provided to render the `Id`.

The following example demonstrates `BookingSystemSettings` for a hard-coded Single Seller:

```csharp
protected override ILegalEntity GetSeller(SellerIdComponents sellerIdComponents)
{
    return new Organization
    {
        Id = this.RenderSingleSellerId(),
        Name = "Test Seller",
        TaxMode = TaxMode.TaxGross
    };
}
```

## **Step 5: Run Test Suite**

Tests should pass for C1 and C2.  


