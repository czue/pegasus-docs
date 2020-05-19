# Using Payments

The following guide explains how to incorporate the payments demo that ships with Pegasus into your own application.

This is a tedious process at the moment, and making it more seamless is a priority.

In the meantime, hopefully it helps show all the steps for doing this and how it could be
applied to other examples.

## Create the payments app

First create the app:

```bash
mkdir apps/payments
./manage.py startapp payments apps/payments
```

Then, in the created `apps/payments/apps.py` file, 
change the `name = 'apps.payments'` to fix the path.

Finally, in `settings.py` add `'apps.payments.apps.PaymentsConfig',` to `PROJECT_APPS`.

## Setup Database Models

First copy the models file from the example into your new app: 

```bash
cp pegasus/apps/examples/models/payments.py apps/payments/models.py
```

Then change the `related_name` attribute on the `user` field to `'payments'` 
(this is to prevent a conflict with the Pegasus example payment model).

Create migrations and migrate:

```bash
./manage.py makemigrations
./manage.py migrate
```

## Setup UI

### Create Views

Start with the relevant example views

```bash
cp pegasus/apps/examples/views/payments.py apps/payments/views.py 
```

You'll need to fix the models import (remove one of the periods from the import so it
reads like this:

```python
from .models import Payment
```

Then change the template references by replacing `pegasus/examples/payments/` with `payments/` throughout
the file. (We'll create these templates next.)

And fix the URL references by replacing `pegasus_examples:` with `payments:` throughout. 
(We'll also create these momentarily.)

### Create Templates

First copy the example templates into the new app:

```bash
cp -r pegasus/apps/examples/templates/pegasus/examples/payments/ apps/payments/templates/payments/
```

Then edit the first line of `apps/payments/templates/payments/payments.html` to extend your app template
instead of the example template:

```html
{% extends "web/app/app_base.html" %}
```

And find the line containing the `<form>` definition and change it as follows (changing the url
from `pegasus_examples`):

```html
<form action="{% url 'payments:accept_payment' %}" method="post" id="payment-form">
```

### Create URLs

Create a `urls.py` file in `apps/payments` and make sure its contents look like this:

```python
from django.urls import path

from . import views


app_name = 'payments'

urlpatterns = [
    path(r'', views.PaymentView.as_view(), name='payments'),
    path(r'create/', views.accept_payment, name='accept_payment'),
    path(r'confirm/<slug:payment_id>/', views.payment_confirm, name='payment_confirm'),
]
```

Then in your main project `urls.py` file add the following line to `urlpatterns`:

```python
    path('payments/', include('apps.payments.urls')),
```

### Update Navigation

To add a navigation element to your new page, just paste the following snippet in one of the menus:

```html
<li>
  <a href="{% url 'payments:payments' %}">
    <span class="icon"><i class="fa fa-dollar"></i></span><span class="is-hidden-tablet-only">Payments</span>
  </a>
</li>
```

## Success!

If you've successfully completed these steps you should be able to go to click on the navigation 
element (or visit [http://localhost:8000/payments/](http://localhost:8000/payments/)) to see your 
new payment module working.
 
**Note: if payments are giving a Stripe error you likely need to fix your 
[Stripe configuration](configuration.md).**
