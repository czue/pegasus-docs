Wagtail CMS
===========

[Wagtail](https://wagtail.org/) is a powerful CMS (Content Management System) built on top of Django.
You can use it to create rich websites that can be edited directly via an authoring admin interface without writing any code.
It's great for creating marketing sites, blogs, and other mostly-static content.

Pegasus optionally ships with a built-in Wagtail instance that can be used as a starting point
for adding a content section and blog to any Pegasus app.

## Pegasus and Wagtail

If you want to try Wagtail make sure you enable the "Use Wagtail" option in the Pegasus codebase creator.

Out-of-the-box, Pegasus will create a "content" are of your site (available at the `/content/` URL),
a blog index page (available at `/content/blog/`) and a few example blog posts.
All your content can be edited via the Wagtail admin UI (available to superusers at `/cms/` by default).

The data models for your app's content are in the `apps/content/` folder, and can be modified or extended
in the `models.py` folder there.

For more information on Wagtail, check out their [excellent documentation](https://docs.wagtail.org/).

## Alternatives to Wagtail

Some companies prefer to manage their marketing sites completely separate from their application.
In this scenario it's recommended to create a separate marketing site using something like
Wordpress, Webflow, Wix, Squarespace, or any number of other options. You can host this site
at `yourdomain.com` and then host your Pegasus app separately at `app.yourdomain.com` (or similar).

If you choose to set up your content this way, you likely don't want to include wagtail in your Pegasus build.
