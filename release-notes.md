

# Version 0.4

## Subscriptions!

- Added a `subscription` field to `Team` and `CustomUser` objects, and a `customer` field to `CustomUser`.
- Added `SubscriptionModelMixin` helper class for accessing / checking subscription status on a model.

## Javascript build changes

- Added `Pegasus.js` and made different modules available in front end code 
  (see subscriptions upgrade page example usage).

## SaaS / CSS changes

- Added `tooltip` utilities.
- Added a few margin helper classes (e.g. `my-1`, `my-2` )

## New Library Dependencies

- [attrs][https://www.attrs.org/en/stable/]
- [dj-stripe][https://dj-stripe.readthedocs.io/en/stable/]

## Small fixes and changes:

- Remove redundant raw prefix on some `path` url declarations
- Reduced some duplicate access to `team` object when already available via the `request` object.

