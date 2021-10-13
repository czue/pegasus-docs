Subscriptions
=============

## Overview

Subscriptions in Pegasus have three components which must all be setup in order for them to work correctly.

1. **Stripe Billing data**. This is configured in Stripe.
2. **Local Stripe models**. These are synced automatically from Stripe to your local database, using
   [`dj-stripe`](https://github.com/dj-stripe/dj-stripe).
3. **Pegasus metadata**. This is configured in `apps/subsriptions/metadata.py` and used to augment
   the data from Stripe.

The easiest way to setup all three is to follow the guide below.

## Getting Started

Complete the following steps in order to setup your first subscription workflow.

0. If you haven't already, [setup Pegasus and create an account](/getting-started). 
1. Setup your billing plans in your Stripe test account. See 
   [Stripe's documentation](https://stripe.com/docs/billing/subscriptions/set-up-subscription) for help on doing this.
2. Update the `STRIPE_*` variables in `settings.py` or in your os environment variables to match
   the keys from Stripe. See [this page](https://stripe.com/docs/keys) to find your API keys.
3. Run `./manage.py bootstrap_subscriptions`. If things are setup correctly,
   you should see output that includes "Synchronized plan plan_[plan_id]" for each plan you created,
   and an output starting with `ACTIVE_PRODUCTS = ` containing the products you just created.
4. Paste the `ACTIVE_PRODUCTS` output from the previous step into `apps/subsriptions/metadata.py`
   overriding what is there. Update any other details you want, for example,
   the "description" and "features" fields.
5. Optionally edit the `ACTIVE_PLAN_INTERVALS` variable in `apps/subsriptions/metadata.py`
   if you don't plan to include both monthly and annual offerings.

Now login and click the "Subscription" tab in the navigation. 
If you've set things up correctly you should see a page that looks like this:

![Subscription Example](images/subscription-example.png)

If you want to change the contents of the page, just edit the details in `metadata.py`. Watch how the page 
changes when you make changes.

More background and details on this set up can be found in this 
[Django Stripe Integration Guide](https://www.saaspegasus.com/guides/django-stripe-integrate/).

## Customer Portal

Pegasus uses the [Stripe Billing Customer Portal](https://stripe.com/docs/billing/subscriptions/customer-portal)
for subscription management after subscription creation.

To set up the portal, it's recommended you follow along with [Stripe's integration guide](https://stripe.com/docs/billing/subscriptions/integrating-customer-portal).

To use the portal you will also need to set up webhooks as per below.

Pegasus ships with webhooks to handle some common actions taken in the billing portal, including:

- Subscription upgrades and downgrades
- Subscription cancellation (immediately)
- Subscription cancellations (end of billing period)

In the Stripe dashboard, you will need to subscribe to a minimum of `customer.subscription.updated`
and `customer.subscription.deleted` to ensure subscription changes through the portal make it to your app successfully.

Payment method updates are coming in a future release.

## Webhooks

Webhooks are used to notify your app about events that happen in Stripe, e.g. failed payments.
More information can be found in [Stripe's webhook documentation](https://stripe.com/docs/webhooks).

Pegasus ships with webhook functionality ready to go, including default handling of many events
taken in Stripe's billing portal. That said, you are strongly encouraged
to test locally using [Stripe's excellent guide](https://stripe.com/docs/webhooks/test).

The minimum set of webhooks you should configure in stripe are:

For the billing portal:

- `customer.subscription.deleted` 
- `customer.subscription.updated`

For Stripe Checkout:

- `checkout.session.completed`

A few pieces of setup that are required:

- For the webhook URL, it should be https://yourserver.com/stripe/webhook/. **The trailing slash is required.**
  If using the Stripe CLI in development you can use `stripe listen --forward-to localhost:8000/stripe/webhook/`
- Make sure to set `DJSTRIPE_WEBHOOK_SECRET` in your `settings.py` or environment.
  This value can be found when configuring your webhook endpoint in the Stripe dashboard, 
  or read from the console output in the Stripe CLI.

Once webhooks are properly setup, all the underlying Stripe data will be automatically synced from
Stripe with no additional setup on your part.

### Custom Webhook Handling

You may want to do more than just update the underlying Stripe objects when processing webhooks, 
for example, notifying a customer or admin of a failed payment.

Pegasus ships with an example of executing custom logic from a webhook in `apps/subscriptions/webhooks.py`. 
This basic example will mail your project admins when a Subscription is canceled.

More details on custom webhooks can be found in the [dj-stripe documentation](https://dj-stripe.readthedocs.io/en/stable/usage/webhooks.html).

## Feature-Gating

Pegasus ships with a demo page with a few feature-gating examples, which
are available from a new Pegasus installation under the "Subscription Demo" tab.

These include:

1. Changing content on a page based on the user/team's subscription.
1. Restricting access to an entire page based on the user/team's subscription.
1. Showing subscription details like plan, payment details, and renewal date.

For additional questions on feature-gating don't hesitate to get in touch!

## Troubleshooting

**Stripe is not returning to the right site after accessing checkout or the billing portal.**

There are two settings that determine how Stripe will call back to your site.

If Stripe is returning to the *wrong site entirely* it is likely a problem with your Django `Site` configuration.
See the documentation on [absolute URLs](/configuration.html#absolute-urls) to fix this.

If Stripe is returning to the correct site, *but over HTTP instead of HTTPS* (or vice versa) then you
need to change the `USE_HTTPS_IN_ABSOLUTE_URLS` setting in `settings.py` or a production settings file.
