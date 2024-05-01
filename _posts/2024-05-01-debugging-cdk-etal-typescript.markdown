

- debug cfg (as long as no spawn)
- debugging without sourcemap (formatting)
- debugging with sourcemap (TBD!)


- clone
- make sure python3, dotnet, docker are available (Docker Desktop on Mac does it)
- `yarn install --frozen-lockfile`
- `NODE_OPTIONS="--max-old-space-size=4096" npx lerna run build --concurrency 1` (several times when OoM; this skips tests we don't need; don't overwhelm)
- adjust *stack-activity-monitor.ts* L161 with *as any*

Problem I: Sourcemaps are not reliably pulled, one can not set breakpoints there ahead of time

Problem II: imports from node_modules need the **JS** version of things (xcept `exports {".": "folder/index.ts"}`)

Advantage I: when symlinking single files, relative resolution is to the *original* dir (todo: proof, besides taht it works)

Advantage II: when we have JS and TS, import will pick **TS**

Synthesis: symlink TS next to JS as far down the chain as we need (works in general, not just for CDK)

```javascript
        {
            "type": "node",
            "request": "launch",
            "env": {
                "TS_NODE_PREFER_TS_EXTS": "true",
                "TS_NODE_COMPILER_OPTIONS": "{\"allowJs\": false, \"sourceMap\": false}", // enforce debugging TS with a baseball bat
            },
            "runtimeArgs": [
                "-r",
                "./node_modules/ts-node/register"
            ],
            "name": "CDK synth",
            "program": "${workspaceFolder}/cdk/node_modules/aws-cdk/bin/cdk.ts", // .js; pre-task does not cut it - this at least stops w/in our own source
            "args": ["synth"],
            "cwd": "${workspaceFolder}/cdk", // important
            "outFiles": [], // this, along with compiler options, is required to actually debug TS
        },
```
= best way to transpile transparently

when no TS but readable JS, prettify this or move away and symlink TS equivalent (e.g. aws-cdk-lib) + add/adjust `exports` in package.json

So, these are the strategies: 
- I: any how -r ts-node/register in debug cfg
- IIa: just prettify and hope for the best
- IIb: set env vars, outFiles, symlink aws-cdk-lib source, (pot. build before or carry `export` to deps as well), step through actual TS all the way
- (same strategies apply globally)

So: write it up nicely ;-) 
