# Day 4: C1 and C2 without leases

## Step 1: GetOrderItem

Implement the `GetOrderItem` within the implementations of ****`OpportunityStore` created in Day 3.

## Step 2: StoreBookingEngineSettings

Customise the following `StoreBookingEngineSettings` within `Startup.cs` or `ServiceConfig.cs` :

* `CustomerPersonSupportedFields`
* `CustomerOrganizationSupportedFields`
* `BrokerSupportedFields`
* `BookingServiceDetails`

## **Step 3: SellerStore**

Implement a new `SellerStore`, with the method `GetSeller`.

Configure the `SellerStore` setting of  `StoreBookingEngineSettings` within `Startup.cs` or `ServiceConfig.cs`.

Implement the `SellerIdTemplate` setting of  `StoreBookingEngineSettings` within `Startup.cs` or `ServiceConfig.cs`:

```csharp
SellerIdTemplate = new SingleIdTemplate<SellerIdComponents>(
    "{+BaseUrl}api/sellers/{SellerIdLong}"
    ),
```

## **Step 4: Test Suite**

Tests should pass for C1 and C2  


