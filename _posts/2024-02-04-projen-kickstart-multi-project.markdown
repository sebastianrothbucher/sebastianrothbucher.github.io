---
layout: post
title: "Using projen to kickstart well-architected projects"
date:   2024-02-04 18:00:00 +0100
categories: architecture cloud projen templates
---

Starting from scratch is educational - but not efficient. Copying and pasting a great template is superb already - until you have to update every project. Tools like [*projen*](https://projen.io) (newest kid of many in the block) solve the issue by being able to start from a template - and *updating* from it as well. That allows more experienced devs / hands-on architects (is there any other kind?) to help projects get off the ground quickly. Especially when there's many similar projects. This post explores some specifics and details of how do do it.

Taking heavy lifting out of projects is not really new (see parent POM, archetypes, rails scaffolding, yeoman, many more) - but nonetheless powerful. It gets even more important in the cloud where an individual project can be quite small (like one lambda). 

The core idea is pretty simple: instead of creating `package.json` and other files oneself, `.projenrc.ts` is creating the file and applies whatever we give in as typescript (or python or ...). As all is just plain typescript, it's simple to have logic, spawn several subprojects (like for each lambda) - and to re-use code and defaults. 

## Basics

```
const project = new awscdk.AwsCdkTypeScriptApp({
  cdkVersion: '2.1.0',
  defaultReleaseBranch: 'main',
  name: 'projentest',
  projenrcTs: true,
  github: false,

  // deps: [],                /* Runtime dependencies of this module. */
  // description: undefined,  /* The description is just a string that helps people understand the purpose of the package. */
  // devDeps: [],             /* Build dependencies for this module. */
  // packageName: undefined,  /* The "name" in package.json. */
});
project.synth();
```

The above is all it really takes to create a CDK project (see further down for a node-based Lambda). Based on this, `npx projen` creates a whole bunch of files (incl. gitignore) with sensible defaults. (Caveat: `npm i -g yarn` needs to be run before the first `npx projen`) Via options and via `project.add<sth>`, the generation process can also be customized. 

There is also no need to limit oneself to defaults - here is how to create an own class with defaults: 

```
export class MyLambdaProject extends typescript.TypeScriptAppProject {
  super(options);
  this.addDevDeps('@aws-sdk/client-dynamodb@^3.500.0', '@aws-sdk/client-s3');
  // (more, see also below)
```

It can then be used as follows: 

```
const lambda1 = new MyLambdaProject({
  entrypoint: 'src/index.ts',
  defaultReleaseBranch: 'main',
  parent: project,
  outdir: 'functions/lambda1',
  name: 'lambda1',
  // (more if we want)
});
lambda1.synth();
```

So it's really just plain typescript. Plain typescript could be applied to e.g. loop over needed Lambdas.

Above example also shows *subprojects* - i.e. in my case, I have CDK as parent project and lambdas as children - in the same (mono)repo. Point is: all lambdas have the same defaults (in this case: dependencies added on top of what we configure). 

By the way: the `<dependency>@<version>` notation is optional, but it allows fixing versions if we *really* want it. 

If monorepo is not the way to go, there's other ways to distribute the `MyLambdaProject` class, e.g. a private NPM repo or a postinstall action cloning it to the local code or even a git submodule (though, to me, as a pragmatist, the latter seems overkill).

## Sample VS managed files

projen generates files upon `npx projen`. That's the standard files like `package.json` and more - but it can also be any custom files. There's two types: 

- `SampleFile` is only generated when it does not yet exist - can can be overwritten by devs afterwards. Useful e.g. for an empty lambda code file
- `TextFile` and other *managed files* (as projen calls it) are overwritten every time. projen is (btw) smart enough to create those write-protected so VSCode and others make it clear that editing is not a good idea. 

Here's a sample of both: 

```
export class MyLambdaProject extends typescript.TypeScriptAppProject { // could be a git module!
  constructor(options: typescript.TypeScriptProjectOptions) {
    super(options);
    // (more stuff)
    // add one static asset
    new SampleFile(this, 'sample.txt', {
      contents: 'Schmu123', // stays
    });
    // and one that updates
    new TextFile(this, 'managed.txt', {
      lines: 'Schmu123'.split(/[\r\n]+/), // gets overwritten
    });
  }
}
```

## projen VS CodeCatalyst blueprints

Disclaimer upfront: I'm still learning the latter as well. From all I see, the ideas are very similar indeed - add sth from a template and be able to *update* from the template (CodeCatalyst calls this *lifecycle management*). Sure the people involved were talking ;-) So, much can be achieved with both. When using blueprints, though, one is tying the knot with CodeCatalyst. Individual decision whether that makes sense.

## Wrap-up

projen is a quite useful new take on a well-established idea. It's hands-on and simple - and just allows to have many similar projects at almost zero effort for individual teams. There was not much point in monolith codebases - but with the cloud becoming the platform *and framework* more and more, this is getting really handy.

As ever: hope you found this useful - let me know what you think!
