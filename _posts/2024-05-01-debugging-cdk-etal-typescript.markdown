

- debug cfg (as long as no spawn)
- debugging without sourcemap (formatting)
- debugging with sourcemap (TBD!)


- clone
- make sure python3, dotnet, docker are available (Docker Desktop on Mac does it)
- `yarn install --frozen-lockfile`
- `NODE_OPTIONS="--max-old-space-size=4096" npx lerna run build --concurrency 1` (several times when OoM; this skips tests we don't need; don't overwhelm)
- adjust *stack-activity-monitor.ts* L161 with *as any*

Problem I: Sourcemaps are not reliably pulled, one can not set breakpoints there ahead of time

Problem II: imports from node_modules need the **JS** version of things

Advantage I: when symlinking single files, relative resolution is to the *original* dir (todo: proof, besides taht it works)

Advantage II: when we have JS and TS, import will pick **TS**

Synthesis: symlink TS next to JS as far down the chain as we need (works in general, not just for CDK)

```javascript
        {
            "type": "node",
            "request": "launch",
            "env": {
                "TS_NODE_PREFER_TS_EXTS": "true",
            },
            "runtimeArgs": [
                "-r",
                "./node_modules/ts-node/register"
            ],
            "name": "CDK synth",
            "program": "${workspaceFolder}/cdk/node_modules/aws-cdk/bin/cdk.ts", // .js; pre-task does not cut it - this at least stops w/in our own source
            "args": ["synth"],
            "cwd": "${workspaceFolder}/cdk", // important
        },
```
= best way to transpile transparently
