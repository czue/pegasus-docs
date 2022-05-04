Getting Started
===============

Here's everything you need to start your first Pegasus project.

## Watch the video

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 1em;">
    <iframe src="https://www.youtube.com/embed/mod5WwUWOZw" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>

Visual learner? The above video should get you going.
Else read on below for the play-by-play.

## Create and download your project codebase

If you haven't already, you'll need to [purchase a Pegasus License on saaspegasus.com](http://www.saaspegasus.com/licenses/).

Then, [create a new project on saaspegasus.com](https://www.saaspegasus.com/projects/),
following the prompts and filling in whatever configuration options you want to use for your new project.
Make sure that the "license" field at the bottom is set.

Once you're done, download your project's source code as a zip file.
Unzip it to a folder where you want to do your development and you're ready to go!

## Set up source control

It is highly recommended to use git for source control.
[Install git](https://git-scm.com/downloads) and then run the following commands:

```
git init
git add .
git commit -am "initial project creation"
```

It is also recommended to create a `pegasus` branch at this time for future upgrades.

```
git branch pegasus
```

You can read [more about upgrading here](/upgrading).

## Get up and running

There are two supported ways of running your project.

The quickest way to get up and running is to use Docker for development.
For that option, continue to the [Docker instructions](/docker).

For a more customizable approach that is more effort to set up, but can be easier to maintain,
you can follow the instructions on [native installation](/native).

After you've gotten up and running, you'll likely want to complete some [post-installation steps](/post-install).
