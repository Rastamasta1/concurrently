# Command Shortcuts

Package managers that execute scripts from a `package.json` / `package.json5` or `deno.(json|jsonc)` file can be shortened when in concurrently.<br/>
The following are supported:

| Syntax          | Expands to            |
| --------------- | --------------------- |
| `npm:<script>`  | `npm run <script>`    |
| `pnpm:<script>` | `pnpm run <script>`   |
| `yarn:<script>` | `yarn run <script>`   |
| `bun:<script>`  | `bun run <script>`    |
| `node:<script>` | `node --run <script>` |
| `deno:<script>` | `deno task <script>`  |

> [!NOTE]
>
> `node --run` is only available from [Node 22 onwards](https://nodejs.org/en/blog/announcements/v22-release-announce#running-packagejson-scripts).

For example, given the following `package.json` contents:

```jsonc
{
  // ...
  "scripts": {
    "lint:js": "...",
    "lint:ts": "...",
    "lint:fix:js": "...",
    "lint:fix:ts": "...",
    // ...
  },
  // ...
}
```

It's possible to run some of these with the following command line:

```bash
$ concurrently 'pnpm:lint:js'
# Is equivalent to
$ concurrently -n lint:js 'pnpm run lint:js'
```

Note that the command automatically receives a name equal to the script name.

If you have several scripts with similar name patterns, you can use the `*` wildcard to run all of them at once.<br/>
The spawned commands will receive names set to whatever the `*` wildcard matched.

```bash
$ concurrently 'npm:lint:fix:*'
# is equivalent to
$ concurrently -n js,ts 'npm run lint:fix:js' 'npm run lint:fix:ts'
```

If you specify a command name when using wildcards, it'll be a prefix of what the `*` wildcard matched:

```bash
$ concurrently -n fix: 'npm:lint:fix:*'
# is equivalent to
$ concurrently -n fix:js,fix:ts 'npm run lint:fix:js' 'npm run lint:fix:ts'
```

Filtering out commands matched by wildcard is also possible. Do this with by including `(!<some pattern>)` in the command line:

```bash
$ concurrently 'yarn:lint:*(!fix)'
# is equivalent to
$ concurrently -n js,ts 'yarn run lint:js' 'yarn run lint:ts'
```

> [!NOTE]
> If you use this syntax with double quotes (`"`), bash and other shells might fail
> parsing it. You'll need to escape the `!`, or use single quote (`'`) instead.<br/>
> See [here](https://serverfault.com/a/208266/160539) for more information.

## File Wildcards

Wildcards aren't limited to npm/yarn/pnpm/bun/node/deno script names. In a plain command, an unquoted `*` token in a file path expands to one command per matching file in that directory, using the same `(!<some pattern>)` omission syntax described above. The name of each spawned command is set to whatever the `*` wildcard matched, just like with script wildcards.

```bash
$ concurrently 'webpack --config webpack.config.*(!base).js'
# spawns one webpack per matching config file, e.g. given
# webpack.config.base.js, webpack.config.dev.js and webpack.config.prod.js:
$ concurrently -n dev,prod 'webpack --config webpack.config.dev.js' 'webpack --config webpack.config.prod.js'
```

> [!NOTE]
> A wildcard that's inside quotes (for example `find . -name "*.js"`) is passed through to the shell untouched, since concurrently only expands wildcards it can see in the unquoted parts of the command line.
>
> If a file wildcard matches zero files, the command is left as written instead of being dropped.
