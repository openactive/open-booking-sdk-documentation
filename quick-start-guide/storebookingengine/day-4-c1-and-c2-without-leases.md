# Day 4: C1 and C2 without leases

## Objective for Day 4

Implement C1 and C2 responses without leases.

### Rationale

The C1, C2 and B responses contain opportunity data that describes the bookable opportunities. This step focusses on just returning valid opportunity data `GetOrderItem` and completing the other configuration required for C1 and C2 to return responses as expected.

## **Step 1: Set up your Order ID structure**

The Order IDs are configured similarly to `SellerIdComponents`. Within `BookingEngineSettings` the `OrderIdTemplate` setting controls how the Order ID is serialised and deserialised, using the base URL defined in `OrderBaseUrl`.

Note that unlike other JSON-LD IDs, the Order ID must be resolvable, and so must match the endpoint bindings configured in the controller. Hence `OrderBaseUrl` used for Order IDs instead of  `JsonLdIdBaseUrl`.

There is a built-in POCO named `OrderIdComponents` that is defined as follows:

```csharp
public class OrderIdComponents
{
    public OrderType? OrderType { get; set; }
    public string ClientId { get; set; }
    public string uuid { get; set; }
    public long? OrderItemIdLong { get; set; }
    public string OrderItemIdString { get; set; }
}
```

The `uuid` must be included in the `OrderIdTemplate`, as must the `OrderType` \(which is resolved to either "`order-quotes`" or "`orders`"\).

Depending on the type of your internal Order Item ID, you may use either `OrderItemIdLong` or `OrderItemIdString` within the URL template. Once you have chosen which one to use, simply reference that same property consistently wherever you use the Order Item ID throughout your code, and ignore the other property.

Note that the `ClientId` is reserved for use for authentication \(see [Day 8](day-8-authentication.md)\), and **must not** be included in the Order ID.

The following example demonstrates `BookingSystemSettings` to configure Order IDs:

```csharp
OrderBaseUrl = new Uri(baseUrl + "api/openbooking/"),
OrderIdTemplate = new OrderIdTemplate(
    "{+BaseUrl}{OrderType}/{uuid}",
    "{+BaseUrl}{OrderType}/{uuid}#/orderedItems/{OrderItemIdLong}"
    ),
```

## Step 2: GetOrderItem

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

* **`SetResponseOrderItem`** - Set the provided `OrderItem` as a response, and checks that the Seller provided matches that specified by `context.SellerIdComponents`.
* **`SetResponseOrderItemAsSkeleton`** -  Create a skeleton response `OrderItem` from the request `OrderItem` \(useful for adding errors\)
* **`AddError`** - Adds an error to the response. Note this must be called after a response exists \(i.e. after `SetResponseOrderItem` or `SetResponseOrderItemAsSkeleton`\).
* **`ValidateAttendeeDetails`** - Automatically validates attendee details in the request and sets errors on the response based on the **response** values of `AttendeeDetailsRequired` and `OrderItemIntakeForm` \(it requires these response values to be set before this is called\).

### GetOrderItem implementation requirements

Your implementation of `GetOrderItem` must achieve the following behaviour:

* For each `OrderItemContext` provided in the list, parse the **request** and set a **response**.

Note that for the common case of multi-party bookings the same `RequestBookableOpportunityOfferId` may be found across multiple `OrderItemContext`.

Additionally, it must handle the following error cases using `AddError`. Note that depending on the architecture of the booking system, some of these error conditions may be more reliably checked at `LeaseOrderItems` or `BookOrderItems` within the transaction \(see [Day 5](day-5-b-and-delete-order.md)\).

Also note that capacity errors must take into account any opportunities that are leased by this current request's UUID \(e.g. from previous calls to C1 and C2\) and must exclude these from capacity calculations.

<table>
  <thead>
    <tr>
      <th style="text-align:left">Error case</th>
      <th style="text-align:left"><code>AddError(...)</code>
      </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">Specified opportunity does not exist</td>
      <td style="text-align:left"><code>UnknownOpportunityDetailsError</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Specified offer does not exist within that opportunity</td>
      <td style="text-align:left"><code>UnknownOfferError</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Specified opportunity and offer do exist, but are not bookable for any
        reason</td>
      <td style="text-align:left"><code>UnavailableOpportunityError</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">There are conflicts between OrderItems provided e.g. if multiple opportunities
        use the same physical space, and may not be booked together</td>
      <td style="text-align:left"><code>OpportunityIsInConflictError</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">No spaces available for an opportunity at all</td>
      <td style="text-align:left"><code>OpportunityIsFullError</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>Taking into account the number of <code>OrderItemContext</code> instances
          in the list that relate to the same opportunity, there is insufficient
          capacity to book this particular <code>OrderItemContext</code>.</p>
        <p></p>
        <p>As per the <a href="https://www.openactive.io/open-booking-api/EditorsDraft/#order-creation-orderitem-errors">Open Booking API specification</a>,
          this error must only be included against the <code>OrderItemContext</code> instances
          which are in excess of the capacity - for example in an Opportunity with
          a <code>remainingAttendeeCapacity</code> of 2 and with 5 <code>OrderItemContext</code> instances
          related to it, this error would only be emitted against 3 of the <code>OrderItemContext</code> instances.</p>
      </td>
      <td style="text-align:left"><code>OpportunityHasInsufficientCapacityError</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>Taking into account the number of <code>OrderItemContext</code> instances
          in the list that relate to the same opportunity, where is insufficient
          capacity to book this particular <code>OrderItemContext</code> due to a lease
          being held.</p>
        <p></p>
        <p>As per the <a href="https://www.openactive.io/open-booking-api/EditorsDraft/#order-creation-orderitem-errors">Open Booking API specification</a>,
          this error must only be included against the <code>OrderItemContext</code> instances
          which are in excess of the capacity specifically due to a lease being held
          (instead of the error <code>OpportunityHasInsufficientCapacityError</code>)
          - for example in an Opportunity with a <code>remainingAttendeeCapacity</code> of
          3, with 1 additional space held by another lease, then where there are
          9 <code>OrderItemContext</code> related to that same Opportunity, this error
          would only be emitted against 1 of the <code>OrderItemContext</code> instances,
          with <code>OpportunityHasInsufficientCapacityError</code> emitted against
          the other 5 of the <code>OrderItemContext</code> instances. This helps a
          Customer to know whether they should try again.</p>
      </td>
      <td style="text-align:left"><code>OpportunityCapacityIsReservedByLeaseError</code>
      </td>
    </tr>
  </tbody>
</table>

And optionally validate attendee details provided:

* Validate any `attendeeDetails` or `orderItemIntakeFormResponse` provided and use `AddError` to add an `IncompleteAttendeeDetailsError`, `IncompleteIntakeFormError` or `InvalidIntakeFormError`.
* This can be achieved using `ctx.ValidateDetails(flowContext.Stage)`, with additional validation logic executed afterwards as required.
* This is only required if your booking system supports attendee details, otherwise simply ignore these properties in the request and they will therefore not appear in the response. 

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
</table>

## Step 3: Supported Fields

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

## **Step 4: Booking Service Details**

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

## **Step 5: SellerStore**

Implement a new `SellerStore`, with the method `GetSeller` that returns an  `Organization` or `Person` object. This object must include at least `Name` and `TaxMode` properties.

If the Seller is not found, simply return `null` to generate the correct `OpenBookingError`.

Configure the `SellerStore` setting of  `BookingEngineSettings` within `EngineConfig.cs` to use this new implementation of `SellerStore`:

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
    using (var db = FakeBookingSystem.Database.Mem.Database.Open())
    {
        var seller = db.SingleById<SellerTable>(sellerIdComponents.SellerIdLong);
        if (seller == null)
        {
            // Seller not found
            return null;
        }

        return new Organization
        {
            Id = RenderSellerId(new SellerIdComponents { SellerIdLong = seller.Id }),
            Name = seller.Name,
            TaxMode = seller.IsTaxGross ? TaxMode.TaxGross : TaxMode.TaxNet,
            LegalName = seller.Name,
            // Note these strings should be populated from the database
            Address = new PostalAddress
            {
                StreetAddress = "1 Hidden Gem",
                AddressLocality = "Another town",
                AddressRegion = "Oxfordshire",
                PostalCode = "OX1 1AA",
                AddressCountry = "GB"
            },
            TermsOfService = new List<Terms>
            {
                new PrivacyPolicy
                {
                    Name = "Privacy Policy",
                    Url = new Uri("https://example.com/privacy.html"),
                    RequiresExplicitConsent = false
                }
            },
            IsOpenBookingAllowed = true,
        };
    }
}
```

### Booking systems supporting a [Single Seller](../design-considerations.md#booking-system-architecture)

For a Single Seller booking system, your implementation of `GetSeller` must return an `Organization` or `Person` that represents the Single Seller for which your booking system is configured.

In Single Seller mode both `SellerIdLong` and `SellerIdString` properties of `sellerIdComponents` will be `null`, and the data for your Seller should instead be retrieved from your booking system configuration table. 

`RenderSingleSellerId()` is provided to render the `Id`.

The following example demonstrates `BookingSystemSettings` for a hard-coded Single Seller:

```csharp
protected override ILegalEntity GetSeller(SellerIdComponents sellerIdComponents)
{
    // For Single Seller booking systems, no ID will be available from sellerIdComponents, and this data should instead come from your configuration table
    return new Organization
    {
        Id = RenderSingleSellerId(),
        Name = "Test Seller",
        TaxMode = TaxMode.TaxGross,
        LegalName = "Test Seller Ltd",
        Address = new PostalAddress
        {
            StreetAddress = "1 Hidden Gem",
            AddressLocality = "Another town",
            AddressRegion = "Oxfordshire",
            PostalCode = "OX1 1AA",
            AddressCountry = "GB"
        },
        TermsOfService = new List<Terms>
        {
            new PrivacyPolicy
            {
                Name = "Privacy Policy",
                Url = new Uri("https://example.com/privacy.html"),
                RequiresExplicitConsent = false
            }
        },
        IsOpenBookingAllowed = true,
    };
}
```

## **Step 6: Run Test Suite**

The [availability-check](https://github.com/openactive/openactive-test-suite/blob/master/packages/openactive-integration-tests/test/features/core/availability-check/README.md) feature within the `openactive-integration-tests` test suite should pass.

Run this test in isolation as follows:

```text
npm start -- --runInBand test/features/core/availability-check/
```





  


