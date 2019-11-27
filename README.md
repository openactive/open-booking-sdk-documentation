# Implementation Options

## What's in the box?

This **Open Booking SDK** will create **Open Opportunity Data Feeds**, a **Dataset Site**, and an [**Open Booking API**](https://www.openactive.io/open-booking-api/EditorsDraft/).

The Open Opportunity Data Feeds and Dataset Site features within the Open Booking SDK leverage libraries that are also available [separately](reference/dependencies.md) for advanced or non-bookable implementations.

## Implementation Options Overview

The library system is designed to be modular, and is designed with the following three implementation routes in mind:

1. **`StoreBookingEngine`** provides the quickest and most prescribed approach. It has a very tightly defined contract, and its API has a minimal surface area - making it the most inherently stable and maintainable route for implementation. It is also the current focus of this guide. 
2. **`CustomBookingEngine`** provides more flexibility, and requires the implementer to do more of the heavy-lifting, including implementing error conditions and validation.
3. **`OpenBookingHelper`** provides helper methods that may be useful for a totally bespoke implementation. They are designed to be used independently, and do not have any dependencies on the other two routes.

The SDK is planned to be made [available](reference/dependencies.md) in **.NET**, **PHP** and **Ruby**. Only the [**.NET version**](https://github.com/openactive/OpenActive.Server.NET) ****is currently available.

## State of Development: Feedback Welcome

{% hint style="info" %}
**`StoreBookingEngine`** for .NET is currently released as a [**Stable Beta**](https://github.com/openactive/OpenActive.Server.NET)**.**
{% endhint %}

The `StoreBookingEngine` approach provides the quickest and most opinionated implementation, and is currently in a stable state ready for early adoption. Early implementers are **strongly encouraged** to use this approach \(which is the focus of this guide, and provided examples\), and provide feedback on any key gaps or constraints which would otherwise have forced them to move towards using the `CustomBookingEngine` or `OpenBookingHelper` approaches. We will move quickly to add features as required in line with your implementation timescales. 

One advantage of being an early implementer is that the `StoreBookingEngine` becomes adapted for your use case - reducing the cost of your implementation and increasing maintainability. Please note that we may not accept all suggestions, in cases where they are very specific to your implementation, and may instead suggest viable alternative approaches.

### Note for advanced implementers

{% hint style="danger" %}
**`CustomBookingEngine`** and **`OpenBookingHelper`** for .NET are available as a **Nightly Build**.
{% endhint %}

The contracts with `CustomBookingEngine` and the `OpenBookingHelper` classes are not yet stable, except where needed to support the `StoreBookingEngine` approach documented in this guide. We encourage exploration and prototyping approaches based solely on these at this stage, and feedback is very welcome as we move to stabilise these over the coming couple of months. Moving functionality between `StoreBookingEngine` and `CustomBookingEngine` \(to make the latter more/less powerful vs customisable\) is a particular area of interest. 

Documentation on the `CustomBookingEngine` and `OpenBookingHelper` classes is primarily within the codebase itself at present - separate guides have not yet been written - although looking at the `StoreBookingEngine` is a good introduction to an example implementation of `CustomBookingEngine`.

