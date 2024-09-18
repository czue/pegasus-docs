# E-Commerce / Payments

Pegasus (version 2023.9.1 and up) includes an out-of-the-box E-Commerce/Payments demo.
In a few clicks you can have a fully functional digital storefront in your application,
allowing you to collect and track one-time or recurring payments with Stripe.

## Watch a video

To see how this feature works, you can watch the following video:

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 1em;">
    <iframe src="https://www.youtube.com/embed/S4LlQtGD1jc" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>

## Getting Started

### Set up Stripe Products

First add your products in the Stripe dashboard.
Be sure to add readable product names, descriptions, and images, as these
will be used for the in-app store.
Additionally, make sure each product includes at least one Price.

### Set up your development environment

Setting up your development is similar to the [process for subscriptions](./subscriptions.md), but has fewer steps.

1. If you haven't already, update the `STRIPE_*` variables in `settings.py` or in your os environment variables to match
   the keys from Stripe. See [this page](https://stripe.com/docs/keys) to find your API keys.
2. Run `python manage.py bootstrap_ecommerce` to sync your Stripe products and prices to your local database.

Once you've done this, login and click on the e-commerce tab in the navigation, and you should see your store.

## Data models

### `ProductConfiguration`

What shows up in your store is controlled by the `ProductConfiguration` data model.
You can manage these objects from the Django admin (available at 
[http://localhost:8000/admin/ecommerce/productconfiguration/](http://localhost:8000/admin/ecommerce/productconfiguration/) locally).
For example, to remove a product from the store you can uncheck "is active".

The `ProductConfiguration` model is also a good place to add additional information to your products.
For example, you can add additional display data there, or add a `FileField` if you want purchases to grant access
to a digital download.

### `Purchase`

The `Purchase` model is used to record user purchases.
A `Purchase` is associated with a `User` and a `ProductConfiguration` and also has details of the Stripe checkout session,
date of purchase, and product/price used at the time of purchase.

## Feature gating

The `@product_required` decorator can be used to restrict access to a view based on whether or not
the logged-in user has purchased a particular product. This decorator expects a `product_slug` field
in the URL / view with the slug of the `ProductConfiguration` object to be checked. 
If the user owns the product, they will be granted access to the view.

Additionally, if the user gets access, two additional field will be populated on the `request` object:

- `request.product_config` will have the `ProductConfiguration` object.
- `request.product_purchase` will have the `Purchase` object.

If the user does *not* have access to the product, the decorator will redirect them back to the store homepage.

## Webhooks

Like subscriptions, it's recommended to use webhooks to ensure you receive all updates from Stripe.
For the e-commerce store, the only required webhook is `checkout.session.completed`.

Follow [the subscriptions documentation](subscriptions.md#webhooks) to set
up webhooks in development and production.
