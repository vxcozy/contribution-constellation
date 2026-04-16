# contribution-constellation

A 3D visualization of your GitHub contribution graph, rendered as glowing orbs in space using Three.js.

![Screenshot placeholder](docs/screenshot-placeholder.png)

## Setup

1. Fork this repository.
2. Go to **Settings > Pages** and set the source to the `main` branch.
3. Go to **Actions** and enable workflows (they are disabled by default on forks).
4. Manually trigger the "Update contribution data" workflow, or wait for the daily schedule.
5. Visit `https://<your-username>.github.io/contribution-constellation/`.

The GitHub Action runs daily at 06:23 UTC. It fetches your contribution data via the GitHub GraphQL API and commits a fresh `contributions.json`. GitHub Pages serves the result.

## Documentation

- [Tutorial](docs/tutorial.md) -- first-time setup walkthrough
- [How-to guides](docs/howto.md) -- customize colors, camera, hosting, etc.
- [Reference](docs/reference.md) -- data schema, action config, scene constants
- [Explanation](docs/explanation.md) -- how the 3D mapping, glow, and particles work

## License

MIT
