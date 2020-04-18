## Using the CLI

Running `tsc` locally will compile the closest project defined by a `tsconfig.json`, you can compile a set of TypeScript
files by passing in a glob of files you want.

```sh
# Run a compile based on a backwards look through the fs for a tsconfig.json
tsc 

# Transpile just the index.ts with the compiler defaults
tsc index.ts

# Transpile any .ts files in the folder src, with the default settings
tsc src/*.ts

# Transpile any .ts files in the folder src, with the compiler settings from tsconfig.json
tsc --project tsconfig.json src/*.ts
```

## Compiler Options

If you're looking for more information about the compiler options in a tsconfig, check out the TSConfig Reference beta
available in [the v2 site](https://www.typescriptlang.org/v2/en/tsconfig).

Option                                         | Type      | Default                        | Description
-----------------------------------------------|-----------|--------------------------------|----------------------------------------------------------------------
`--allowJs`                                    | `boolean` | `false`                        | Allow JavaScript files to be compiled.
`--allowSyntheticDefaultImports`               | `boolean` | `module === "system"` or `--esModuleInterop` | Allow default imports from modules with no default export. This does not affect code emit, just typechecking.
`--allowUmdGlobalAccess`                       | `boolean` | `false`                        | Allow accessing UMD globals from modules.
`--allowUnreachableCode`                       | `boolean` | `false`                        | Do not report errors on unreachable code.
`--allowUnusedLabels`                          | `boolean` | `false`                        | Do not report errors on unused labels.
`--alwaysStrict`                               | `boolean` | `false`                        | Parse in strict mode and emit `"use strict"` for each source file
`--assumeChangesOnlyAffectDirectDependencies`  | `boolean` | `false`                        | Have recompiles in '--incremental' and '--watch' assume that changes within a file will only affect files directly depending on it
`--baseUrl`                                    | `string`  |                                | Base directory to resolve non-relative module names. See [Module Resolution documentation](./Module%20Resolution.md#base-url) for more details.
`--build`<br/>`-b`                             | `boolean` | `false`                        | Builds this project and all of its dependencies specified by [Project References](./Project%20References.md). Note that this flag is not compatible with others on this page. See more [here](./Project%20References.md)
`--charset`                                    | `string`  | `"utf8"`                       | The character set of the input files.
`--checkJs`                                    | `boolean` | `false`                        | Report errors in `.js` files. Use in conjunction with `--allowJs`.
`--composite`                                  | `boolean` | `true`                         | Ensure TypeScript can determine where to find the outputs of the referenced project to compile project.
`--declaration`<br/>`-d`                       | `boolean` | `false`                        | Generates corresponding `.d.ts` file.
`--declarationDir`                             | `string`  |                                | Output directory for generated declaration files.
`--declarationMap`                             | `boolean` | `false`                        | Generates a sourcemap for each corresponding '.d.ts' file.
`--diagnostics`                                | `boolean` | `false`                        | Show diagnostic information.
`--disableSizeLimit`                           | `boolean` | `false`                        | Disable size limitation on JavaScript project.
`--downlevelIteration`                         | `boolean` | `false`                        | Provide full support for iterables in `for..of`, spread and destructuring when targeting ES5 or ES3.
`--emitBOM`                                    | `boolean` | `false`                        | Emit a UTF-8 Byte Order Mark (BOM) in the beginning of output files.
`--emitDeclarationOnly`                        | `boolean` | `false`                        | Only emit '.d.ts' declaration files.
`--emitDecoratorMetadata`<sup>[1]</sup>        | `boolean` | `false`                        | Emit design-type metadata for decorated declarations in source. See [issue #2577](https://github.com/Microsoft/TypeScript/issues/2577) for details.
`--esModuleInterop`                            | `boolean` | `false`                        | Emit `__importStar` and `__importDefault` helpers for runtime babel ecosystem compatibility and enable `--allowSyntheticDefaultImports` for typesystem compatibility.
`--experimentalDecorators`<sup>[1]</sup>       | `boolean` | `false`                        | Enables experimental support for ES decorators.
`--extendedDiagnostics`                        | `boolean` | `false`                        | Show verbose diagnostic information
`--forceConsistentCasingInFileNames`           | `boolean` | `false`                        | Disallow inconsistently-cased references to the same file.
`--generateCpuProfile`                         | `string`  | `profile.cpuprofile`           | Generates a cpu profile at the given path. Passing an existing directory name instead of a file path will cause a timestamp-named profile to be generated in that directory instead.
`--help`<br/>`-h`                              |           |                                | Print help message.
`--importHelpers`                              | `boolean` | `false`                        | Import emit helpers (e.g. `__extends`, `__rest`, etc..) from [`tslib`](https://www.npmjs.com/package/tslib)
`--importsNotUsedAsValues`                     | `string`  | `remove`                       | Specify emit/checking behavior for imports that are only used for types. `"remove"` and `"preserve"` specify whether to emit unused imports for side effects, and `"error"` enforces that imports used only for types are written with `import type`.
`--incremental`                                | `boolean` | `true` if `composite` is on, `false` otherwise | Enable incremental compilation by reading/writing information from prior compilations to a file on disk. This file is controlled by the `--tsBuildInfoFile` flag.
`--inlineSourceMap`                            | `boolean` | `false`                        | Emit a single file with source maps instead of having a separate file.
`--inlineSources`                              | `boolean` | `false`                        | Emit the source alongside the sourcemaps within a single file; requires `--inlineSourceMap` or `--sourceMap` to be set.
`--init`                                       |           |                                | Initializes a TypeScript project and creates a `tsconfig.json` file.
`--isolatedModules`                            | `boolean` | `false`                        | Perform additional checks to ensure that separate compilation (such as with [`transpileModule`](https://github.com/Microsoft/TypeScript/wiki/Using-the-Compiler-API#a-simple-transform-function) or [@babel/plugin-transform-typescript](https://babeljs.io/docs/en/babel-plugin-transform-typescript)) would be safe.
`--jsx`                                        | `string`  | `"preserve"`                   | Support JSX in `.tsx` files: `"react"`, `"preserve"`, `"react-native"`. See [JSX](./JSX.md).
`--jsxFactory`                                 | `string`  | `"React.createElement"`        | Specify the JSX factory function to use when targeting react JSX emit, e.g. `React.createElement` or `h`.
`--keyofStringsOnly`                           | `boolean` | `false`                        | Resolve `keyof` to string valued property names only (no numbers or symbols).
`--useDefineForClassFields`                    | `boolean` | `false`                        | Emit class fields with ECMAScript-standard semantics.
`--lib`                                        | `string[]`|                                | List of library files to be included in the compilation.<br/>Possible values are:  <br/>► `ES5` <br/>► `ES6` <br/>► `ES2015` <br/>► `ES7` <br/>► `ES2016` <br/>► `ES2017`  <br/>► `ES2018` <br/>► `ESNext` <br/>► `DOM` <br/>► `DOM.Iterable` <br/>► `WebWorker` <br/>► `ScriptHost` <br/>► `ES2015.Core` <br/>► `ES2015.Collection` <br/>► `ES2015.Generator` <br/>► `ES2015.Iterable` <br/>► `ES2015.Promise` <br/>► `ES2015.Proxy` <br/>► `ES2015.Reflect` <br/>► `ES2015.Symbol` <br/>► `ES2015.Symbol.WellKnown` <br/>► `ES2016.Array.Include` <br/>► `ES2017.object` <br/>► `ES2017.Intl` <br/>► `ES2017.SharedMemory` <br/>► `ES2017.String` <br/>► `ES2017.TypedArrays` <br/>► `ES2018.Intl` <br/>► `ES2018.Promise` <br/>► `ES2018.RegExp` <br/>► `ESNext.AsyncIterable` <br/>► `ESNext.Array` <br/>► `ESNext.Intl` <br/>► `ESNext.Symbol` <br/><br/> Note: If `--lib` is not specified a default list of libraries are injected. The default libraries injected are:  <br/> ► For `--target ES5`: `DOM,ES5,ScriptHost`<br/>  ► For `--target ES6`: `DOM,ES6,DOM.Iterable,ScriptHost`
`--listEmittedFiles`                           | `boolean` | `false`                        | Print names of generated files part of the compilation.
`--listFiles`                                  | `boolean` | `false`                        | Print names of files part of the compilation.
`--locale`                                     | `string`  | *(platform specific)*          | The locale to use to show error messages, e.g. en-us. <br/>Possible values are:  <br/>► English (US): `en` <br/>► Czech: `cs` <br/>► German: `de` <br/>► Spanish: `es` <br/>► French: `fr` <br/>► Italian: `it` <br/>► Japanese: `ja` <br/>► Korean: `ko` <br/>► Polish: `pl` <br/>► Portuguese(Brazil): `pt-BR` <br/>► Russian: `ru` <br/>► Turkish: `tr` <br/>► Simplified Chinese: `zh-CN`  <br/>► Traditional Chinese: `zh-TW`
`--mapRoot`                                    | `string`  |                                | Specifies the location where debugger should locate map files instead of generated locations. Use this flag if the .map files will be located at run-time in a different location than the .js files. The location specified will be embedded in the sourceMap to direct the debugger where the map files will be located. This flag will not create the specified path and generate the map files in that location. Instead, create a post build step that moves the files to the specified path.
`--maxNodeModuleJsDepth`                       | `number`  | `0`                            | The maximum dependency depth to search under node_modules and load JavaScript files. Only applicable with `--allowJs`.
`--module`<br/>`-m`                            | `string`  | `target === "ES3" or "ES5" ? "CommonJS" : "ES6"`   | Specify module code generation: `"None"`, `"CommonJS"`, `"AMD"`, `"System"`, `"UMD"`, `"ES6"`, `"ES2015"` or `"ESNext"`.<br/>► Only `"AMD"` and `"System"` can be used in conjunction with `--outFile`.<br/>► `"ES6"` and `"ES2015"` values may be used when targeting `"ES5"` or lower.
`--moduleResolution`                           | `string`  | `module === "AMD" or "System" or "ES6" ?  "Classic" : "Node"`                    | Determine how modules get resolved. Either `"Node"` for Node.js/io.js style resolution, or `"Classic"`. See [Module Resolution documentation](./Module%20Resolution.md) for more details.
`--newLine`                                    | `string`  | *(platform specific)*          | Use the specified end of line sequence to be used when emitting files: `"crlf"` (windows) or `"lf"` (unix)."
`--noEmit`                                     | `boolean` | `false`                        | Do not emit outputs.
`--noEmitHelpers`                              | `boolean` | `false`                        | Do not generate custom helper functions like `__extends` in compiled output.
`--noEmitOnError`                              | `boolean` | `false`                        | Do not emit outputs if any errors were reported.
`--noErrorTruncation`                          | `boolean` | `false`                        | Do not truncate error messages.
`--noFallthroughCasesInSwitch`                 | `boolean` | `false`                        | Report errors for fallthrough cases in switch statement.
`--noImplicitAny`                              | `boolean` | `false`                        | Raise error on expressions and declarations with an implied `any` type.
`--noImplicitReturns`                          | `boolean` | `false`                        | Report an error when not all code paths in function return a value.
`--noImplicitThis`                             | `boolean` | `false`                        | Raise error on `this` expressions with an implied `any` type.
`--noImplicitUseStrict`                        | `boolean` | `false`                        | Do not emit `"use strict"` directives in module output.
`--noLib`                                      | `boolean` | `false`                        | Do not include the default library file (`lib.d.ts`).
`--noResolve`                                  | `boolean` | `false`                        | Do not add triple-slash references or module import targets to the list of compiled files.
`--noStrictGenericChecks`                      | `boolean` | `false`                        | Disable strict checking of generic signatures in function types.
`--noUnusedLocals`                             | `boolean` | `false`                        | Report errors on unused locals.
`--noUnusedParameters`                         | `boolean` | `false`                        | Report errors on unused parameters.
~~`--out`~~                                    | `string`  |                                | DEPRECATED. Use `--outFile` instead.
`--outDir`                                     | `string`  |                                | Redirect output structure to the directory.
`--outFile`                                    | `string`  |                                | Concatenate and emit output to single file. The order of concatenation is determined by the list of files passed to the compiler on the command line along with triple-slash references and imports. See [output file order documentation](https://github.com/Microsoft/TypeScript/wiki/FAQ#how-do-i-control-file-ordering-in-combined-output---out-) for more details.
`paths`<sup>[2]</sup>                          | `Object`  |                                | List of path mapping entries for module names to locations relative to the `baseUrl`. See [Module Resolution documentation](./Module%20Resolution.md#path-mapping) for more details.
`--preserveConstEnums`                         | `boolean` | `false`                        | Do not erase const enum declarations in generated code. See [const enums documentation](https://github.com/Microsoft/TypeScript/blob/master/doc/spec.md#94-constant-enum-declarations) for more details.
`--preserveSymlinks`                            | `boolean` | `false`                       | Do not resolve symlinks to their real path; treat a symlinked file like a real one.
`--preserveWatchOutput`                        | `boolean` | `false`                        | Keep outdated console output in watch mode instead of clearing the screen
`--pretty`                                     | `boolean` | `true` unless piping to another program or redirecting output to a file | Stylize errors and messages using color and context.
`--project`<br/>`-p`                           | `string`  |                                | Compile a project given a valid configuration file.<br/>The argument can be a file path to a valid JSON configuration file, or a directory path to a directory containing a `tsconfig.json` file.<br/>See [tsconfig.json](./tsconfig.json.md) documentation for more details.
`--reactNamespace`                             | `string`  | `"React"`                      | DEPRECATED. Use `--jsxFactory` instead.<br/>Specifies the object invoked for `createElement` and `__spread` when targeting `"react"` JSX emit.
`--removeComments`                             | `boolean` | `false`                        | Remove all comments except copy-right header comments beginning with `/*!`
`--resolveJsonModule`                          | `boolean` | `false`                        | Include modules imported with `.json` extension.
`--rootDir`                                    | `string`  | *(common root directory is computed from the list of input files)*   | Specifies the root directory of input files. Only use to control the output directory structure with `--outDir`.
`rootDirs`<sup>[2]</sup>                       | `string[]`|                                | List of <i>root</i> folders whose combined content represent the structure of the project at runtime. See [Module Resolution documentation](./Module%20Resolution.md#virtual-directories-with-rootdirs) for more details.
`--showConfig`                                 | `boolean` | `false`                        | Rather than actually execute a build with the other input options and config files, show the final implied config file in the output.
`--skipDefaultLibCheck`                        | `boolean` | `false`                        | DEPRECATED. Use `--skipLibCheck` instead.<br/>Skip type checking of [default library declaration files](./Triple-Slash%20Directives.md#-reference-no-default-libtrue).
`--skipLibCheck`                               | `boolean` | `false`                        | Skip type checking of all declaration files (`*.d.ts`).
`--sourceMap`                                  | `boolean` | `false`                        | Generates corresponding `.map` file.
`--sourceRoot`                                 | `string`  |                                | Specifies the location where debugger should locate TypeScript files instead of source locations. Use this flag if the sources will be located at run-time in a different location than that at design-time. The location specified will be embedded in the sourceMap to direct the debugger where the source files will be located.
`--strict`                                     | `boolean` | `false`                        | Enable all strict type checking options. <br/>Enabling `--strict` enables `--noImplicitAny`, `--noImplicitThis`, `--alwaysStrict`, `--strictBindCallApply`, `--strictNullChecks`, `--strictFunctionTypes` and `--strictPropertyInitialization`.
`--strictBindCallApply`                        | `boolean` | `false`                        | Enable stricter checking of the `bind`, `call`, and `apply` methods on functions.
`--strictFunctionTypes`                        | `boolean` | `false`                        | Disable bivariant parameter checking for function types.
`--strictPropertyInitialization`               | `boolean` | `false`                        | Ensure non-undefined class properties are initialized in the constructor. This option requires `--strictNullChecks` be enabled in order to take effect.
`--strictNullChecks`                           | `boolean` | `false`                        | In strict null checking mode, the `null` and `undefined` values are not in the domain of every type and are only assignable to themselves and `any` (the one exception being that `undefined` is also assignable to `void`).
`--suppressExcessPropertyErrors`               | `boolean` | `false`                        | Suppress excess property checks for object literals.
`--suppressImplicitAnyIndexErrors`             | `boolean` | `false`                        | Suppress `--noImplicitAny` errors for indexing objects lacking index signatures. See [issue #1232](https://github.com/Microsoft/TypeScript/issues/1232#issuecomment-64510362) for more details.
`--target`<br/>`-t`                            | `string`  | `"ES3"`                        | Specify ECMAScript target version: <br/>► `"ES3"` (default) <br/>► `"ES5"` <br/>► `"ES6"`/`"ES2015"` <br/>► `"ES2016"` <br/>► `"ES2017"` <br/>► `"ES2018"` <br/>► `"ES2019"` <br/>► `"ES2020"` <br/>► `"ESNext"`<br/><br/> Note: `"ESNext"` targets latest supported [ES proposed features](https://github.com/tc39/proposals).
`--traceResolution`                            | `boolean` | `false`                        | Report module resolution log messages.
`--tsBuildInfoFile`                            | `string`  | `.tsbuildinfo`                 | Specify what file to store incremental build information in.
`--types`                                      | `string[]`|                                | List of names of type definitions to include. See [@types, --typeRoots and --types](./tsconfig.json.md#types-typeroots-and-types) for more details.
`--typeRoots`                                  | `string[]`|                                | List of folders to include type definitions from. See [@types, --typeRoots and --types](./tsconfig.json.md#types-typeroots-and-types) for more details.
`--version`<br/>`-v`                           |           |                                | Print the compiler's version.
`--watch`<br/>`-w`                             |           |                                | Run the compiler in watch mode. Watch input files and trigger recompilation on changes. The implementation of watching files and directories can be configured using environment variable. See [configuring watch](./Configuring%20Watch.md) for more details.

* <sup>[1]</sup> These options are experimental.
* <sup>[2]</sup> These options are only allowed in `tsconfig.json`, and not through command-line switches.

## Related

* Setting compiler options in [`tsconfig.json`](./tsconfig.json.md) files.
* Setting compiler options in [MSBuild projects](./Compiler%20Options%20in%20MSBuild.md).
