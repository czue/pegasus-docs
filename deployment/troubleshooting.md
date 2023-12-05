# Troubleshooting

Below are some common issues related to deployment, and how to fix them.

### Page displaying a 400 Bad Request error page

**Problem:** 

Your site deploys but you get a "400 Bad Request" when opening it in a browser.

**Solution:**

This is usually caused by a misconfigured `ALLOWED_HOSTS` setting.
See [the section on `ALLOWED_HOSTS`](set-your-allowed-hosts) to fix.

### App is online but all styles are broken

**Problem:**

Your app is working but all the pages look horrible and unstyled.

**Solution:**

It's likely that your static file set up is not correct.
If you use Pegasus-supported deployments, this shouldn't happen,
but if you've veered from them at all it's a common failure mode.

To fix, Ensure that you are running `collectstatic` somewhere in your deployment pipeline,
and that your `STATIC_ROOT` is properly configured.
More on static files in production can be found in the [Django documentation](https://docs.djangoproject.com/en/4.1/howto/static-files/deployment/).

### JavaScript API clients not working

**Problem**

JavaScript API clients are failing to load data.
This is likely the problem if the employee React demo or the teams list UI don't work properly.

**Solution:**

This is usually caused by a misconfigured Django site.
See the documentation on [absolute URLs](../configuration.md#absolute-urls) to fix.

### Invitation / account emails have the wrong links

**Problem**

When you try to confirm an email address or accept a team invitation you are sent to the wrong site
(e.g. localhost).

**Solution:**

This is usually caused by a misconfigured Django site.
See the documentation on [absolute URLs](../configuration.md#absolute-urls) to fix.

### Stripe callbacks are going to the wrong place

**Problem**

After completing a payment in Stripe Checkout, you are redirected to the wrong place (e.g. localhost).

**Solution:**

This is usually caused by a misconfigured Django site.
See the documentation on [absolute URLs](../configuration.md#absolute-urls) to fix.
