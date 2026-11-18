# SysML Fabric Pages

Publish a SysML v2 model to **GitHub Pages**: an interactive, read-only site of
its diagrams, and the model itself as a **`.kpar`**, the SysML v2 project
interchange archive other SysML v2 tools import. Rebuilt on every push, so
nothing goes stale. With `images: true` it also writes an **SVG and a PNG of
every view** at stable addresses, to embed in MkDocs, Docusaurus, Confluence, a
README or any documentation that takes an image link.

**For public repositories.** In a private or internal repository the action
stops with an error before it builds anything.

Write the model with [SysML Fabric](https://marketplace.visualstudio.com/items?itemName=ue-sys.sysml-fabric)
(VS Code, free) or the SysML Fabric web app; the layout you drag into place is
saved beside it in `.fabric` files, and this action draws exactly that.

## Use

First, once: **Settings → Pages → Build and deployment → Source: GitHub
Actions** (before the first run, or its deploy fails). Then add
`.github/workflows/sysml-fabric-pages.yml` to your repository — nothing else is
copied into it; this action runs from here:

```yaml
name: SysML diagrams
on:
  push:
    branches: [main]
  workflow_dispatch:
permissions:
  contents: read
  pages: write
  id-token: write
concurrency:
  group: pages
  cancel-in-progress: false
jobs:
  pages:
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deploy.outputs.page_url }}
    steps:
      - uses: actions/checkout@v7
      - uses: ue-sys/sysml-fabric-pages@v1
        with:
          folder: model   # where the .sysml files are; default: the whole repository
      - id: deploy
        uses: actions/deploy-pages@v5
```

The site is at `https://<owner>.github.io/<repo>/`. With `images: true`, each
view's image is at `views/<model file>/<view>-<kind>.svg` (and `.png`) under it — for example
`views/vehicle/SystemView-ibd.svg` for the `SystemView` view, drawn as an
internal block diagram, in `vehicle.sysml`. The run's summary lists every image
with the Markdown to embed it.

The `.kpar` is the run's **workflow artifact** (*Actions → the run →
Artifacts*), `<title>.kpar` itself, not zipped again — so a matrix over several
model folders sets a different `title` for each. The action's `kpar` output is
its path for a later step (attach it to a release, push it to a registry). It
holds the `.sysml` files only — never layout — with the index, checksums and
metamodel the standard asks for. Filter it with `kpar-include` /
`kpar-exclude`, globs read as `.gitignore` reads them: one package per file
makes a file glob a package filter, so `kpar-exclude: drafts, **/*-wip.sysml`
keeps work in progress out of what you hand on. A filter that leaves nothing
writes no archive (a warning says so). It is not deployed to Pages.

| Input | Default | |
|---|---|---|
| `folder` | `.` | the model folder, relative to the repository root |
| `title` | the repository's name | the site's title |
| `images` | `false` | `true` to render `views/` (needs Chrome — GitHub's ubuntu runners have it; elsewhere set `CHROME_BIN`) |
| `system-views` | `false` | also render each file's built-in Containment / General View / Interconnection |
| `path` | `_site` | where the site is written |
| `upload` | `true` | upload it as the Pages artifact; `false` to use the files in a later step instead (your own docs build, a commit) |
| `kpar` | `true` | write the model as a `.kpar` and upload it as a workflow artifact |
| `kpar-include` | every model file | `.gitignore`-style globs, relative to `folder`, comma- or newline-separated: only these go in |
| `kpar-exclude` | none | globs, as above: these stay out (after `kpar-include`) |
| `kpar-version` | the run's tag (`v1.2.0` → `1.2.0`), else `0.0.0` | the archive's version |

What it cannot draw: a library `libraries.json` names from a SysML Fabric
server's enterprise or personal tier (a warning names it; vendor it as a
`.sysmllib` file instead); a PNG or JPG image shape; a matrix view, which is a
table, not a picture.

Proprietary software of Unified Engineering Systems, free to use under its
[licence](LICENSE); the model, site and images it makes are yours.
Third-party components: `THIRD_PARTY_NOTICES`.

