## Overview

Compiler options can be specified using MSBuild properties within an MSBuild project.

## Example

```XML
  <PropertyGroup Condition="'$(Configuration)' == 'Debug'">
    <TypeScriptRemoveComments>false</TypeScriptRemoveComments>
    <TypeScriptSourceMap>true</TypeScriptSourceMap>
  </PropertyGroup>
  <PropertyGroup Condition="'$(Configuration)' == 'Release'">
    <TypeScriptRemoveComments>true</TypeScriptRemoveComments>
    <TypeScriptSourceMap>false</TypeScriptSourceMap>
  </PropertyGroup>
  <Import
      Project="$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v$(VisualStudioVersion)\TypeScript\Microsoft.TypeScript.targets"
      Condition="Exists('$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v$(VisualStudioVersion)\TypeScript\Microsoft.TypeScript.targets')" />
```

## Mappings

Compiler Option                              | MSBuild Property Name                      | Allowed Values
---------------------------------------------|--------------------------------------------|-----------------
`--allowJs`                                  | *Not supported in MSBuild*                 |
`--allowSyntheticDefaultImports`             | TypeScriptAllowSyntheticDefaultImports     | boolean
`--allowUnreachableCode`                     | TypeScriptAllowUnreachableCode             | boolean
`--allowUnusedLabels`                        | TypeScriptAllowUnusedLabels                | boolean
`--alwaysStrict`                             | TypeScriptAlwaysStrict                     | boolean
`--baseUrl`                                  | TypeScriptBaseUrl                          | File path
`--charset`                                  | TypeScriptCharset                          |
`--declaration`                              | TypeScriptGeneratesDeclarations            | boolean
`--declarationDir`                           | TypeScriptDeclarationDir                   | File path
`--diagnostics`                              | *Not supported in MSBuild*                 |
`--disableSizeLimit`                         | *Not supported in MSBuild*                 |
`--emitBOM`                                  | TypeScriptEmitBOM                          | boolean
`--emitDecoratorMetadata`                    | TypeScriptEmitDecoratorMetadata            | boolean
`--emitDeclarationOnly`                      | TypeScriptEmitDeclarationOnly              | boolean
`--esModuleInterop`                          | TypeScriptESModuleInterop                  | boolean
`--experimentalAsyncFunctions`               | TypeScriptExperimentalAsyncFunctions       | boolean
`--experimentalDecorators`                   | TypeScriptExperimentalDecorators           | boolean
`--forceConsistentCasingInFileNames`         | TypeScriptForceConsistentCasingInFileNames | boolean
`--help`                                     | *Not supported in MSBuild*                 |
`--importHelpers`                            | TypeScriptImportHelpers                    | boolean
`--inlineSourceMap`                          | TypeScriptInlineSourceMap                  | boolean
`--inlineSources`                            | TypeScriptInlineSources                    | boolean
`--init`                                     | *Not supported in MSBuild*                 |
`--isolatedModules`                          | TypeScriptIsolatedModules                  | boolean
`--jsx`                                      | TypeScriptJSXEmit                          | `react`, `react-native`, `preserve`
`--jsxFactory`                               | TypeScriptJSXFactory                       | qualified name
`--lib`                                      | TypeScriptLib                              | Comma-separated list of strings
`--listEmittedFiles`                         | *Not supported in MSBuild*                 |
`--listFiles`                                | *Not supported in MSBuild*                 |
`--locale`                                   | *automatic*                                | Automatically set to PreferredUILang value
`--mapRoot`                                  | TypeScriptMapRoot                          | File path
`--maxNodeModuleJsDepth`                     | *Not supported in MSBuild*                 |
`--module`                                   | TypeScriptModuleKind                       | `AMD`, `CommonJs`, `UMD`, `System` or `ES6`
`--moduleResolution`                         | TypeScriptModuleResolution                 | `Classic` or `Node`
`--newLine`                                  | TypeScriptNewLine                          | `CRLF` or `LF`
`--noEmit`                                   | *Not supported in MSBuild*                 |
`--noEmitHelpers`                            | TypeScriptNoEmitHelpers                    | boolean
`--noEmitOnError`                            | TypeScriptNoEmitOnError                    | boolean
`--noFallthroughCasesInSwitch`               | TypeScriptNoFallthroughCasesInSwitch       | boolean
`--noImplicitAny`                            | TypeScriptNoImplicitAny                    | boolean
`--noImplicitReturns`                        | TypeScriptNoImplicitReturns                | boolean
`--noImplicitThis`                           | TypeScriptNoImplicitThis                   | boolean
`--noImplicitUseStrict`                      | TypeScriptNoImplicitUseStrict              | boolean
`--noStrictGenericChecks`                    | TypeScriptNoStrictGenericChecks            | boolean
`--noUnusedLocals`                           | TypeScriptNoUnusedLocals                   | boolean
`--noUnusedParameters`                       | TypeScriptNoUnusedParameters               | boolean
`--noLib`                                    | TypeScriptNoLib                            | boolean
`--noResolve`                                | TypeScriptNoResolve                        | boolean
`--out`                                      | TypeScriptOutFile                          | File path
`--outDir`                                   | TypeScriptOutDir                           | File path
`--outFile`                                  | TypeScriptOutFile                          | File path
`--paths`                                    | *Not supported in MSBuild*                 |
`--preserveConstEnums`                       | TypeScriptPreserveConstEnums               | boolean
`--preserveSymlinks`                         | TypeScriptPreserveSymlinks                 | boolean
`--listEmittedFiles`                         | *Not supported in MSBuild*                 |
`--pretty`                                   | *Not supported in MSBuild*                 |
`--reactNamespace`                           | TypeScriptReactNamespace                   | string
`--removeComments`                           | TypeScriptRemoveComments                   | boolean
`--rootDir`                                  | TypeScriptRootDir                          | File path
`--rootDirs`                                 | *Not supported in MSBuild*                 |
`--skipLibCheck`                             | TypeScriptSkipLibCheck                     | boolean
`--skipDefaultLibCheck`                      | TypeScriptSkipDefaultLibCheck              | boolean
`--sourceMap`                                | TypeScriptSourceMap                        | File path
`--sourceRoot`                               | TypeScriptSourceRoot                       | File path
`--strict`                                   | TypeScriptStrict                           | boolean
`--strictFunctionTypes`                      | TypeScriptStrictFunctionTypes              | boolean
`--strictNullChecks`                         | TypeScriptStrictNullChecks                 | boolean
`--strictPropertyInitialization`             | TypeScriptStrictPropertyInitialization     | boolean
`--stripInternal`                            | TypeScriptStripInternal                    | boolean
`--suppressExcessPropertyErrors`             |  TypeScriptSuppressExcessPropertyErrors    | boolean
`--suppressImplicitAnyIndexErrors`           | TypeScriptSuppressImplicitAnyIndexErrors   | boolean
`--target`                                   | TypeScriptTarget                           | `ES3`, `ES5`, or `ES6`
`--traceResolution`                          | *Not supported in MSBuild*                 |
`--types`                                    | *Not supported in MSBuild*                 |
`--typeRoots`                                | *Not supported in MSBuild*                 |
`--useDefineForClassFields`                  | TypeScriptUseDefineForClassFields          | boolean
`--watch`                                    | *Not supported in MSBuild*                 |
*MSBuild only option*                        | TypeScriptAdditionalFlags                  | *Any compiler option*

## What is supported in my version of Visual Studio?

Look in your `C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v$(VisualStudioVersion)\TypeScript\Microsoft.TypeScript.targets` file.
The authoritative mappings between MSBuild XML tags and `tsc` compiler options live in there.

## ToolsVersion

The value of `<TypeScriptToolsVersion>1.7</TypeScriptToolsVersion>` property in the project file identifies the compiler version to use to build (1.7 in this example).
This allows a project to build against the same versions of the compiler on different machines.

If `TypeScriptToolsVersion` is not specified, the latest compiler version installed on the machine will be used to build.

Users using newer versions of TS, will see a prompt to upgrade their project on first load.

## TypeScriptCompileBlocked

If you are using a different build tool to build your project (e.g. gulp, grunt , etc.) and VS for the development and debugging experience, set `<TypeScriptCompileBlocked>true</TypeScriptCompileBlocked>` in your project.
This should give you all the editing support, but not the build when you hit F5.
