# Troubleshooting

Below are some common issues related to deployment, and how to fix them.

### Page displaying a 400 Bad Request error page

**Problem:** 

Your site deploys but you get a "400 Bad Request" when opening it in a browser.

**Solution:**

This is usually caused by a misconfigured `ALLOWED_HOSTS` setting.
See [the section on `ALLOWED_HOSTS`](https://docs.saaspegasus.com/deployment/production-checklist.html#set-your-allowed-hosts) to fix.

### JavaScript API clients not working

**Problem**

JavaScript API clients are failing to load data.
This is likely the problem if the employee React demo or the teams list UI don't work properly.

**Solution:**

This is usually caused by a misconfigured Django site.
See the documentation on [absolute URLs](https://docs.saaspegasus.com/configuration.html#absolute-urls) to fix.

### Invitation / account emails have the wrong links

**Problem**

When you try to confirm an email address or accept a team invitation you are sent to the wrong site
(e.g. localhost).

**Solution:**

This is usually caused by a misconfigured Django site.
See the documentation on [absolute URLs](https://docs.saaspegasus.com/configuration.html#absolute-urls) to fix.

### Stripe callbacks are going to the wrong place

**Problem**

After completing a payment in Stripe Checkout, you are redirected to the wrong place (e.g. localhost).

**Solution:**

This is usually caused by a misconfigured Django site.
See the documentation on [absolute URLs](https://docs.saaspegasus.com/configuration.html#absolute-urls) to fix.
