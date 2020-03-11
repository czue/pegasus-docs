Subscriptions
=============

**Note: Subscriptions are a pre-release feature and this documentation is still a work in progress.
To request access to Subscriptions, please email me at cory@saaspegasus.com**

## Getting Started

Complete the following steps in order to setup your first subscription workflow.

0. If you haven't already, [setup Pegasus and create an account](/getting-started). 
1. Setup your billing plans in your Stripe test account. See 
   [Stripe's documentation](https://stripe.com/docs/billing/subscriptions/set-up-subscription) for help on doing this.
2. Update the `STRIPE_*` variables in `settings.py` or in your os environment variables.
3. Run `./manage.py djstripe_sync_plans_from_stripe`. If things are setup correctly, 
   you should see output that includes "Synchronized plan plan_[plan_id]" for each plan you created. 
4. Edit the `ACTIVE_PLAN_INTERVALS` and `ACTIVE_PRODUCTS` variables in `apps/subsriptions/metadata.py`
   to match the ones you setup in Stripe.
   Specifically, make sure the `stripe_id` of each product points to a valid product ID you setup when you 
   configured billing on Stripe, and update any other details you want.

Now login and click the "Subscription" tab in the navigation. 
If you've set things up correctly you should see a page that looks like this:

![Subscription Example](images/subscription-example.png)

If you want to change the contents of the page, just edit the details in `metadata.py`. Watch how the page 
changes when you make changes.

When you're ready, repeat the same process on your live Stripe account.

Good luck collecting your first subscription payment!
