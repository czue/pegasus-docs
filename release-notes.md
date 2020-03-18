

# Version 0.5 (beta)

This release is not yet public. [Get in touch](https://www.saaspegasus.com/support/) if you want to try it out!

## Subscriptions!

- Added a `subscription` field to `Team` and `CustomUser` objects, and a `customer` field to `CustomUser`.
- Added `SubscriptionModelMixin` helper class for accessing / checking subscription status on a model.

## Javascript build changes

- Added `Pegasus.js` and made different modules available in front end code 
  (see subscriptions upgrade page example usage).

## Sass / CSS changes

- Added `tooltip` utilities.
- Added a few margin helper classes (e.g. `my-1`, `my-2` )

## New Library Dependencies

- [attrs][https://www.attrs.org/en/stable/]
- [dj-stripe][https://dj-stripe.readthedocs.io/en/stable/]

## Small fixes and changes:

- Moved app-specific templates from inside the apps to global Pegasus templates directory as recommended by 
  Two Scoops of Django
- Remove redundant raw prefix on some `path` url declarations
- Reduced some duplicate access to `team` object when already available via the `request` object.
- Made team permission template tags more consistent with rest of site (also allow access to superusers)
- Upgraded various `npm` packages
- Removed `PEGASUS_USING_TEAMS` and `pegasus_settings` context processor. All config is now handled at installation
  time instead of by settings variables.


## Documentation

- Added release notes page (this one)
- Added [subscriptions overview page](/subscriptions)
- Updated "delete teams code" cookbook to reflect latest team changes 
  (all the backend work is now done for you on installation)

# Version 0.4 and earlier

Release notes for earlier versions are tracked at [https://www.saaspegasus.com/releases/](https://www.saaspegasus.com/releases/).
