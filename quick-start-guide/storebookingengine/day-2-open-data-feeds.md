# Day 2: Open data feeds

## **Objective for Day 2**

Create open data RPDE feeds live from your application.

### Rationale

Open Booking API is built on top of open data feed publishing, and RPDE feeds are therefore a prerequisite to implementing the rest of the API.

## Step 1 - Understand the RPDE specification

Before continuing with this tutorial, the following video offers a good theoretical understanding of the RPDE specification. Further information can be found in the [developers guide](https://developer.openactive.io/publishing-data/data-feeds/how-a-data-feed-works) and [RPDE specification](https://www.w3.org/2017/08/realtime-paged-data-exchange/).

{% embed url="https://www.youtube.com/watch?v=yHZS24xzY-8" %}

## **Step 2 - Set up your** Opportunity and Offer pair **ID structure**

In order for opportunities and offers within open RPDE feeds to be bookable via the Open Booking API, they must have IDs.

The Open Booking API is based on [JSON-LD](https://json-ld.org/).  JSON-LD IDs always take the form of a URL, and hence all IDs within the Open Booking API also take the form of a URL. The `StoreBookingEngine` automatically constructs and parses these URLs for you, and deserialises them into POCOs. It also handles routing based on the URL template to the relevant store depending on the type of opportunity. 

In order to take advantage of these features, you need to configure the URL structure specifically for your application.

{% hint style="info" %}
When making changes to these setting, specifically for `IBookableIdComponents`, consider using Visual Studio's refactoring tools to alter the example provided, to ensure that the rest of the project still compiles.
{% endhint %}

The first URLs we'll set up are for the [Opportunity and Offer pair](https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair)s, such as the IDs found in the Open Booking API request snippet below:

```csharp
"orderedItem": [
  {
    "@type": "OrderItem",
    "acceptedOffer": {
      "@type": "Offer",
      "@id": "https://example.com/api/identifiers/api/session-series/100002#/offers/0"
    },
    "orderedItem": {
      "@type": "ScheduledSession",
      "@id": "https://example.com/api/identifiers/api/session-series/100002/scheduled-sessions/100003"
    }
  }
]
```

### IBookablePairIdTemplate and IBookableIdComponents

There are two key components that handle JSON-LD ID serialisation and deserialisation within the `StoreBookingEngine`:

* **`IBookablePairIdTemplate`** instances specify the ID URL templates used by your application and associates them with OpenActive opportunity types, as well as the hierarchy of types that are in use from the [OpenActive data model](https://www.openactive.io/modelling-opportunity-data/). They also define the types of opportunities that are bookable, and which feeds are used to publish the opportunities.
* **`IBookableIdComponents`** are POCO objects that you define specifically based on the underlying data model within your booking system, used to deserialise the ID URL. The properties within these are mapped by name to the placeholders within the associated `IBookablePairIdTemplate` URL templates. `IBookableIdComponents` represents an [Opportunity and Offer pair](https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair), and hence includes _both_ the Opportunity ID and Offer ID, with their matching overlapping components.

Note that the JSON-LD IDs do not need to resolve to actual endpoints, provided they are unique and exist within your domain.

### **Configuring IBookablePairIdTemplates** 

`IBookablePairIdTemplate` instances are configured in `Startup.cs` or `ServiceConfig.cs` within `BookingEngineSettings`:

```csharp
new BookingEngineSettings
{
    // This assigns the ID pattern used for each ID
    IdConfiguration = new List<IBookablePairIdTemplate> {
        // Note that ScheduledSession is the only opportunity type that allows offer inheritance  
        new BookablePairIdTemplateWithOfferInheritance<SessionOpportunity> (
            // Opportunity
            new OpportunityIdConfiguration
            {
                OpportunityType = OpportunityType.ScheduledSession,
                AssignedFeed = OpportunityType.ScheduledSession,
                OpportunityIdTemplate = "{+BaseUrl}api/session-series/{SessionSeriesId}/scheduled-sessions/{ScheduledSessionId}",
                OfferIdTemplate =       "{+BaseUrl}api/session-series/{SessionSeriesId}/scheduled-sessions/{ScheduledSessionId}#/offers/{OfferId}",
                Bookable = true
            },
            // Parent
            new OpportunityIdConfiguration
            {
                OpportunityType = OpportunityType.SessionSeries,
                AssignedFeed = OpportunityType.SessionSeries,
                OpportunityIdTemplate = "{+BaseUrl}api/session-series/{SessionSeriesId}",
                OfferIdTemplate =       "{+BaseUrl}api/session-series/{SessionSeriesId}#/offers/{OfferId}",
                Bookable = false
            }),
            
         new BookablePairIdTemplate<EventOpportunity>(
            // Opportunity
            new OpportunityIdConfiguration
            {
                OpportunityType = OpportunityType.Event,
                AssignedFeed = OpportunityType.Event,
                OpportunityUriTemplate = "{+BaseUrl}api/events/{EventId}",
                OfferUriTemplate =       "{+BaseUrl}api/events/{EventId}#/offers/{OfferId}",
                Bookable = true
            }),
            
        ...
    }
}
```

<table>
  <thead>
    <tr>
      <th style="text-align:left">Where you these referenced in the example above...</th>
      <th style="text-align:left">The following annotations apply...</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><code>SessionOpportunity</code> and <code>EventOpportunity</code>
      </td>
      <td style="text-align:left">Examples of an <code>IBookableIdComponents</code> POCO used to map to placeholders
        within the ID templates defined</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>BookablePairIdTemplate</code>
      </td>
      <td style="text-align:left">A template that contains a hierarchy of up to three OpenActive types,
        in order from child to parent (e.g. <code>ScheduledSession</code> &#x2192; <code>SessionSeries</code> &#x2192; <code>EventSeries</code>). <code>BookablePairIdTemplate</code> accepts
        the generic parameter of an <code>IBookableIdComponents</code> type, which
        binds the placeholders within the URL templates specified to the properties
        defined within the <code>IBookableIdComponents</code> POCO.</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>BookablePairIdTemplateWithOfferInheritance</code>
      </td>
      <td style="text-align:left">A specialism of<b> </b><code>BookablePairIdTemplate</code> designed for
        the special case of <code>ScheduledSession</code> &#x2192; <code>SessionSeries</code> where
        Offers may be inherited from <code>SessionSeries</code> to <code>ScheduledSession</code>.</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>OpportunityType</code>
      </td>
      <td style="text-align:left">The OpenActive opportunity type represented</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>AssignedFeed</code>
      </td>
      <td style="text-align:left">The feed within which this opportunity type is published (for example,
        the <code>EventSeries</code> may be embedded within a <code>SessionSeries</code> feed
        as shown in <a href="https://validator.openactive.io/?url=https%3A%2F%2Fwww.openactive.io%2Fdata-models%2Fversions%2F2.x%2Fexamples%2Fsessionseries-eventseries-split_example_1.json&amp;version=2.0">this example</a>).</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><code>OpportunityUriTemplate</code>
        </p>
        <p><code>OfferUriTemplate</code>
        </p>
      </td>
      <td style="text-align:left">The URL Template<code>s</code> used to serialise/deserialise the POCO for
        an Opportunity ID and Offer ID. Note that the Opportunity ID and Offer
        ID are deserialised into the same POCO, as an <a href="https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair">Opportunity and Offer pair</a>,
        and hence any shared components within these must match (e.g. an Offer
        is specific to an Event, and hence will include both the Event ID and and</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>Bookable</code>
      </td>
      <td style="text-align:left">Whether the specified <a href="https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair">Opportunity and Offer pair</a> is
        bookable within this booking system using the Open Booking API. Note that <code>SessionSeries</code> and <code>FacilityUse</code> are
        not bookable within the Open Booking API, which instead prefers <code>ScheduledSessions</code> or <code>Slots</code>.</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>BaseUrl</code>
      </td>
      <td style="text-align:left">This is a placeholder within the ID templates that has a special function
        of ensuring conformity with the <code>JsonLdIdBaseUrl</code> setting within <code>BookingEngineSettings</code>,
        both for serialisation and deserialisation.</td>
    </tr>
  </tbody>
</table>### **Configuring IBookableIdComponents**

 `IBookableIdComponents` are defined by the classes in the `IdComponents` directory.

These classes must be created by the booking system. There is a choice of `string`, `long?` and `Uri` available as the type for each component of the ID, as well as any `enum` type. The names of the components must exactly match the placeholders within the associated `IBookablePairIdTemplates`.

```csharp
public class SessionOpportunity : IBookableIdComponentsWithInheritance
{
    public OpportunityType? OpportunityType { get; set; }
    public OpportunityType? OfferOpportunityType { get; set; }
    public long? SessionSeriesId { get; set; }
    public long? ScheduledSessionId { get; set; }
    public long? OfferId { get; set; }
}
```

| In the example above | Description |
| :--- | :--- |
| `OpportunityType`  | The `OpportunityType` represents the type in the hierarchy that the  [Opportunity and Offer pair](https://www.openactive.io/open-booking-api/EditorsDraft/#definition-of-a-bookable-opportunity-and-offer-pair) represents \(e.g. `HeadlineEvent` or its child `Event`\). This property is available by the class extending either `IBookableIdComponents` or `IBookableIdComponentsWithInheritance`. |
| `OfferOpportunityType` | For `SessionSeries`, where the `Offer` can be inherited to the child `ScheduledSession`, `OfferOpportunityType` indicates which `Offer` is being referenced - the parent or the child. This property is only available by the class extending `IBookableIdComponentsWithInheritance`. |

## **Step 3 - Set up your Seller ID structure**

The [Seller](https://www.openactive.io/open-booking-api/EditorsDraft/#dfn-seller) IDs are configured slightly differently to the Opportunity and Offer pair IDs, for simplicity.

There is a built-in POCO named `SellerIdComponents`that is defined as follows:

```csharp
public class SellerIdComponents
{
    public long? SellerIdLong { get; set; }
    public string SellerIdString { get; set; }
}
```

Within `BookingEngineSettings` the `SellerIdTemplate` setting controls how the Seller ID is serialised and deserialised. There are two options for Seller IDs: [Single Seller and Multiple Seller](../design-considerations.md#booking-system-architecture).

### Booking systems supporting [Multiple Sellers](../design-considerations.md#booking-system-architecture)

Depending on the type of your internal Seller ID, you may use either `SellerIdLong` or `SellerIdString` within the URL template. Once you have chosen which one to use, simply reference that same property consistently wherever you use the Seller ID throughout your code, and ignore the other property.

The following example demonstrates `BookingSystemSettings` for Multiple Sellers:

```csharp
SellerIdTemplate = new SingleIdTemplate<SellerIdComponents>(
    "{+BaseUrl}api/sellers/{SellerIdLong}"
    ),
```

### Booking systems supporting a [Single Sellers](../design-considerations.md#booking-system-architecture)

To use Single Seller mode, simply use neither `SellerIdLong` or `SellerIdString` within the URL template, and set `HasSingleSeller` to `true`. Then simply do not reference `SellerIdComponents` anywhere in your code, as both `SellerIdLong` and `SellerIdString` will be null. `RenderSingleSellerId` is provided for scenarios where a Single Seller ID needs to be rendered.

The following example demonstrates `BookingSystemSettings` for a Single Seller:

```csharp
HasSingleSeller = true,
SellerIdTemplate = new SingleIdTemplate<SellerIdComponents>(
    "{+BaseUrl}api/seller"
    ),
```

## **Step 4 - Create RPDE Feed Generators** 

The `StoreBookingEngine` handles the serialisation and parameter validation that would usually be required when implementing an RPDE feed. All that is required for each feed is to implement a single method within an `IOpportunityDataRPDEFeedGenerator` class.

{% hint style="info" %}
If you have already implemented RPDE feeds within your application using [OpenActive.NET](https://github.com/openactive/OpenActive.NET), your existing database queries and OpenActive model mapping should easily transferable to within the generator method.
{% endhint %}

Within `BookingEngineSettings` within `EngineConfig.cs`  the `OpenDataFeeds` setting configures the routing to the different feed generators from the `GetOpenDataRPDEPageForFeed` method being called in the controller.

```csharp
OpenDataFeeds = new Dictionary<OpportunityType, IOpportunityDataRPDEFeedGenerator> {
    {
        OpportunityType.ScheduledSession, new AcmeScheduledSessionRPDEGenerator()
    },
    {
        OpportunityType.SessionSeries, new AcmeSessionSeriesRPDEGenerator()
    },
    {
        OpportunityType.FacilityUse, new AcmeFacilityUseRPDEGenerator()
    }
    ,
    {
        OpportunityType.FacilityUseSlot, new AcmeFacilityUseSlotRPDEGenerator()
    }
},
```

Three implementations of `IOpportunityDataRPDEFeedGenerator` are available depending on your prefered [RPDE Ordering Strategy](https://www.w3.org/2017/08/realtime-paged-data-exchange/#ordering-strategies), and whether your ID is a `long` or `string`.

Using the example of the `IBookableIdComponents` class `SessionOpportunity` from [Step 2](day-2-open-data-feeds.md#step-2-set-up-your-opportunity-and-offer-pair-id-structure), and the OpenActive feed root type of `ScheduledSession`,  a generator is created by subclassing one of the following:

* `RPDEFeedModifiedTimestampAndIDLong<SessionOpportunity, ScheduledSession>`
* `RPDEFeedModifiedTimestampAndIDString<SessionOpportunity, ScheduledSession>`
* `RPDEFeedIncrementingUniqueChangeNumber<SessionOpportunity, ScheduledSession>`

Any of the above would require the same `GetRPDEItems` method be implemented, as below:

```csharp
public class AcmeScheduledSessionRPDEGenerator : 
    RPDEFeedModifiedTimestampAndIDLong<SessionOpportunity, ScheduledSession>
{
    protected override List<RpdeItem<ScheduledSession>> GetRPDEItems(long? afterTimestamp, long? afterId)
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
</table>## **Step 5 - Configure Dataset Site**

The `DatasetSiteGeneratorSettings` within `EngineConfig.cs`  can be used to configure your dataset site.

Only the `OpenDataFeedBaseUrl` is used by the `StoreBookingEngine`, so this must be accurate to proceed with testing.

All other settings are described within the documentation for [OpenActive.DatasetSite.NET](https://github.com/openactive/OpenActive.DatasetSite.NET#simple-implementation).

{% hint style="info" %}
If you have already implemented a dataset site within your application using OpenActive.DatasetSite.NET, the settings should be easily transferable to `EngineConfig.cs`.
{% endhint %}

## Step 6 - Test data feeds and dataset site

If you run your application and navigate to the dataset site endpoint \(e.g. [https://localhost:44307/openactive](https://localhost:44307/openactive)\), you should find it now reflects your updated settings, and the links to the RPDE pages on that page work as expected.





