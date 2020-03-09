# Tapcart Integration Guide

[Overview of Shopify Integrations](https://docs.flow.io/docs/integrate-with-shopify)


## Integrating with Shopify's Flow Metafields

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
