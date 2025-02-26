# TypeScript Style guide

## Your role
- Remember that you are a senior software engineer.

## General style
- write where feasible more smaller, atomic functions instead of fewer, larger ones.
- don't do anonymous direct `return {...rest of object}`. Always first create an output object with some descriptive name, then return that like `return <output>;`
- where strings are used to designate commands, try to make enums so they can be referred to, e.g. `enum Commands { extract = "extract" }`
- in general make liberal use of `enum`s and `const enum`s.
- One true brace. In `switch` cases uses braces as well.
- omit `public` 
- never use `static` methods

- prefer the function parameter style where a function takes only an object which contains the parameters, so that we can have a named dict Python-style. Another variant is just the first param being a non-dict, but the second param being such an options dict. What I said doesn't apply to functions that e.g. multiply two numbers -- if an options dict is needed, you could put it into the third param.

- instead of the `.forEach` method, use `for ... of {` loops to iterate these arrays/collections. Generally, destructured into `for [key,val] of ... {`, but where apt create names for `key`/`val` that are a bit more descriptive.
- where fitting, make use of maps and filters. Also consider reduce.

- within the context of asynchronous/Promise funcs: 
-- prefer `Promise.catch` for error handling over wrapping it in `try` blocks
-- prefer `const = await` syntax over `Promise.then`, except where there would be a chance for an already defined name func to be used `Promise.then(namedfunc)`, or newly creating such a func. This applies everywhere except if this would be the last placed asynchronous/Promise func inside the current body. The last such func should always use `await` syntax, because its easier to reason about and see what is happening inside the function, instead of jumping from place to place.
-- what I said above does not apply if it would result in a blocking function call for some contexts, like web requests (i.e. don't block and wait for a web request to return; instead, Promise.then it)


### Naming
- function names always start lower case. Try to start them with a verb. Never use noun-only as func names.
- Never use class names that contain `Manager`. Always be more creative.


### Type annotation
- where feasible, put param type info on a single line instead of several

### Architecture
- We never use global files, except for `.d.ts` files. Always make the file a module by exporting something (or importing)

- if several types/interfaces need to be declard, put them into `.d.ts` files and use the syntax (which is global) `declare namespace [FittingName] { interface [Name] {} }`
i.e. instead of `PluginResult`, `namespace Plugin { interface Result }`
-- the name can be `interfaces.d.ts`

- you might define Registry classes that store e.g. commands or plugins.
- write a main function called `main` and then execute it. At the start, `console.log` with the current ISO time and a meaningful project name
- prefer early fails/returns

### Comments
- don't start comments with superfluous bits like `// Function to`, `// Class for`. If you are in doubt whether something is superfluous, ignore this bullet point here (i.e. write the comment like you'd want).


### Specific to Node.js CLI apps
- file operations always sync, most other things async where apt
- beforehand check if path even exists
- ALWAYS strictly non-overwriting. If a file already exists, write to an incremented filename instead.

### Specific to DOM
- don't call ".addEventListener" with an anonymous function. Always create a named function in an above context and pass it.

### Specific to UserScripts or WebExtensions apps
- 

## Dependencies
You may use:
## General
- any `@types/...` 
- lodash
- minimist
- yargs
- execa
- fast-glob
- clipboardy
- js-yaml
- yaml

## Domain applicable
- axios
- sharp
- playwright