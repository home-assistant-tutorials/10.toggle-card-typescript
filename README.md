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

I start by exchanging the file endings from `.js` to `.ts` within the `src/`
folder.

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
*reactive properties*. This is tricky again. It is related to *Lit*. This
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
annotations of *Lit*.

### `editor.ts`

The file gets fixed the same way.

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


