# Explanation

## What Problem Are We Solving?

The `typescript-tools` enforce a set of invariants within a JavaScript/TypeScript
monorepo, and provide a way to query related properties of the monorepo. These
invariants keep the monorepo in a _consistent_ state, so other tools like [lerna],
[make], and [tsc], will "just work" with minimum surprises.

[lerna]: https://github.com/lerna/lerna
[make]: https://www.gnu.org/software/make
[tsc]: https://www.typescriptlang.org/docs/handbook/compiler-options.html

## Invariants

### Accurate TypeScript Project References

> command: `link`

Whereas [lerna] was created for managing JavaScript monorepos, TypeScript monorepos
have additional requirements introduced by the compilation step. Consider the
following example monorepo with 3 TypeScript packages:

![Monorepo with `dependency A` and `dependency B` consumed by `consumer C`](res/example-link.png)

We'd like to offer a simple[^1] developer experience for building any package
in our monorepo: invoke `tsc` and the target package will compile. Unfortunately,
this doesn't work out of the box. If we try to compile `consumer C`, the TypeScript
compiler will error because `dependency A` and `dependency B` haven't been built yet.
Thankfully we can leverage TypeScript [project references] to declare `consumer C`'s
dependencies on other packages in the monorepo. With project references configured,
`tsc` will build the target package and all referenced dependencies in topological
order.

The `typescript-tools` offer a `link` command to update the TypeScript project
references for all packages, using each `package.json` as the source of truth for
identifying internal dependencies. We can hook this command into the monorepo's
build workflow such that `link` is invoked before `tsc`, turning an imperative
compilation workflow into a declarative one. That is, instead of invoking `tsc` on
all dependencies and then the target package, we can tell `tsc` "build the target
package, and do whatever it takes to make that possible".

[project references]: https://www.typescriptlang.org/docs/handbook/project-references.html

### Use Current Dependency Versions

> command: `pin`

To prevent against [dependency confusion attacks], we recommend using lerna with the
`--force-local` flag[^2]. The premise of the attack is tricking `npm` to download an
adversary's remote package instead of using the monorepo's locally defined package. The
`--force-local` flag causes

> the `bootstrap` (or `link`) command to always symlink local dependencies regardless of
> matching version range

(parenthetical text mine)

We can see from the description of the flag that lerna will not fall victim to any
dependency confusion attacks. But what about when we're not using lerna? For example,
after we've published the package to our registry and are installing it in
another project, or inside a docker build.

We need a solution that guarantees `npm` will always install the intended version of
our internal dependencies, even when lerna is not in play. The simplest approach here,
and the one minimizes head-scratching for developers not privvy to the `--force-local`
flag, is to require all depending packages refer to the same version number that
the dependency declares.

![`pin` updating `my-app`'s package.json to depend on `my-library`'s declared version number](res/example-pin.png)

The `typescript-tools` offer a `pin` command to update the version number of internal
dependencies in every `package.json` file. We can hook this command into the monorepo's
bootstrap workflow such that `pin` is always invoked before dependencies are installed,
minimizing the number of developer surprises by making the relationship between
child and parent packages explicit.

[dependency confusion attacks]: https://medium.com/@alex.birsan/dependency-confusion-4a5d60fec610

[^1]: https://www.infoq.com/presentations/Simple-Made-Easy/
[^2]: https://github.com/lerna/lerna/tree/main/commands/bootstrap#--force-local
