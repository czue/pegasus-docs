Subscriptions
=============

**Note: Subscriptions are a pre-release feature and this documentation is still a work in progress.
To request access to Subscriptions, please email me at cory@saaspegasus.com**

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
2. Update the `STRIPE_*` variables in `settings.py` or in your os environment variables.
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

When you're ready, repeat the same process on your live Stripe account.

Good luck collecting your first subscription payment!
