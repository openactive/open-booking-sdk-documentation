# Day 8: Authentication

{% hint style="info" %}
**Note: This functionality is not yet stable, and its release is pending feedback on Day 1-7**
{% endhint %}

## Objective for Day 8

Configure OpenID Connect authentication using [IdentityServer4](https://identityserver.io/) to allow Booking Partners to easily and securely gain access to book on behalf of Sellers.

### Rationale

[OpenID Connect](https://openid.net/connect/) is the Open Booking API recommendation for authentication, as it most easily fulfils the requirements of the specification and is also very widely supported and well understood. Security best practice recommends against creating your own security layer and instead suggests leveraging existing tried-and-tested standards and libraries. IdentityServer4 is the most widely supported option for implementing OpenID Connect in .NET.

## Step 1: Understanding the boundaries of responsibility

As you have seen in Day 1, the `StoreBookingEngine` does not include any authentication functionality by design. The endpoint bindings to the StoreBookingEngine accept the `bookingPartnerClientId` and `sellerId`, which are expected to be provided by authentication.  

![Database structure to support Open Booking API](../../.gitbook/assets/booking-system-data-structure.png)

| Entity | Description |
| :--- | :--- |
|  |  |

Step 2: Configure AuthToken expiry

An expiry duration of 15 minutes is recommended for AuthToken expiry, to give the Seller control over the relationship.

[https://stackoverflow.com/questions/35304038/identityserver4-register-userservice-and-get-users-from-database-in-asp-net-core](https://stackoverflow.com/questions/35304038/identityserver4-register-userservice-and-get-users-from-database-in-asp-net-core)

Custom claim "https://schema.org/seller" is used for the Seller ID, which is collision-resistant in accordance with the OIDC specification, and also allows for compatibility with existing authentication implementations which might be using "sub" already.

Is the Booking Partner ID also a custom claim? Surely we must know this from the authToken?

