# Tapcart Integration Guide

[Overview of Shopify Integrations](https://docs.flow.io/docs/integrate-with-shopify)


## Flow's Shopify Metafields and related models

### Localized Item Pricing
Excerpt from [Display Localized Pricing - Server Side](https://docs.flow.io/docs/display-localized-pricing#section-server-side):
"All of the information from Flow is stored in a single String metafield. The data is compressed into a single value to ensure optimal performance when syncing information between Flow and Shopify. Since Shopify API enforces rate limits on how quickly it can provide data, having a single metafield per variant guarantees a deterministic amount of time to sync pricing information."

Below is the model of the metafield we store for each Shopify variant: 
```json
"shopify_variant_flow_metafield": {
    "description": "The shopify variant metafield defines the individual metafield values we write into Shopify for each variant. This model was introduced to enable server side rendering of content (e.g. the price on the product detail page). Each field in this model is available as its own metafield within a namespace named 'price_abc' where abc is a unique, short identifier for an experience.",
    "fields": [
        {
            "name": "prices_item",
            "type": "string",
            "description": "The item price in local currency",
            "example": "C$150.95"
        },
        {
            "name": "prices_currency",
            "type": "string",
            "description": "ISO 4217 3 currency code in upper case of the item price",
            "example": "CAD"
        },
        { 
            "name": "prices_includes",
            "type": "string",
            "required": false,
            "description": "Defines what is included in the item price",
            "example": "Includes VAT" 
        },
        { 
            "name": "prices_vat",
            "type": "string",
            "required": false,
            "description": "The VAT in local currency",
            "example": "C$20.00" 
        },
        { 
            "name": "prices_vat_name",
            "type": "string",
            "required": false,
            "description": "The name of the VAT for this experience country",
            "example": "VAT or HST"
        },
        { 
            "name": "prices_duty",
            "type": "string",
            "required": false,
            "description": "The Duty in local currency",
            "example": "C$10.00" 
        },
        { 
            "name": "prices_compare_at",
            "type": "string",
            "required": false,
            "description": "The compare at item price in local currency",
            "example": "C$250.00" 
        },
        { 
            "name": "prices_status",
            "type": "io.flow.catalog.v0.enums.subcatalog_item_status",
            "description": "Indicates whether the variant should be included or excluded from being sold in the experience."
        },
        { 
            "name": "inventory_status",
            "type": "io.flow.fulfillment.v0.enums.item_availability_status",
            "required": false,
            "description": "The inventory availability for the variant. Can be used to make decisions on how to present products to customers based on inventory condition" 
        }
    ]
}
```

Here are the possible subcatalog item statuses:
```json
"subcatalog_item_status": {
    "description": "Status indicating availability of a subcatalog item in an experience.",
    "values": [
        {
            "name": "excluded",
            "description": "The user has chosen to exclude the item from the associated subcatalog." 
        },
        { 
            "name": "included",
            "description": "The item is included in the associated subcatalog." 
        },
        { 
            "name": "restricted",
            "description": "Item is not allowed to be sold in the market associated with the given subcatalog." 
        }
    ]
}
```

Here are the possible item availability statuses:
```json
"item_availability_status": {
    "values": [
        {
            "name": "available",
            "description": "Inventory is generally available for purchase" 
        },
        {
            "name": "low",
            "description": "Inventory is low and may soon become unavailable for purchase (# inventory items <= 5). Unless there is a specific use case for low inventory, it can be treated the same as 'available'" 
        },
        {
            "name": "out_of_stock",
            "description": "There is no inventory available and is not available for purchase. Sample actions that can be taken are hiding the item or marking as `sold out` on the frontend" 
        }
    ]
}
```

### Redirecting to Checkout
We have multiple options for redirecting users to checkout. The most secure recommendation for custom integrations is with our Checkout Token API [Redirecting with Checkout tokens](https://docs.flow.io/docs/redirect-users-to-checkout-ui#section-redirecting-with-checkout-tokens-server-side-redirects).

To create a Checkout Token, use POST https://api.flow.io/fashionnova-sandbox/checkout/tokens with a body containing a checkout_token_form:
```json
"checkout_token_form": {
  "discriminator": "discriminator",
  "types": [
      {
          "type": "checkout_token_order_form"
      },
      {
          "type": "checkout_token_reference_form",
          "default": true 
      }
  ]
}
```

checkout_token_form accepts one of two possible types, either with an order form (checkout_token_order_form) which creates a new order or a reference to an existing order form (checkout_token_reference_form):
```json
"checkout_token_order_form": {
  "description": "Use this form to securly pass order and optional customer information to be created or updated.",
  "fields": [
      { 
          "name": "order_form",
          "type": "io.flow.experience.v0.models.order_form" 
      },
      { 
          "name": "customer",
          "type": "io.flow.customer.v0.models.customer_form",
          "required": false 
      },
      { 
          "name": "address_book",
          "type": "io.flow.customer.v0.models.customer_address_book_form",
          "required": false 
      },
      { 
          "name": "payment_sources",
          "type": "[io.flow.payment.v0.unions.payment_source_form]",
          "required": false 
      },
      { 
          "name": "session_id",
          "type": "string",
          "description": "We will update the order, if needed, to this session ID" 
      },
      { 
          "name": "urls",
          "type": "checkout_urls_form",
          "required": false 
      },
      { 
          "name": "identifiers",
          "type": "[io.flow.experience.v0.models.order_submission_identifier_form]",
          "description": "Optionally provide one or more order identifiers to attach to the order automatically.",
          "required": false 
      }
  ]
}
```
```json
"checkout_token_reference_form": {
  "description": "Use this form when order number and session id are known. Optional customer information will be created or updated.",
  "fields": [
    {
        "name": "order_number",
        "type": "string" 
    },
    {
        "name": "session_id",
        "type": "string",
        "description": "We will update the order, if needed, to this session ID" 
    },
    {
        "name": "urls",
        "type": "checkout_urls_form" 
    }
  ]
}
```

Checkout URLS:
```json
"checkout_urls_form": {
    "fields": [
    { "name": "continue_shopping", "type": "string", "required": false, "description": "If specified, will be stored on the order in the attribute named 'flow_continue_shopping_url' and will be used as the target URL for when a user chooses to Continue Shopping from Flow Checkout UI" },
    { "name": "confirmation", "type": "string", "required": false, "description": "If specified, will be stored on the order in the attribute named 'flow_confirmation_url' and indicates that instead of showing the Flow Checkout UI Confirmation page, we redirect to this URL instead." },
    { "name": "invalid_checkout", "type": "string", "required": false, "description": "If specified, the user will be redirected to the Invalid Checkout URL when Checkout determines that it cannot proceed with accepting order (e.g. perhaps authorization expired, inventory out of stock, etc). This URL should expect an HTTP Post with an order_put_form as the body." }
    ]
}
```

Order form:
```json
"order_form": {
  "description": "The order form is used to create an open order, providing the details on pricing and delivery options for destination and items/quantities specified",
  "fields": [
      { 
          "name": "customer",
          "type": "io.flow.common.v0.models.order_customer_form",
          "description": "The customer who actually is making the purchase. We recommend providing as much information as you have, notably email address which can be used to increase acceptance rates if Flow is processing payment for this order. If you can also provide your customer number - we can link multiple orders for each customer in the Flow console.",
          "required": false
      },
      { 
          "name": "items",
          "type": "[io.flow.common.v0.models.line_item_form]",
          "minimum": 1 
      },
      { 
          "name": "delivered_duty",
          "type": "io.flow.common.v0.enums.delivered_duty",
          "required": false,
          "description": "Options returned will only use tiers with the matching delivered duty. This would also affect whether duties are included in the total or not. If not specified, defaults based on the experience default setting." 
      },
      { 
          "name": "number",
          "type": "string",
          "required": false,
          "description": "If not provided, will default to the generated unique order identifier." 
      },
      { 
          "name": "destination",
          "type": "order_address",
          "required": false
      },
      { 
          "name": "discount",
          "type": "io.flow.common.v0.models.money",
          "required": false,
          "description": "An optional discount to apply to the entire order" 
      },
      { 
          "name": "discounts",
          "type": "io.flow.common.v0.models.discounts_form",
          "required": false,
          "description": "Optional discount(s) to apply to the entire order." 
      },
      { 
          "name": "attributes",
          "type": "map[string]",
          "description": "A set of key/value pairs that you can attach to the order. It can be useful for storing additional information about the charge in a structured format.",
          "required": false 
      },
      { 
          "name": "authorization_keys",
          "type": "[string]",
          "description": "Sets the authorization keys to associate with this order. Each authorization, if valid, will then be added to the order.payments field.",
          "required": false 
      },
      { 
          "name": "options",
          "type": "order_options",
          "required": false,
          "description": "Optional behaviors to enable for this order" 
      }
  ]
}
```

Order Customer Form:
```json 
"order_customer_form": {
    "fields": [
        { 
            "name": "name",
            "type": "name",
            "required": false,
        },
        { 
            "name": "number",
            "type": "string",
            "required": false,
        },
        { 
            "name": "phone",
            "type": "string",
            "required": false,
            "description": "Customer phone number. Useful for both fraud and order delivery.",
        },
        { 
            "name": "email",
            "type": "string",
            "required": false,
            "description": "Customer email address. Useful for fraud.",
            "example": "user@flow.io" 
        },
        { 
            "name": "address",
            "type": "billing_address",
            "required": false,
        },
        { 
            "name": "invoice",
            "type": "customer_invoice",
            "required": false,
            "description": "Customer invoice details."
        }
    ]
}
```

Line Item Form:
```json
"line_item_form": {
    "description": "Line items represent the items a consumer is purchasing, including additional information to complete the transaction. Note that you may pass in as many line items as you like - including repeating item numbers across line items.",
    "fields": [
        {
            "name": "number",
            "type": "string",

        },
        { 
            "name": "quantity",
            "type": "long",
            "minimum": 1 
        },
        { 
            "name": "shipment_estimate",
            "type": "datetime_range",
            "description": "For items that may not immediately ship out from the origin because of different models of inventory (e.g. drop-ship, sell-first), this is a way for a client to communicate when the items can ship out. This will be used to calculate delivery option windows.",
            "required": false 
        },
        { 
            "name": "price",
            "type": "money",
            "description": "The price of this item for this order. If not specified, we will use the item price from the experience",
            "required": false 
        },
        { 
            "name": "attributes",
            "type": "map[string]",
            "description": "A set of key/value pairs that you can attach to the order. It can be useful for storing additional information about the charge in a structured format.",
            "required": false 
        },
        { 
            "name": "center",
            "type": "string",
            "description": "Optional center key associated with this item. Used for orders and quotes to specify where to ship an item from. If not specified, Flow will infer based on inventory setup.",
            "required": false 
        },
        { 
            "name": "discount",
            "type": "money",
            "description": "The total discount, if any, to apply to this line item. Note that the discount is the total discount to apply regardless of the quantity here",
            "required": false 
        },
        { 
            "name": "discounts",
            "type": "discounts_form",
            "description": "The discounts, if any, to apply to this line item. Note that the discount is the total discount to apply regardless of the quantity here",
            "required": false 
        }
    ]
},
```

Money:
```json
"money": {
  "description": "Money represents an amount in a given currency",
  "fields": [
    { "name": "amount", "type": "double", "example": "100" },
    { "name": "currency", "type": "string", "description": "ISO 4217 3 currency code as defined in https://api.flow.io/reference/currencies", "example": "CAD" }
  ]
}
```

Discounts:
```json
"discounts_form": {
  "fields": [
    { "name": "discounts", "type": "[discount_form]"}
  ]
}
```

Discounts Form:
```json
"discount_form": {
  "fields": [
    { "name": "offer", "type": "discount_offer" },
    { "name": "target", "type": "discount_target", "default": "item", "required": false, "description": "Indicates the target of the discount." },
    { "name": "label", "type": "string", "required": false, "description": "Label to display (e.g. the discount code). Discounts with the same label represent aggregated offers." }
  ]
}
```

Discount Offer:
```json
"discount_offer": {
  "discriminator": "discriminator",
  "types": [
    { "type": "discount_offer_fixed" },
    { "type": "discount_offer_percent" }
  ]
}
```

Discount Offer Percent:
```json
Discount Offer Fixed:
"discount_offer_fixed": {
  "fields": [
    { "name": "money", "type": "money" }
  ]
}
```

Discount Offer Percent:
```json
"discount_offer_percent": {
  "fields": [
    { "name": "percent", "type": "decimal", "minimum": 0, "maximum": 100 }
  ]
}
```

Discount Target:
```json
"discount_target": {
  "values": [
    { "name": "item", "description": "Discount is targeted to an item." },
    { "name": "shipping", "description": "Discount is targeting to shipping. Only applicable if the discount is provided at the order level." }
  ]
}
```

Billing Address:
```json
"billing_address": {
  "fields": [
    { "name": "name", "type": "name", "description": "The name of the customer associated with the billing address", "required": false, "annotations": ["personal_data"] },
    { "name": "streets", "type": "[string]", "description": "Array for street line 1, street line 2, etc., in order", "required": false, "annotations": ["personal_data"] },
    { "name": "city", "type": "string", "required": false },
    { "name": "province", "type": "string", "required": false },
    { "name": "postal", "type": "string", "required": false },
    { "name": "country", "type": "string", "required": false, "description": "The ISO 3166-3 country code. Case insensitive. See https://api.flow.io/reference/countries", "example": "CAN" },
    { "name": "company", "type": "string", "description": "Business entity or organization name of this contact", "required": false , "annotations": ["personal_data"]}
  ]
},
```
