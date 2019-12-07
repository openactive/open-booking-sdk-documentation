# Day 8: Authentication

{% hint style="info" %}
**Note: This functionality is not yet stable, and its release is pending feedback on Day 1-7, and on the design below.**
{% endhint %}

## Objective for Day 8

Configure OpenID Connect and OAuth 2.0 authentication using [IdentityServer4](https://identityserver.io/) to allow Booking Partners to easily and securely gain access to book on behalf of Sellers.

### Rationale

[OpenID Connect](https://openid.net/connect/) and OAuth 2.0 are the Open Booking API recommendation for authentication, as it most easily fulfils the requirements of the specification and is also very widely supported and well understood. Security best practice recommends against creating your own security layer and instead suggests leveraging existing tried-and-tested standards and libraries. IdentityServer4 is the most widely supported option for implementing OpenID Connect and OAuth 2.0 in .NET.

## Guides available

This page covers the overall approach to using OAuth 2.0 and OpenID Connect for the Open Booking API. 

Read this page first, then jump to the appropriate guide:

{% page-ref page="dotnet-core-web-api.md" %}

{% page-ref page="dotnet-framework.md" %}

## Boundaries of responsibility

As you have seen in Day 1 and Day 5, the `StoreBookingEngine` does not include any authentication functionality by design. The endpoint bindings to the StoreBookingEngine accept the `clientId` and `sellerId`, which are expected to be provided by the Access Token in the authentication layer \(shown in red on the diagram below\).  

![Schema to support Open Booking API](../../../.gitbook/assets/booking-system-data-structure-1.png)

| Entity | Description |
| :--- | :--- |
| AuthToken \(JWT\) | IdentityServer4 allows AuthTokens  to contain custom claims, such as the `sellerId` and `clientId`. |
| Booking Partner \(OAuth Client\) | IdentityServer4 manages this as a table of OAuth Clients. |

## Endpoint Access Tokens

An OAuth **scope** defines access to a set of endpoints \(and also expectations about claims returned, see [later](./#claims)\). 

An **Access Token** that includes the required scope \(and which may be acquired via the required flow\) must be included in the Authorization header of the request to access the Open Booking API endpoints:

<table>
  <thead>
    <tr>
      <th style="text-align:left">Flow</th>
      <th style="text-align:left">Scope</th>
      <th style="text-align:left">Endpoints</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">No authentication</td>
      <td style="text-align:left">N/A</td>
      <td style="text-align:left">
        <p>Dataset Site</p>
        <p>Open Data RPDE feeds</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">OpenID Connect Authorization Code Flow</td>
      <td style="text-align:left"><code>openactive-openbooking</code>
      </td>
      <td style="text-align:left">
        <p>OrderQuote Creation (C1)</p>
        <p>OrderQuote Creation (C2)</p>
        <p>OrderQuote Deletion</p>
        <p>Order Creation (B)</p>
        <p>Order Deletion</p>
        <p>Order Cancellation</p>
        <p>Order Status</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Client Credentials flow</td>
      <td style="text-align:left"><code>openactive-ordersfeed</code>
      </td>
      <td style="text-align:left">Orders RPDE Feed</td>
    </tr>
  </tbody>
</table>### OpenID Connect Authorization Code Flow \(`openactive-openbooking`\)

To call endpoints specific to the Seller, the Booking Partner must first acquire a valid **Access Token** with an `openactive-openbooking` scope, by having the Seller complete the Authorization Code Flow. Sellers will be familiar with this flow from websites that offer "Login with my Google Account".

Note however that to complete this flow, the Authorization Request must include both the `openactive-openbooking` and `openid` scopes, to ensure that an ID Token is returned.

A **Refresh Token** is also provided during this flow, which allows the Booking Partner to request another Access Token once it has expired, without the Seller needing to reauthenticate.

Additionally, a "one-time usage" **ID Token** is provided during this flow which contains the SellerId and other details of the Seller. This allows the Booking Partner to store the Access Token and Refresh Token against the correct SellerId in their database, so they can use these when booking the Seller's opportunities.

For this flow, the OpenID Connect subject is recommended **not to be the end user** who is following the OAuth flow, but is instead the **Seller** that they represent - such that if, for example, the end user no longer works for the Seller and deletes their account, their authentication grants remain unaffected. This recommendation conforms with OpenID Connect from a technical perspective, which is useful when leveraging existing libraries.

![OpenID Connect Authorization Code Flow](../../../.gitbook/assets/authorization-code-flow-1.png)

### Client Credentials Flow \(`openactive-ordersfeed`\)

The straightforward Client Credentials Flow can be used to retrieve an **Access Token** with an `openactive-ordersfeed` scope, which grants access to the Orders Feed endpoint as above.

To complete this flow, the Authorization Request must include only the `openactive-ordersfeed` scope.

![Client Credentials Flow](../../../.gitbook/assets/client-credentials-flow.png)

## Access Token expiry

An expiry duration of 15 minutes is recommended for Access Token expiry, to give the Seller control over the relationship.

## Custom Claims

**Claims** are simply key-value pairs that are included in Access Tokens and ID Tokens; each claim is "claiming" a fact about the subject \(in this case, the Seller\). For example a token may include a "claim" that the Seller has a website URL of "https://example.com".

### ID Token claims

The ID Token is designed to be read by the Booking Partner to give them information about the Seller that has just authenticated.  This allows the Booking Partner to store the Access Token and Refresh Token against the correct SellerId in their database, so they can use these when booking the Seller's opportunities.

The `openactive-openbooking` scope includes an implicit request that claims listed below are included in the ID Token.

The following custom claims are for use by the booking partner, and must conform to the custom claim names specified below. The custom claim names are collision-resistant in accordance with the OIDC specification. 

| Custom claim | Description | Exactly matches |
| :--- | :--- | :--- |
| `https://openactive.io/sellerName` | The seller name. | `name` of  `seller` |
| `https://openactive.io/sellerLogo` | A URL of the logo of the Seller. | `logo` of  `seller` |
| `https://openactive.io/sellerUrl` | The URL of the website of the Seller. | `url` of  `seller` |
| `https://openactive.io/sellerId` | The Seller ID as a JSON-LD ID. Also allows for compatibility with existing authentication implementations which might be using "sub" to include a different identifier. Booking partners will use this to determine which Seller ID the provided accessToken is intended for.  | `id` of  `seller` |
| `https://openactive.io/bookingServiceName` | The `name` of the Booking System | `name` of  `bookingService` |
| `https://openactive.io/bookingServiceUrl` | The `url` of the website of the Booking System | `url` of  `bookingService` |

### Access Token claims

To help simplify the implementation, it is recommended that Access Tokens \(which are used to authenticate each request\) include the following custom claims.

The Access Token is only read internally by the Booking System, and so these claims are simply a recommendation. Hence the claim names do not need to be standardised as long as they are internally consistent.

Additionally the **Access Token** may be either a [self-contained or a reference token](http://docs.identityserver.io/en/latest/topics/reference_tokens.html), as it is opaque to the booking partner, however a self-contained token simplifies implementation with IdentityServer4.

| Custom claim | Description | Scopes |
| :--- | :--- | :--- |
| `https://openactive.io/clientId` | Recommended to be used for the booking partner Client ID that requested the Access Token. Note that "[cid](https://developer.okta.com/docs/reference/api/oidc/#access-token-scopes-and-claims)", "client\_id" and similar custom claims may also be available in the libraries you are using by default, and so may be used instead. Also note that this claim is due to be featured in a future OAuth 2.0 specification: [https://tools.ietf.org/html/draft-ietf-oauth-token-exchange-19\#section-4.3](https://tools.ietf.org/html/draft-ietf-oauth-token-exchange-19#section-4.3).  | `openactive-openbooking` and `openactive-ordersfeed` |
| `https://openactive.io/sellerId` | Recommended to be used for the Seller ID, which is useful to be provided to your endpoints to determine which seller the Access Token is intended for. It is also consistent with the claim name used in the ID Token. | `openactive-openbooking` |

