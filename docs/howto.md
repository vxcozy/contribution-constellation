# How-to guides

## Customize colors

The color scheme is defined in the `TIER_COLORS` array in `index.html`. Each tier has a `base` color (the core sphere) and a `glow` color (the additive-blended halos).

```javascript
const TIER_COLORS = [
  { base: 0x222222, glow: 0x333333 },  // tier 0: no contributions
  { base: 0x0e4429, glow: 0x1a5c3a },  // tier 1: low
  { base: 0x006d32, glow: 0x00913e },  // tier 2: medium-low
  { base: 0x26a641, glow: 0x32c755 },  // tier 3: medium
  { base: 0x39d353, glow: 0x4de86a },  // tier 4: high
];
```

To use a blue theme, for example:

```javascript
const TIER_COLORS = [
  { base: 0x222222, glow: 0x333333 },
  { base: 0x0e2944, glow: 0x1a3c5c },
  { base: 0x00326d, glow: 0x004491 },
  { base: 0x2641a6, glow: 0x3255c7 },
  { base: 0x3953d3, glow: 0x4d6ae8 },
];
```

Also update the neural strand color on this line:

```javascript
color: 0x39d353,  // change to match your tier 4 base
```

And the core particle color:

```javascript
color: 0x39d353,  // same here
```

Both appear in the strand and core particle material definitions.

## Change camera and controls behavior

The `OrbitControls` instance is configured after camera creation:

```javascript
controls.enablePan = false;        // set true to allow panning
controls.enableDamping = true;
controls.dampingFactor = 0.05;     // lower = more momentum
controls.minDistance = 2;          // closest zoom
controls.maxDistance = 18;         // farthest zoom
controls.rotateSpeed = 0.5;       // drag sensitivity
controls.autoRotate = true;       // set false to disable idle rotation
controls.autoRotateSpeed = 0.15;  // degrees per frame at 60fps
```

The three camera presets (perspective, top, front) are defined in `viewPositions`:

```javascript
const viewPositions = {
  perspective: { pos: [4, 4, 12], target: [centerX, 0, 0] },
  top:         { pos: [centerX, 16, 0.01], target: [centerX, 0, 0] },
  front:       { pos: [centerX, 1.2, 12], target: [centerX, 0, 0] },
};
```

Modify the `pos` arrays to change the starting angle. The `target` arrays set the look-at point.

## Add custom back-links

The bottom-right corner contains a `#back-link` div. Replace the placeholder anchor with your own links:

```html
<div id="back-link">
  <a href="https://github.com/yourusername">github</a>
  <a href="https://yourusername.dev">site</a>
  <a href="https://twitter.com/yourusername">twitter</a>
</div>
```

Links are styled with 16px gaps between them. The CSS is in the `#back-link` and `#back-link a` rules.

## Host outside of GitHub Pages

The project is a single HTML file that fetches `contributions.json` via a relative path. It works on any static file host.

1. Run the GitHub Action at least once to generate `contributions.json` (or build it yourself -- see the [reference](reference.md) for the schema).
2. Copy `index.html` and `contributions.json` to your host.
3. Serve them from the same directory.

The page loads Three.js and OrbitControls from `cdn.jsdelivr.net` via an import map, so no build step is needed. The host must serve `.json` files with the correct MIME type (`application/json`). Most static hosts do this by default.

For hosts that require a build step (Vercel, Netlify, etc.), just point the build output to the repository root. There is nothing to compile.

To keep the data fresh without the GitHub Action, set up a cron job that runs the GraphQL query from the workflow file and writes the output to `contributions.json`.

## Use with a GitHub organization account

The Action uses `${{ github.repository_owner }}` as the username for the GraphQL query. For an org-owned fork, this resolves to the org name, which must be a GitHub user (not an org) for the contributions API to return data.

GitHub organizations do not have contribution graphs. To display a specific user's contributions from an org-owned repo:

1. Replace `${{ github.repository_owner }}` in the workflow file with a hardcoded username:

   ```yaml
   GITHUB_USER: "your-username"
   ```

2. The `GITHUB_TOKEN` only needs read access. The default token works if the user's contributions are public. For private contributions, you need a personal access token stored as a repository secret:

   ```yaml
   GH_TOKEN: ${{ secrets.PAT_TOKEN }}
   ```

   The token needs the `read:user` scope.
