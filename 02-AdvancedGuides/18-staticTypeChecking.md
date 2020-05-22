##### 5/20/2020
# Advanced Guides - Static Type Checking
Static type checkers like [`Flow`](https://flow.org/) and `TypeScript` identify certain types of problems before you even run your code.  They can also improve developer workflow by adding features like auto-completion. For this reason, we recommend using `Flow` or `TS` instead of `PropTypes` for larger code bases.

---

## `Flow`:
`Flow` is a static type checker for your `JS` coe.  It is developed at Facebook and is often used with `React`.  It lets you annotate the variables, `functions`, and `React` components with a special type syntax, and catch mistakes early.

To use `Flow`, you need to:
  * Add `Flow` to your project as a dependency
  * Ensure that `Flow` syntax is stripped from the compiled code
  * Add type annotations and run `Flow` to check them

## Adding `Flow` to a Project:
First, navigate to your project directory in the terminal.  You will need to run the following command:

If you use `Yarn`, run:

```
yar add --dev flow-bin
```

If you use `npm`, run:

```
npm install --save-dev flow-bin
```

This command installs the latest version of `Flow` into your project.

Now, add `flow` to the `"scripts"` section of your `package.json` to be able to use this from the terminal:

```json
{
  "scripts": {
    "flows": "flow"
  }
}
```

Finally, run one of the following commands:

`Yarn`:

```
yarn run flow init
```

`npm`:

```
npm run flow init
```

This command will create a `Flow` configuration file that you will need to commit.

## Stripping `Flow` Syntax from the Compiled Code:
`Flow` extends the `JS` language with a special syntax for type annotations. However, browsers aren't aware of this syntax, so we need to make sure it doesn't end up in the compiled `JS` bundle that is sent to the browser.

The exact way to do this depends on the tools you use to compile `JS`.

### Create React App:
If your project was set up using Create React App, congratulations!  The `Flow` annotations are already being stripped by default so you don't need to do anything else in this step.

### `Babel`:
  > **NOTE**: These instructions are _not_ for Create React App users.  Even though Create React App uses `Babel` under the hood, it is already configured to understand `Flow`.  Only follow this step if you _don't_ use Create React App.

If you manually configured `Babel` for you project, you will need to install a special preset for `Flow`.

`Yarn`:

```
yarn add --dev @babel/preset-flow
```

`npm`:

```
npm install --save-dev @babel/preset-flow
```

Then add the `flow` preset to your `Babel` config.

  > **NOTE**: `Flow` does not require the `react` preset, but they are often used together. `Flow` itself understands `JSX` syntax out of the box.

### Other Build Setups:
If you don't use either Create React App or `Babel`, you can use `flow-remove-types` to strip the type annotations.

## Running `Flow`:
If you followed the instructions above, you should be able to run `Flow` for the first time.

```
yarn flow
```

```
npm run flow
```

You should see a message like:

```
No errors!
✨  Done in 0.17s.
```

## Adding `Flow` Type Annotations:
By defalt, `Flow` only checks the files that include this annotations:

```
// @flow
```

Typically it is placed at the top of a file.  Try adding it to some files in your project and run `yarn flow` or `npm run flow` to see if `Flow` already found any issues.

There is also [an option](https://flow.org/en/docs/config/options/#toc-all-boolean) to force `Flow` to check _all_ files regardless of the annotation.  This can be too noisy for existing projects, but is reasonable for a new project if you want to fully type it with `Flow`.

---

## `TypeScript`:
`TS` is a programming language developed by Microsoft.  it is a typed superset of `JS`, and includes its own compiler.  being a typed language, `TS` can catch errors and bugs at build time, long before you app goes lives.  

  > You can learn more about using `TS` with `React` [HERE](https://github.com/Microsoft/TypeScript-React-Starter#typescript-react-starter).

To use `TS`, you need to:
  * Add `TS` as a dependency to your project
  * Configure the `TS` compiler options
  * Use the right file extensions
  * Add definitions for libraries you use

## Using `TS` with `Create React App`:
Create React App supports `TS` out of the box.

To create a **new project** with `TS` support, run:

```
npx create-react-app my-app --template typescript
```

You can also add it to an **existing Create React App project**, as documented [HERE](https://create-react-app.dev/docs/adding-typescript/).

## Adding `TS` to a Project:
It all begins with running one command in your terminal.

`Yarn`:

```
yarn add --dev typescript
```

`npm`:

```
npm install --save-dev typescript
```

Congrats!  You've installed the latest version of `TS` into your project. Installing `TS` gives us access to the `tsc` command.  Before configuration, let's add `tsc` to the `"scripts"` section in your `package.json`:

```json
{
  "scripts": {
    "build": "tsc"
  }
}
```

## Configuring the `TS` Compiler: 
The compiler is of on help to use until we tell it what to do.  In `TS`, these rules are defined in a special file called `tsconfig.json`.  To generate this file:

`Yarn`:

```
yarn run tsc --init
```

`npm`:

```
npx tsc --init
```

Looking at the now generated `tsconfig.json`, you can see that there are many options you can use to configure the compiler. For a detailed description of all the options, check [HERE](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html).

Of the many options, we'll look at `rootDir` and `outDir`.  In is true fashion, the compiler will take in `TS` files and generate `JS` files. However we don't want to get confused with our source files and the generated output.

We'll address this in two steps:
  1. Let's arrange our project structure like this.  We'll place all our source code in the `src` directory:

  ```
  |__package.json
  |__src
  |    |__index.ts
  |__tsconfig.json
  ```
  2. We'll tell the compiler where our source code is and where the output should go:

  ```json
  // tsconfig.json
  {
    "compilerOptions": {
      "rootDir": "src",
      "outDir": "build"
    }
  }
  ```

Great!  Now when we run our build script the compiler will output the generated `JS` to the `build` folder.  The [TypeScript React Starter](https://github.com/Microsoft/TypeScript-React-Starter/blob/master/tsconfig.json) provides a `tsconfig.json` with a good set of rules to get you started.

Generally, you don't want to keep the generated `JS` in your source control, so be sure to add the build folder to your `.gitignore`.

## File Extensions:
In `React`, you most likely write your components in a `.js` file.  In `TS`, we have 2 file extensions:

`.ts` is the default file extension while `.tsx` is a special extension used for files which contain `JSX`.

## Running `TS`:
If you followed the instructions above, you should be able to run `TS` for the first time.

`Yarn`:

```
yarn build
```

`npm`:

```
npm run build
```

If you see no output, it means that it completed successfully.

## Type Definitions:
To be able to show errors and hints from other packages, the compiler relies on declaration files.  A declaration files provides all the type information about a library. This enables us to use `JS` libraries like those on `npm` in our project.

There are two main ways to get declarations for a library:

**Bundled**: The library bundles its own declaration file. This is great for us, since all we need to do is install the library, and we can use it right away.  To check if a library has bundled types, look for an `index.d.ts` file in the project.  Some libraries will have it specified in their `package.json` under the `typings` or `types` field.

**DefinitelyTyped**: `DefinitelyTyped` is a huge repository of declarations for libraries that don't bundle a declaration file.  The declarations are crowd-sourced and managed by Microsoft and open source contributors.  `React` for example doesn't bundle its own declaration file.  Instead we can get it from `DefinitelyTyped`.  To do so enter this command in your terminal.

```
# yarn
yarn add --dev @types/react

#npm
npm i --save-dev @types/react
```

**Local Declarations**: Sometimes the package that you want to use doesn't bundle declarations nor is it available on `DefinitelyTyped`. In that case, we can have a local declaration file.  To do this, create a `declarations.d.ts` file in the root of your source directory.  A simple declaration could look like this:

```ts
declare module 'querystring' {
  export function stringify(val: string) : string
  export function parse(val: string) : object
}
```

You are now ready to code!

---

[React Docs](https://reactjs.org/docs/static-type-checking.html)