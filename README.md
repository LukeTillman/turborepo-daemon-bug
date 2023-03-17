Repository for reproducing: https://github.com/vercel/turbo/issues/4137

To reproduce:

1. Clone the repository
2. Install dependencies: `npm install`
3. Make sure Turborepo is installed globally: `npm install turbo -g`
4. Run the build command for the library repeatedly (see output below that indicates you've triggered the bug): `turbo run build --filter=lib-a`

This bug seems like it's possibly timing-related, and so you won't see it happen every time you run the build script. When it runs successfully without the bug happening, you'll see something like this:

```
$ turbo run build --filter=lib-a
• Packages in scope: lib-a
• Running build in 1 packages
• Remote caching disabled
lib-a:clean: cache bypass, force executing dc59c72252acb1de
lib-a:clean:
lib-a:clean: > lib-a@0.1.0 clean
lib-a:clean: > rimraf dist/
lib-a:clean:
lib-a:build:esm: cache hit, replaying output 3924692684ffd04c
lib-a:build:types: cache hit, replaying output 8f1d91e12889474b
lib-a:build:esm:
lib-a:build:esm: > lib-a@0.1.0 build:esm
lib-a:build:esm: > swc ./src -d ./dist --config-file ../.swcrc
lib-a:build:esm:
lib-a:build:esm: Successfully compiled: 1 file with swc (6.36ms)
lib-a:build:types:
lib-a:build:types: > lib-a@0.1.0 build:types
lib-a:build:types: > tsc
lib-a:build:types:

 Tasks:    3 successful, 3 total
Cached:    2 cached, 3 total
  Time:    520ms

$ ls lib-a/dist
index.d.ts     index.d.ts.map index.js       index.js.map
```

And the `dist` folder in `lib-a` will have the build output. When the bug happens, you'll see output like this instead:

```
$ turbo run build --filter=lib-a
• Packages in scope: lib-a
• Running build in 1 packages
• Remote caching disabled
lib-a:clean: cache bypass, force executing dc59c72252acb1de
lib-a:clean:
lib-a:clean: > lib-a@0.1.0 clean
lib-a:clean: > rimraf dist/
lib-a:clean:
lib-a:build:types: Skipping cache check for lib-a#build:types, outputs have not changed since previous run.
lib-a:build:types: cache hit, replaying output 8f1d91e12889474b
lib-a:build:esm: Skipping cache check for lib-a#build:esm, outputs have not changed since previous run.
lib-a:build:esm: cache hit, replaying output 3924692684ffd04c
lib-a:build:types:
lib-a:build:types: > lib-a@0.1.0 build:types
lib-a:build:types: > tsc
lib-a:build:types:
lib-a:build:esm:
lib-a:build:esm: > lib-a@0.1.0 build:esm
lib-a:build:esm: > swc ./src -d ./dist --config-file ../.swcrc
lib-a:build:esm:
lib-a:build:esm: Successfully compiled: 1 file with swc (6.36ms)

 Tasks:    3 successful, 3 total
Cached:    2 cached, 3 total
  Time:    320ms

$ ls lib-a/dist
ls: lib-a/dist: No such file or directory
```

The "skipping cache check" message is the one that indicates you've likely triggered the bug. If you check the `dist` folder after seeing that message, you'll see it's missing/doesn't exist.
