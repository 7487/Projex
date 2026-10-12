# Projex CLI `--help` drift baseline

Raw `--help` output for every command, captured **2026-10-12** from
`@manningworks/projex` v1.6.0 (branch `skill/projex-agent-skill`, built with
`pnpm build`, invoked as `node packages/core/dist/cli.js`).

How to re-capture: in the repo root, `pnpm install && pnpm build`, then run
`node packages/core/dist/cli.js <args>` for each invocation below and diff.

## `projex --version`

```
1.6.0
```

## `projex --help`

```
Usage: projex [options] [command]

CLI tool for Projex project showcase pages

Options:
  -V, --version                 output the version number
  -h, --help                    display help for command

Commands:
  init [options]                Initialize a new Projex project
  add [options] <type> [id]     Add a Projex component or content to your
                                project
  edit [options] <type> [id]    Edit an existing project
  remove [options] <type> [id]  Remove a project or content entry
  list                          List all projects
  help [command]                display help for command
```

## `projex init --help`

```
Usage: projex init [options]

Initialize a new Projex project

Options:
  --github           Auto-detect projects from GitHub repos
  -y, --yes          Skip all prompts and use default values
  -i, --interactive  Guided interactive setup
  -h, --help         display help for command
```

## `projex add --help`

```
Usage: projex add [options] <type> [id]

Add a Projex component or content to your project

Arguments:
  type                         Content type (project, learning, challenge,
                               timeline, post) or component name
  id                           Project ID (required for learning, timeline,
                               post)

Options:
  -f, --force                  Overwrite existing files without prompting
  --type <type>                Project type (for add project)
  --name <name>                Project name (for add project)
  --status <status>            Project status (for add project)
  --featured                   Mark as featured (for add project)
  --stack <stack>              Comma-separated tech stack (for add project)
  --repo <repo>                GitHub repository (for add project)
  --package <package>          npm package name (for add project)
  --slug <slug>                Product Hunt slug (for add project)
  --channel-id <channelId>     YouTube channel ID (for add project)
  --product-id <productId>     Gumroad product ID (for add project)
  --store-id <storeId>         LemonSqueezy store ID (for add project)
  --username <username>        Dev.to username (for add project)
  --description <description>  Project description (for add project)
  --text <text>                Text content (for add learning)
  --date <date>                Date in YYYY-MM-DD format (for add timeline,
                               post)
  --note <note>                Milestone note (for add timeline)
  --title <title>              Post title (for add post)
  --url <url>                  URL (for add post)
  -h, --help                   display help for command
```

## `projex edit --help`

```
Usage: projex edit [options] <type> [id]

Edit an existing project

Arguments:
  type                         Content type (project)
  id                           Project ID

Options:
  --name <name>                Project name
  --description <description>  Project description
  --status <status>            Project status
  --featured                   Mark as featured
  --no-featured                Remove featured status
  --stack <stack>              Comma-separated tech stack
  --repo <repo>                GitHub repository
  --package <package>          npm package name
  --slug <slug>                Product Hunt slug
  --channel-id <channelId>     YouTube channel ID
  --product-id <productId>     Gumroad product ID
  --store-id <storeId>         LemonSqueezy store ID
  --username <username>        Dev.to username
  --unset <field>              Remove a field from the project
  -h, --help                   display help for command
```

## `projex remove --help`

```
Usage: projex remove [options] <type> [id]

Remove a project or content entry

Arguments:
  type             Content type (project, learning, challenge, timeline, post)
  id               Project ID

Options:
  -f, --force      Skip confirmation prompt
  --index <index>  Index of entry to remove (for learning, timeline, post)
  -h, --help       display help for command
```

## `projex list --help`

```
Usage: projex list [options]

List all projects

Options:
  -h, --help  display help for command
```

## Verified behavioral notes (beyond `--help`)

Captured by executing the CLI in a throwaway Next.js-style sandbox
(2026-10-12, `@manningworks/projex` 1.6.0 from npm):

1. `init -y` — writes `projex.config.ts` scaffold, then auto-installs the
   package. If the install fails, the command exits 1 with
   `✖ Failed to create projex.config.ts` **even though the file was written**.
2. `init --github -y` — GitHub username taken from `git config user.name`.
   Excludes forks; filters archived / `*.template` / `*.github.io` /
   no-description repos. Generated entries include `background: null` and
   `why: null`, which **fail `defineProjects` validation** (`z.string().optional()`
   does not accept null) until those lines are removed.
3. `add project` — prompts (cancels non-interactively, exit 0) when
   `--stack` or `--featured` is omitted. `--stack ""` is accepted and skips
   the stack prompt. There is no `--no-featured` on `add` (edit-only).
4. `add post` — prompts for `--url` even though it is "optional" (cancels
   non-interactively, exit 0, nothing added).
5. `add learning` / `add challenge` — `--text` alone suffices (type defaults
   to `learning` / is forced to `challenge` respectively).
6. `add timeline` — `--date` (YYYY-MM-DD) and `--note` both required.
7. `edit project` — no flags → interactive field picker. `--unset` is
   standalone-only; protected: `id, type, struggles, timeline, posts`.
   Type-specific flags validated against the project type.
8. `remove project` without `--force` — confirm prompt; non-TTY cancels,
   exit 0, project kept.
9. `remove learning/challenge/timeline/post` without `--index` — interactive
   picker; non-TTY cancels, exit 0, nothing removed.
10. `add <component>` — copies to `components/projex/<Name>/`; rewrites
    `from '../types'` → `from '@manningworks/projex'`. From the published
    package only `.d.ts` files are copied (filter matches `.tsx`/`.ts`; dist
    ships `.js` + `.d.ts`) — no runtime code in the copied directory.
11. `add <theme>` — copies to `styles/projex-<name>.css`.
12. All cancelled prompts print `✖ Cancelled.` and exit **0**.
13. `import * as P from '@manningworks/projex'` (ESM, bundler-style
    resolution) exposes 53 exports incl. `ProjectCard`, `ProjectView`,
    `ProjectGrid`, `ProjectList`, `FeaturedProject`, `GitHubCard`, `NpmCard`,
    `ShowcaseCard`, `CommitList`, `ProjectSearch`, `ProjectSort`,
    `ProjectFilterBar`, `ProjectFilterTag`, `ProjectStatusFilterBar`,
    `ProjectTypeFilterBar`, `defineProjects`, `normalise`.
14. Full pipeline verified: `defineProjects([...])` →
    `Promise.all(inputs.map(input => normalise(input)))` → normalized
    `ProjexProject` objects with fetched GitHub data and derived `links`.
