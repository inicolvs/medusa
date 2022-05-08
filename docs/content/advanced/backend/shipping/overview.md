---

title: Shipping Architecture Overview

---

# Architecture Overview

This document gives an overview of the shipping architecture and its 3 most important components.

## Introduction

In Medusa, the Shipping architecture relies on 3 components: **Shipping Profiles**, **Shipping Options**, and **Shipping Methods**.

The distinction between the 3 is important. It has been carefully planned and put together to support all the different ecommerce use cases and shipping providers that can be integrated.

It’s also constructed to support multiple regions, provide different shipment configurations and options for different product types, provide promotional shipments for your customers, and much more.

## Summary

- **Shipping Profiles:** created by the admin. They are used to group products that should be shipped in a different manner than the default. Shipping profiles can have multiple shipping options.
- S**hipping Options:** created by the admin and belong to a shipping profile. They are specific to certain regions and can have cart conditions. They use an underlying fulfillment provider. Once a customer checks out, they can choose the shipping option that’s available and most relevant to them.
- **Shipping Method:** created when the customer chooses a shipping option on checkout. The shipping method is basically a copy of the shipping option, but with values specific to the customer and the cart it’s associated with. When the order is placed, the shipping method will then be associated with the order and fulfilled based on the integration with the fulfillment provider.

:::note

Fulfillment providers are used to ship the products to your customers, whether physically or virtually. An example of a fulfillment provider would be FedEx.

:::

![Shipping.png](https://i.imgur.com/RnC2esy.png)

## Shipping Profile

### Overview

Shipping profiles are the highest in the hierarchy in the shipping architecture.

Shipping profiles are created by the admin. The admin can specify the name of the shipping profile which will be a name that the customer can see.

A shipping profile is not associated with any fulfillment providers. It has multiple shipping options that can be associated with different providers.

### Purpose

Shipping profiles are used to group products that can be shipped in the same manner.

The default shipping profile is one that groups all of your store’s products. You also get a shipping profile that’s specific to gift cards. This is because, generally speaking, all products would be delivered similarly, whereas gift cards would be delivered in different behavior.

Although this might be the general case, there are still some use cases where you will have a set of products that should be shipped differently than others.

For example, shipping heavy items might be more expensive than others, which would enforce different price rates. In that case, you can create a new shipping profile that groups together heavy products. This would allow you to give these products more suitable price rates when creating their shipping options.

### Model Overview

The `ShippingProfile` model can have a set of `Product` instances. These would be the products the shipping profile is providing shipping options for.

The `ShippingProfile` has a `type` attribute that can be `default`, `gift_card`, or `custom`.

The `ShippingProfile` model also has an array of `ShippingOption` instances.

## Shipping Option

### Overview

After the admin adds a shipping profile, they can add shipping options that belong to that shipping profile from the admin dashboard.

Shipping options have a set of conditions like the region they’re available in or cart-specific conditions. For example, if your company operates in the United States as well as Germany, you might use a different shipping option for each of the two countries.

Among the configurations that the admin has to set when creating a shipping option is specifying the fulfillment provider it uses. This means that when you create a plugin for a fulfillment provider, that provider needs to be chosen as the fulfillment provider of a shipping option to be used in the store.

Shipping options are only shown to a customer during checkout if their cart satisfies the option’s conditions. Also, as they belong to a shipping profile, they’re only shown when products that belong to the same shipping profile are in the cart.

### Purpose

The first purpose that a shipping option has is showing the customer during checkout what shipping options are available for them.

Then, once the customer chooses a shipping option, that shipping option is used to create a shipping method with details specific to the customer and their cart. Then, the shipping method is associated with the cart, and the shipping option remains untouched.

Think of a shipping option as a template defined by the admin that indicates what data and values the shipping method should have when it’s chosen by the customer during checkout.

### Model Overview

The `ShippingOption` model belongs to the `ShippingProfile` model.

The `ShippingOption` model also belongs to a `FulfillmentProvider`. This can be either a custom third-party provider or one of Medusa’s default fulfillment providers.

It has the `price_type` attribute to indicate whether the shipping option’s rate is `calculated` by the provider or a fixed `flat_rate` price. It also has the `amount` attribute to set an amount for the shipping option if the `price_type` is `flat_rate`.

`ShippingOption` also belongs to a `Region`, which resembles one or more countries. This defines where the shipping option is available.

`ShippingOption` has a set of `ShippingOptionRequirement` instances. The `ShippingOptionRequirement` model allows defining cart rules which determine whether the shipping option will be available or not for a customer during checkout. For example, you can set a minimum subtotal amount for a shipping option to be available for a customer’s cart.

The `is_return` attribute is used to indicate whether the shipping option is used for shipping orders or returning orders. Shipping options can only be used for one or the other.

The `data` attribute is used to specify any data necessary for fulfilling the shipment based on the underlying fulfillment provider. When you integrate a fulfillment provider, you can check in that provider’s documentation for any data necessary when creating a new shipment.

The `data` attribute does not have any specific format. It’s up to you to choose whatever data is included here.

## Shipping Method

### Overview

Unlike the previous two components, a shipping method is not created by the admin. It’s created when a `POST` request is sent to `/store/carts/:id/shipping-methods` after the customer chooses a shipping option. 

The shipping method will be created based on the chosen shipping option and it’ll be associated with the customer’s cart. Then, when the order is placed, the shipping method is associated with the order.

A shipping method can be fulfilled automatically or manually through the admin dashboard. This is based on the fulfillment provider associated with the shipping option the shipping method is based on.

### Purpose

It’s important to understand the distinction between shipping methods and shipping options. Shipping options are templates created by the admin to indicate what shipping options should be shown to a customer. This provides customization capabilities in a store, as an admin is free to specify configurations for that option such as what fulfillment provider it uses or what are its rates.

When handling the order and fulfilling it, you, as a developer, will be mostly interacting with the shipping method.

This separation allows for developers to implement the custom integration with third-party fulfillment providers as necessary while also ensuring that the admin has full control of their store.

## Model Overview

A lot of the shipping method’s attributes are similar to the shipping option’s attribute.

The `ShippingMethod` model belongs to a `ShippingOption`.

Similar to the `data` attribute explained for the `ShippingOption` model, a `ShippingMethod` has a similar `data` attribute that includes all the data to be sent to the fulfillment provider when fulfilling the order.

The `ShippingMethod` belongs to a `Cart`. This is the cart the customer is checking out with.

The `ShippingMethod` also belongs to the `Order` model. This association is accomplished when the order is placed.

The `ShippingMethod` instance holds a `price` attribute, which will either be the flat rate price or the calculated price.

## What’s Next :rocket:

- [Learn how to Create a Fulfillment Provider.](./add-fulfillment-provider.md)
- Check out [available shipping plugins](https://github.com/medusajs/medusa/tree/master/packages).