# Reproducible use-case for `@nx/js` issue due to reliance on `ts-node` 10.9.1

See [this filed issue](https://github.com/nrwl/nx/issues/28492) for details.

Step to reproduce:

```bash
git clone https://github.com/tdd/nx-js-issue-demo.git
cd nx-js-issue-demo
npm install
npx nx release --first-release --dry-run
```

Output:

```
Convert compiler options from json failed, File '@nx-demo/tsconfig/exported-config.json' not found.

 NX   There was an error when resolving the configured changelog renderer at path: node_modules/nx/release/changelog-renderer

The relevant config is defined here: nx.json, lines 5-10


 NX   Cannot read properties of undefined (reading 'map')

Pass --verbose to see the stacktrace.


NOTE: The "dryRun" flag means no changes were made.
```

This is because when following the TS config extension chain (from the matching `tsconfig.json` file to its `extends` package and so on), `@nx/js` oddly relies on `ts-node` to parse these, and 10.9.1 incorrectly ignores *import conditions*, so when one of the modules providing a config file exposes it through its `exports` package field, and its actual path differs from the exported path, resolution fails.

(Note that the output above is a bit nicer than what I get in my original project, where I just get the "changelog renderer" line, not the line above, which is super-confusing.)

This can be fixed trivially by updating `@nx/js` dependency on `ts-node` to 10.9.2, which is the latest version (Dec 8, 10 months ago!) and does honor import conditions.

Longer-term, depending on ts-node when `@swc/core` / `@swc-node/register` are installed is weird, as `ts-node` is more or less abandonware these days.
