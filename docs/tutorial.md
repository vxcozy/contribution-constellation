# Tutorial: Deploy your contribution constellation

This walkthrough takes you from zero to a live 3D contribution graph hosted on GitHub Pages. Estimated time: 5 minutes.

## Prerequisites

- A GitHub account with some contribution history.
- A browser that supports ES modules and WebGL (all modern browsers).

## Step 1: Fork the repository

Go to the repository page and click **Fork**. Keep the default name or rename it. The fork must be owned by the account whose contributions you want to display -- the Action uses the repository owner's username to query the API.

## Step 2: Enable GitHub Pages

1. In your fork, go to **Settings > Pages**.
2. Under **Source**, select **Deploy from a branch**.
3. Set the branch to `main` and the folder to `/ (root)`.
4. Click **Save**.

GitHub will take a minute or two to build the initial deployment. The URL will be `https://<username>.github.io/contribution-constellation/`.

## Step 3: Enable and trigger the Action

Forked repositories have Actions disabled by default.

1. Go to the **Actions** tab in your fork.
2. Click **I understand my workflows, go ahead and enable them**.
3. In the left sidebar, click **Update contribution data**.
4. Click **Run workflow** > **Run workflow** (on the `main` branch).

The workflow fetches your contribution data from the GitHub GraphQL API and commits `contributions.json` to the repository. This takes about 30 seconds.

## Step 4: Verify

Visit your Pages URL. You should see a black background with green orbs arranged in a grid pattern matching your contribution history. The overlay in the top-left corner shows your total contribution count.

If the page is blank or shows the sample data, check:

- Did the Action complete successfully? (Look for a green checkmark on the Actions tab.)
- Did `contributions.json` update? (Check the file's last commit date.)
- Is Pages deployed? (Settings > Pages should show a live URL.)

## Step 5: Embed in your GitHub profile README

To link from your profile README:

```markdown
[View my contribution constellation](https://yourusername.github.io/contribution-constellation/)
```

To embed as an iframe (works in GitHub profile READMEs rendered outside github.com, personal sites, etc.):

```html
<iframe
  src="https://yourusername.github.io/contribution-constellation/"
  width="100%"
  height="400"
  frameborder="0"
></iframe>
```

Note: GitHub's markdown renderer strips iframes. The iframe approach only works on external sites.

## What happens next

The Action runs daily at 06:23 UTC. Each run fetches the latest contribution data and commits the updated JSON. No further maintenance is needed.
