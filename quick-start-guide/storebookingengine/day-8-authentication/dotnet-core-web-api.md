# ASP.NET Core Web API v2.0

{% hint style="info" %}
**Note: This functionality is not yet stable, and its release is pending feedback on Day 1-7, and on the** [**design**](./)**.**
{% endhint %}

## Helpful links

[http://docs.identityserver.io/en/latest/quickstarts/1\_client\_credentials.html](http://docs.identityserver.io/en/latest/quickstarts/1_client_credentials.html)

[https://auth0.com/docs/quickstart/backend/aspnet-core-webapi/01-authorization\#validate-scopes](https://auth0.com/docs/quickstart/backend/aspnet-core-webapi/01-authorization#validate-scopes)

[https://stackoverflow.com/questions/35304038/identityserver4-register-userservice-and-get-users-from-database-in-asp-net-core](https://stackoverflow.com/questions/35304038/identityserver4-register-userservice-and-get-users-from-database-in-asp-net-core)

## Configuring custom claims

### IdentityServer4

This is achieved with a custom ProfileService:

[https://stackoverflow.com/questions/44761058/how-to-add-custom-claims-to-access-token-in-identityserver4](https://stackoverflow.com/questions/44761058/how-to-add-custom-claims-to-access-token-in-identityserver4)

### Auth0

If you are using Auth0 for authentication, the following [rule](https://auth0.com/docs/api-auth/tutorials/adoption/scope-custom-claims) will allow the additional claims to be included in the relevant tokens:

```javascript
function (user, context, callback) {
  const namespace = 'https://openactive.io/';
  const sellerIdBaseUrl = 'https://example.com/api/sellers/';
  context.accessToken[namespace + 'sellerId'] = sellerIdBaseUrl + user.user_id;
  context.accessToken[namespace + 'clientId'] = context.clientID;
  context.idToken[namespace + 'sellerId'] = sellerIdBaseUrl + user.user_id;
  callback(null, user, context);
}
```

## 

