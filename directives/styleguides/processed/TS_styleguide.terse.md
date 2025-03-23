# TypeScript Style Guide

## Your Role
- Function as senior software engineer with advanced codebase experience. Avoid novice or tutorial-style writing

## General Style
- Prefer smaller, atomic, reusable functions over larger ones. Arrow functions acceptable
- Create modular code from outset
- **Strict requirement**: Avoid anonymous object returns. Create `const result` with defined interface before returning
- Avoid anonymous string returns. Store strings in constants first
- Use and export enums for command strings
- Use liberal amounts of `enum`s and `const enum`s
- Follow one true brace style. Use braces in `switch` cases
- Omit `public` 
- Avoid `static` methods. Prefer functions when stateful objects aren't needed
- Use `import` at top level instead of `require` within functions
- Prefer functions taking object parameters for named arguments. Non-object first parameter acceptable with options object as second parameter
- Use `.map` and `.filter` liberally. Use `.reduce` sparingly when performance demands
- Prefer `for...of` loops over `.forEach`. Use descriptive names when destructuring
- Use early fails/returns
- Split complex `if` conditions into separate statements with named variables
- For asynchronous/Promise functions:
  - Prefer `Promise.catch` over `try` blocks for error handling
  - Use `await` syntax over `Promise.then` except where named functions can be used
  - Always use `await` for last asynchronous operation (because easier to reason about)
- Document arrow function purpose on following line
- Export primary functions even when executed within file

## Naming
- Function names start with lowercase, preferably verbs. Avoid noun-only names
- Never use class names containing `Manager`

## Type Annotation
- Bundle interfaces in namespaces
- Keep parameter type information on single line where feasible
- Create interfaces for config objects with >3 properties or those that are semantically extensible

## Type Declarations
- Place multiple types/interfaces in `.d.ts` files using global namespace syntax:
```
declare namespace <name1> {
    interface <name2> {
        ...
```
- Name single declaration file `interfaces.d.ts`

## Error Handling
- Structure catch blocks as:
```
catch (error) {
    // @ts-ignore
    console.error(`<CONTEXT MESSAGE>: ${error.message}`, error);
}
```
- Add `// @ts-ignore` above error property references
- Keep try-catch blocks narrow but practical
- Place constant declarations outside try blocks unless assignment can fail

## Logging
- Build multiline strings before logging rather than consecutive console.log calls
- Consider creating message classes that contain both original data and formatted output

## Architecture
- Make every file module by exporting something
- Consider Registry classes for commands or plugins
- Include `main` function that logs ISO time and project name

## Comments
- Detailed comments preferred over single-line ones
- Avoid superfluous phrases like "Function to..." or "Class for..."

## Examples
- Include example functions when appropriate, labeled with descriptions

## Domain Specific Guidelines

### Node.js CLI Apps
- Use synchronous file operations, async for most other operations
- Verify path existence before operations
- Never overwrite files; increment filenames instead
- When using dotenv:
```
import * as dotenv from 'dotenv';
dotenv.config();
```

### Web APIs
- Execute API request functions in `main`
- Warn about empty HTML that might be progressively loaded

### Web/DOM
- Use named functions instead of anonymous functions with addEventListener

## Dependencies
### General
- Any `@types/...`, lodash, minimist, ms, date-fns, yargs, clipboardy, js-yaml, yaml, dotenv, table,
- axios, sharp, playwright, react, preact, execa, fast-glob, jsdom, cheerio