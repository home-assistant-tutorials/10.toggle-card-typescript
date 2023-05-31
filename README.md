[![hacs_badge](https://img.shields.io/badge/HACS-Custom-41BDF5.svg?style=for-the-badge)](https://github.com/hacs/integration)

# Toggle Card With TypeScript

Migration to TypeScript

***

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

Refer to the [chapter about possible
usages](https://github.com/home-assistant-tutorials/08.toggle-card-with-toolchain#usage)
in tutorial 08.

## Preparing the project

I create the project folder, copy `hacs.json`, `.gitignore`, `package.json` from
*tutorial 09*. I need to update the `name` property in `hacs.json`. Next I copy
the `src/` folder. All types of identifiers like class names etc. are renamed
to match the current tutorial.

Running `npm install` to install the libraries. Running `npm run watch`,
creating the toggle `tcts`, adding the new resource
`/local/tutor/10.toggle-card-typescript/dist/card.js` and adding the card to
the dashboard.

![editor](img/editor.png)

## First steps

I start by exchanging the file endings from `.js` to `.ts` within the `src/`
folder.

![renaming the files](img/ts.png)

Some files count errors. Without any setup *TypeScript* is already at work.

### `index.ts`

`index.ts` complains about a property `customCards` that does not exist. The
global window object represents the browser's window. By convention *Home
Assistant* adds an array named `customCards` to register all custom cards.
This array is not declared anywhere.

![window error](img/window-error.png)

Declaring types of objects, functions and data is what *TypeScript* is all
about. *VScode* was able to magically find an load a lot of such declarations,
but this one is specific to our file.

With the help of stackoverflow I come to this solution and add it into the
head of the file.

```ts
declare global {
  interface Window {
    customCards: Array<Object>;
  }
}
```

The interface ot the Window in the `global` scope gets "extended" by the new
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

### `package.json`

The entry point needs to be updated from `.js` to `.ts`.

```json
  "source": "src/index.ts",
```

### First steps done

We have been able to convert the files from *JavaScript* to *TypeScript* and the
toolchain is still able to bundle the result. As *TypeScript* is an extension of
*JavaScript* each `.js` file should also work as `.ts` file.

But we encountered some special cases where a simple conversion was not enough
to satisfy the *VScode* editor. The bundler would keep working without the
additional declarations as the *JavaScript* was fine already and the *TypeScript*
additions get stripped anyway.

