# Day 4: C1 and C2 without leases

## Step 1: GetOrderItem

Implement the `GetOrderItem` within the implementations of ****`OpportunityStore` created in Day 3.

## Step 2: Supported Fields

Customise the following `StoreBookingEngineSettings` within `Startup.cs` or `ServiceConfig.cs` to include only those fields that your booking system supports \(as these must be reflected back to the Broker\):

* `CustomerPersonSupportedFields`
* `CustomerOrganizationSupportedFields`
* `BrokerSupportedFields`

## **Step 3: Booking Service Details**

Complete the details in the setting below to include information about your booking system:

* `BookingServiceDetails`

## **Step 4: SellerStore**

Implement a new `SellerStore`, with the method `GetSeller`.

Configure the `SellerStore` setting of  `StoreBookingEngineSettings` within `Startup.cs` or `ServiceConfig.cs`.

Implement the `SellerIdTemplate` setting of  `StoreBookingEngineSettings` within `Startup.cs` or `ServiceConfig.cs`:

```csharp
SellerIdTemplate = new SingleIdTemplate<SellerIdComponents>(
    "{+BaseUrl}api/sellers/{SellerIdLong}"
    ),
```

## **Step 5: Test Suite**

Tests should pass for C1 and C2  


