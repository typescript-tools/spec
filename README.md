# TypeScript Tools Specification

This repository describes the target specification that each implementation of the
`typescript-tools` must implement.

Implementations are free to provide additional functionality, but should implement
this base specification to provide a reasonable expectation that swapping one
implementation for another will not produce changes in behavior.

Before defining the specification, it may be helpful to level set with a
[high-level overview of the project].

[high-level overview of the project]: ./explanation.md

## Specification 1.0.0

### `pin`

When invoked, `pin` will make the necessary edits in the `dependencies`,
`devDependencies`, `optionalDependencies`, and `peerDependencies` sections of
the package.json file of each package defined in the Lerna manifest to uphold the
following invariant:

For each package `P` defined in the the lerna manifest, each package `Q` depending on
`P` references `P`'s declared version number.

For example, if `P`'s package.json is the following:

```json
{
  "name": "P",
  "version": "1.2.3"
}
```

then every package depending on `P` must depend on `P` at version `"1.2.3"` exactly
(without a semver range).

### `link`

When invoked, `link` will make the necessary edits in the `references` section of
the tsconfig.json file of each package defined in the Lerna manifest to uphold the
following invariant:

For each package `P` defined in the lerna manifest, each package `Q` depending on
`P` references `P` via shortest relative path in `Q`'s TypeScript project references.

References should be sorted alphabetically[^1] by relative path.

For example, if `O` resides in `packages/O`, `P` resides in `packages/P`,
and `Q` resides in `packages/Q`, and `Q` depends on `O` and `P`, then
`packages/Q/tsconfig.json` should contain

```json
{
  "references": [
    {
      "path": "../O"
    },
    {
      "path": "../P"
    }
  ]
}
```

## Recommended Workflow

### `pin`

Invoke `pin` before `lerna bootstrap`. For example, if you invoke `lerna bootstrap` with npm:

```json
{
  "scripts": {
    "prebootstrap": "monorepo pin",
    "bootstrap": "lerna bootstrap"
  }
}
```

It is also recommended to use `--force-local`[^2] and `--reject-cycles`[^3]. You can configure this in `lerna.json`:

```json
{
  "bootstrap": {
    "forceLocal": true,
    "rejectCycles": true
  },
  "link": {
    "forceLocal": true,
    "rejectCycles": true
  }
}
```

### `link`

Invoke `link` before invoking the TypeScript compiler, `tsc`:

```json
{
  "scripts": {
    "prebuild": "monorepo link",
    "build": "tsc --build --incremental --verbose ."
  }
}
```

[^1]: TODO: define "alphabetically"
[^2]: https://github.com/lerna/lerna/tree/main/commands/bootstrap#--force-local
[^3]: https://github.com/lerna/lerna/tree/main/core/global-options#--reject-cycles
