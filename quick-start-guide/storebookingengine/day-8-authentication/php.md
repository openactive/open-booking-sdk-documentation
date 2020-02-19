# PHP

## Purpose of this page

This page contains useful hints for implementing OpenID Connect using PHP. It exists mainly to document learnings from current implementations to aid future implementations while more thorough PHP documentation is yet to be written.

## Guidance for implementing OpenID Connect

[This library](http://bshaffer.github.io/oauth2-server-php-docs/) makes it easy to create an OpenID Connect server in PHP, following the configuration options recommended by the [Open Booking API specification](https://www.openactive.io/open-booking-api/EditorsDraft/#openid-connect-booking-partner-authentication-for-multiple-seller-systems).

Note that the library above does not support the automatic creation of the JSON file for OpenActive ID Connect Discovery. The OpenID Connect Discovery Document can be manually created based on the example [here](https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfigurationResponse), and hosted at the endpoint `/.well-known/openid-configuration`.







