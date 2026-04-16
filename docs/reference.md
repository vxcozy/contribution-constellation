# Reference

## contributions.json schema

The file is a direct output of the GitHub GraphQL `contributionCalendar` object.

```json
{
  "totalContributions": 996,
  "weeks": [
    {
      "contributionDays": [
        {
          "contributionCount": 12,
          "date": "2025-04-15",
          "color": "#216e39"
        }
      ]
    }
  ]
}
```

| Field                | Type     | Description                                        |
|----------------------|----------|----------------------------------------------------|
| `totalContributions` | integer  | Sum of all contribution counts across all days.     |
| `weeks`              | array    | Array of week objects. Ordered chronologically.     |
| `weeks[].contributionDays` | array | Array of 7 day objects (Sunday through Saturday). |
| `contributionDays[].contributionCount` | integer | Number of contributions on that day. |
| `contributionDays[].date` | string | ISO 8601 date (`YYYY-MM-DD`).                   |
| `contributionDays[].color` | string | GitHub's hex color for that intensity level. Not used by the visualization (it has its own color mapping). |

The data covers approximately one year of contributions. The exact range is determined by GitHub's API.

## GitHub Action

**File:** `.github/workflows/update-contributions.yml`

**Schedule:** Daily at 06:23 UTC (`23 6 * * *`).

**Trigger:** Also supports `workflow_dispatch` for manual runs.

**Permissions:** Requires `contents: write` to commit the updated JSON.

**What it does:**

1. Checks out the repository.
2. Runs a GraphQL query against `gh api graphql` using `GITHUB_TOKEN`.
3. Pipes the result through `--jq` to extract the `contributionCalendar` object.
4. Writes the result to `contributions.json`.
5. Commits and pushes if the file changed.

The commit is authored by `github-actions[bot]`.

**Environment variables:**

| Variable      | Source                          | Purpose                       |
|---------------|---------------------------------|-------------------------------|
| `GH_TOKEN`    | `secrets.GITHUB_TOKEN`          | Authenticates the GraphQL API |
| `GITHUB_USER` | `github.repository_owner`       | Username to query             |

## Three.js scene parameters

### Grid layout

| Constant      | Value  | Description                                   |
|---------------|--------|-----------------------------------------------|
| `cellSpacing` | `0.38` | Distance between grid cells in world units.   |

Grid X position is calculated as `(weekIndex - weeks.length / 2) * cellSpacing`, centering the grid at the origin. Z position maps day-of-week (0-6) to `(dayIndex - 3) * cellSpacing`. Y is fixed at 0.

### Camera

| Parameter     | Value  |
|---------------|--------|
| FOV           | 50     |
| Near plane    | 0.1    |
| Far plane     | 200    |
| Initial position | `(4, 4, 12)` |

### OrbitControls

| Parameter        | Value   |
|------------------|---------|
| `enablePan`      | `false` |
| `enableDamping`  | `true`  |
| `dampingFactor`  | `0.05`  |
| `minDistance`     | `2`     |
| `maxDistance`     | `18`    |
| `rotateSpeed`    | `0.5`   |
| `autoRotate`     | `true`  |
| `autoRotateSpeed`| `0.15`  |

### Node sizing

For days with contributions:

```
baseSize = 0.06 + (count / maxCount) * 0.12
```

Range: 0.06 (1 contribution) to 0.18 (max contributions).

For days with zero contributions: `baseSize = 0.04`.

Each node has three layers:

| Layer  | Scale | Opacity | Blending  |
|--------|-------|---------|-----------|
| Core   | 1.0x  | 0.9     | Normal    |
| Inner  | 1.8x  | 0.08    | Additive  |
| Outer  | 3.2x  | 0.03    | Additive  |

Zero-contribution days use a single sphere: radius 0.02, opacity 0.15.

### Hitbox

Each cell has an invisible `BoxGeometry` of size `cellSpacing * 0.9` on all three axes for raycasting.

## Color tier mapping

Tiers are assigned by the ratio of a day's count to the year's maximum:

| Tier | Percentage range    | Base color | Glow color |
|------|---------------------|------------|------------|
| 0    | count = 0           | `#222222`  | `#333333`  |
| 1    | 0 < pct <= 20%      | `#0e4429`  | `#1a5c3a`  |
| 2    | 20% < pct <= 40%    | `#006d32`  | `#00913e`  |
| 3    | 40% < pct <= 70%    | `#26a641`  | `#32c755`  |
| 4    | 70% < pct <= 100%   | `#39d353`  | `#4de86a`  |

The thresholds mirror GitHub's quartile approach but use the year's actual maximum rather than fixed buckets.

## Particle systems

### Core particles

| Parameter | Value  |
|-----------|--------|
| Count     | 1500   |
| Color     | `0x39d353` (green) |
| Opacity   | 0.08   |
| Blending  | Additive |
| Distribution | Spherical, radius up to 12 units, Z compressed by 0.3x |
| Scale range | 0.008 -- 0.012 |

### Ambient particles

| Parameter | Value  |
|-----------|--------|
| Count     | 800    |
| Color     | `0xffffff` (white) |
| Opacity   | 0.04   |
| Blending  | Normal |
| Distribution | Box: 40 x 15 x 20 units |
| Scale range | 0.012 -- 0.022 |

### Neural web strands

| Parameter     | Value       |
|---------------|-------------|
| Count         | 40          |
| Color         | `0x39d353`  |
| Base opacity  | 0.01 -- 0.035 |
| Peak opacity  | 0.05        |
| Geometry      | Cubic Bezier curves, 20 segments each |

## Seeded PRNG

All random values use a deterministic PRNG (Lehmer/Park-Miller) so the visualization is identical across page loads.

| Usage            | Seed   |
|------------------|--------|
| Node variations  | 42     |
| Neural strands   | 88888  |
| Core particles   | 77777  |
| Ambient particles| 12345  |

## External dependencies

| Dependency     | Version | CDN                                      |
|----------------|---------|------------------------------------------|
| Three.js       | 0.172.0 | `cdn.jsdelivr.net/npm/three@0.172.0`     |
| OrbitControls  | 0.172.0 | Same CDN, `examples/jsm/controls/`       |
| JetBrains Mono | Latest  | Google Fonts                             |
