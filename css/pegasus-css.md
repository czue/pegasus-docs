# Pegasus CSS

In addition to your app's CSS, Pegasus also ships with its own set of CSS classes to provide compatibility
across different frameworks.
These classes are typically proxies for similar classes provided by the underlying frameworks themselves,
and are created using the Sass [`@extend` helper](https://sass-lang.com/documentation/at-rules/extend)
or Tailwind's [`@apply` helper](https://tailwindcss.com/docs/reusing-styles#extracting-classes-with-apply).

Pegasus CSS classes are defined in `pegasus/<framework>.sass/css`, and they all begin with `pg-`.
You are welcome to leave them in and use them throughout your project, or you can replace them
with the framework-specific names (for example, replacing all instances
of `pg-column` with `column` on Bulma, or `col-md` on Bootstrap).

The following table demonstrates some of the most common Pegasus CSS classes and their corresponding values across frameworks.
If you ever need to look up what a class is doing you can look in `./assets/styles/pegasus/`.

| Pegasus Class   | Description         | Value in Bootstrap | Value in Tailwind                                               | Value in Bulma   |
|-----------------|---------------------|--------------------|-----------------------------------------------------------------|------------------|
| `pg-columns`    | Wrapper for columns | `row gy-4`         | `flex flex-col space-y-4 lg:flex-row lg:space-x-4 lg:space-y-0` | `columns`        |
| `pg-column`     | Individual column   | `col-md`           | `flex-1`                                                        | `column`         |
| `pg-title`      | A title             | `h3` (element)     | `text-3xl font-bold text-gray-900 mb-2`                         | `title`          |
| `pg-subtitle`   | A subtitle          | `lead`             | `text-xl text-gray-900 mb-1`                                    | `subtitle`       |
| `pg-button-***` | A styled button     | `btn btn-***`      | `btn btn-***` (from daisyUI)                                    | `button is-***`  |
| `pg-text-***`   | Colored text        | `text-***`         | `text-***` (from daisyUI)                                       | `has-text-***`   |
