Marketplace
===========

## How to connect a marketplace app to a Pegasus project

Follow these steps to connect a marketplace app to a Pegasus project.
We'll do this for [scriv], but the process should be the same for other apps built on Pegasus.

First, [follow the getting started guide](https://www.saaspegasus.com/store/product/scriv/get-started/)
to clone the repository and set up your own fork.

Next, create new Pegasus project for your app from [the projects page](https://www.saaspegasus.com/projects/).
**You should choose the same config choices as the marketplace app**,
which you can find in the `pegasus-config.yaml` file in the repository root.

You can also refer to [this screenshot](/images/marketplace/scriv-pegasus-config.png) for how the configuration
should look as of September, 2024.

Next, connect your Pegasus project to your Github repository:

1. From the project page, select "Get Code".
2. On the "Push to Github" tab, click "Add a repo".
3. Set the owner and name to the fork you created above.
4. Click "Add Repo".

You will be prompted to authenticate with Github.
If you haven't already, you should set that up according to [the Github guide](https://docs.saaspegasus.com/github/#connecting-your-account).

After connecting your Github, you will be prompted to enter a commit ID according to the last Pegasus commit
in the repository. You should enter the commit ID of the most recent Pegasus update, which you can
get from the table below:

| Codebase            | Commit ID                                  | Last Updated |
|---------------------|--------------------------------------------|--------------|
| Scriv               | `2e6a656778a55e115e236680e37fe2773dcc3bd4` | Sep 4, 2024  |
| Translation Creator | `60595de93e05161de3cd2a25678da7ddd9ac0709` | Sep 4, 2024  |

Enter the commit ID and click "Set Commit ID".

Now you should be able to click "Submit Pull Request" and your Pegasus project will sync with your marketplace codebase.
Once this completes you should have a Github pull request with a small number of changes.
You may need to resolve conflicts on this pull request (when in doubt, accept the `main` branch changes).
Once you have resolved all conflicts, merge the pull request.

Your marketplace is now connected to your Pegasus project!
You can change settings, upgrade, and submit pull requests from Pegasus and they will go to your marketplace app repo.

If you have any questions or issues with this process, [get in touch via the support channels](https://www.saaspegasus.com/support/).
