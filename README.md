[![hacs_badge](https://img.shields.io/badge/HACS-Custom-41BDF5.svg?style=for-the-badge)](https://github.com/hacs/integration)

# Toggle Card With TypeScript

Migration to TypeScript

***

* @status: alpha
* @published: June 2023
* @author: Elmar Hinz
* @name: `toggle-card-typescript`
* @id: `tcts`

You learn:

## Goal

Migrating the sources from *JavaScript* to *TypeScript*, installing the tools
and adjusting the setup of the project as required.


## Prerequisites

* previous tutorial to build upon (tutorial 09)
* adding entities, cards and resources (tutorial 04, 02)
* setting up the core developers container (tutorial 01)
* setting up an npm based project (tutorial 08)
* basic experiences with *Lit* (tutorial 09)

Find all sources inside the `src/` folder!

## About TypeScript

## Usage

This tutorial is done in form of a protocol of the migration to
*TypeScript*. I mainly write in the first person therefore. You have several
choices of how to follow along. Refer to the [chapter about possible
usages](https://github.com/home-assistant-tutorials/08.toggle-card-with-toolchain#usage)
in *tutorial 08* which did take a similar approach.

## Preparing the project

I create the project folder, copy `hacs.json`, `.gitignore`, `package.json` from
*tutorial 09*. I need to update the `name` property in `hacs.json`. Next I copy
the `src/` folder. All types of identifiers like class names etc. are renamed
to match the current tutorial.

I run `npm install` to install the libraries. I run `npm run watch`, create the
toggle `tcts`, add the new resource
`/local/tutor/10.toggle-card-typescript/dist/card.js` and add the card to the
dashboard.

![editor](img/editor.png)

## First steps

I start by changing the filename extensions from `.js` to `.ts` within the
`src/` folder.

![renaming the files](img/ts.png)

*VScode* is counting errors for some files. Without any setup *TypeScript* has
already been recognized and used by the editor.

### `package.json`

The entry point needs to be updated from `.js` to `.ts`.

```json
  "source": "src/index.ts",
```

Despite the "errors" shown by the editor the build process keeps working.
*JavaScript* was fine already and the *TypeScript* additions get just stripped
during building. The bundler *Parcel* does know what to do without additional
setup so far.

### `index.ts`

`index.ts` complains about a property `customCards` that does not exist. The
global window object represents the browser's window. By convention *Home
Assistant* adds an array named `customCards` to register all custom cards.
This array is not declared anywhere.

![window error](img/window-error.png)

Declaring types of objects, functions and data is what *TypeScript* is all
about. *VScode* was able to magically find a lot of such declarations, but this
one is specific to our file.

With the help of Stackoverflow I come to this solution and add it into the
head of the file.

```ts
declare global {
  interface Window {
    customCards: Array<Object>;
  }
}
```

The interface of the Window in the `global` scope gets "extended" by the new
property. `Array<Object>` even describes the type to a certain depth.

### `cards.ts`

Again error `ts(2339)`. The time *Vscode* can't find the types of the
*reactive properties*. This is tricky again. It is [related to *Lit*](https://lit.dev/docs/components/properties/#avoiding-issues-with-class-fields). This
properties are not directly declared, but dynamically created by *Lit* based
on the return value of `static get properties()`.

![lit error](img/lit-error.png)

For now I fix it by adding declarations to the class.

```ts
export class ToggleCardTypeScript extends LitElement {
  declare _header;
  declare _entity;
  declare _name;
  declare _state;
  declare _status;
  [ ... ]
```

I don't bother with types. The plan is to replace this approach with `@property`
decorators of *Lit*.

### `editor.ts`

The editor file gets fixed the same way.

```ts
export class ToggleCardTypeScriptEditor extends LitElement {
  declare _config;
  [ ... ]
```

### First steps done

We have been able to convert the files from *JavaScript* to *TypeScript* and the
toolchain keeps working after the redirection of the entry point.

As *TypeScript* is an extension of *JavaScript* each valid `.js` file should
also work as `.ts` file. We encountered some special cases where updating the
file extension was not enough to satisfy the *VScode* editor, though.

## Visiting the documentation

I rely upon the features of *VScode* to use *TypeScript* and upon *Parcel* to
transpile the files. For *VScode* *TypeScript* is a core feature. It does work
out of the box. Also *Parcel* does work out of the box. It follows a zero
configuration philosophy. However configuration still can be added and at a
certain point we always need to add configuration.

I visit the [*TypeScript* page of
*Parcel*](https://parceljs.org/languages/typescript/) to learn more about it.

Some takeaways:

To configure the transpilation `tsconfig.json` is used. This is the standard
configuration file for *TypeScript*. Compared to the *TSC* compiler from
Microsoft only a few options are supported. Other options could
be added to assist the *VSCode* editor only.

Parcel natively transpiles *TypeScript*. It can be configured to use the
official *TSC (TypeScript Compiler)* or to use *Babel*. This would be done in
`.parcelrc`.

The minimal configuration of `tsconfig.json` should inform the editor, that
isolated modules are used by *Parcel*. So cross-file features are to be avoided.

```json
{
  "compilerOptions": {
    "isolatedModules": true,
  }
}
```

I will also be interested in the options `experimentalDecorators` and
`useDefineForClassFields` to support decorators in *Lit*.

### `tsconfig.json`

In the moment I create `tsconfig.json`, even if it is empty, *VSCode* shows a
new error.

![assign error](img/assign-error.png)

I didn't bother with versions so far. Obviously the mere existence of this file
changes the assumptions of *VSCode* about it. We will have to cover two
questions later on. What versions of browsers do we target? What version of
EcmaScript do we want to use.

For now I want to get rid of the errors in the editor an compare some other
cards of *Home Assistant*. Two more settings get it done.

```json
{
  "compilerOptions": {
    "isolatedModules": true,
    "target": "es2017",
    "moduleResolution": "node",
  }
}
```

What do they do? Reading the references:

* [lib](https://www.typescriptlang.org/tsconfig#lib)
* [target](https://www.typescriptlang.org/tsconfig#target)
* [moduleResolution](https://www.typescriptlang.org/tsconfig#moduleResolution)

As for `target`, this quote looks relevant to the error in the recent
screenshot:

> Changing target also changes the default value of lib. You may “mix and match”
> target and lib settings as desired, but you could just set target for
> convenience.

So the standard way is to define a `target` version, which then deals with
`lib`.

Though the editor does not build for a target itself and *Parcel* takes its
browser versions from the settings in `package.json`. So for now I understand
this `target` as the version of *EcmaScript* I want to work with. Then I specify
in `package.json` what browsers I want to support.

### Adding decorators

Decorators are a stage 3 proposal for addition to the ECMAScript standard. Still
[the browsers don't support](https://caniuse.com/decorators) it. We need the
support of the Transpiler to get it working for older browsers.

A reason to use decorators is that I don't need extra declarations in
*TypeScript* any more. Also it's a move forward to an even more declarative
style to code *Lit* elements.

Previous

```js
export class ToggleCardTypeScriptEditor extends LitElement {
  declare _config;

  static get properties() {
    return {
      _config: { state: true },
    };
  }

  [ ... ]
```

becomes

```js
import { property } from "lit/decorators/property";

export class ToggleCardTypeScriptEditor extends LitElement {
  @property({ state: true })
  _config;

  [ ... ]
```

When the code is parsed the `@property` decorator is detected and a matching
function is called. This function does the same as the previous code.

More about decorators:

* [Decorators in TypeScript](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-0.html)
* [Decorators in Lit](https://lit.dev/docs/components/decorators/)

To be able to use decorators I add two lines to the `compileOptions` in
`tsconfig.json`.

```json
    "experimentalDecorators": true,
    "useDefineForClassFields": false,
```

The first setting enables this advanced feature. As for the second one [I quote
Lit](https://lit.dev/docs/components/decorators/#decorators-typescript):

> You should also ensure that the useDefineForClassFields setting is false.
Note, this should only be required when the target is set to esnext or greater,
but it's recommended to explicitly ensure this setting is false.

I try. Setting it to true breaks the reactive properties. Like with the missing
declarations above, this is related to the interactions of class fields and
reactive properties.
