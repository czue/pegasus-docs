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
2. Run `python manage.py djstripe_sync_models price` to sync your Stripe products and prices to your local database.
3. Add the product IDs you want to include to the `ACTIVE_ECOMMERCE_PRODUCT_IDS` setting/environment variable.
   In your `.env` file, you can add a list of products by separating them with commas. E.g. 

```
ACTIVE_ECOMMERCE_PRODUCT_IDS="prod_1,prod_2,prod_3"
```

Once you've done this, login and click on the e-commerce tab in the navigation, and you should see your store.

## Webhooks

Like subscriptions, it's recommended to use webhooks to ensure you receive all updates from Stripe.
For the e-commerce store, the only required webhook is `checkout.session.completed`.

Follow [the subscriptions documentation](https://docs.saaspegasus.com/subscriptions/#webhooks) to set
up webhooks in development and production.
