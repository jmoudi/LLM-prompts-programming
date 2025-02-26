# LLM promps programming

This repo contains directives I give to LLMs for programming.

/directives contains the content passed to the LLM.

This concerns mainly code generation. Other uses are code conversion, letting the LLM output interfaces, and app/programming language documentation.

## Terminology
Prompt: the overall text sent to the LLM
Task: whatever the LLM should accomplish. Whatever the desired output is.
Command: an (optional) shorthand used to tell the LLM what it should do.
Directive: an explanation for the LLM that  modulates how it should do the task. These can range from slightly imperative to very imperative.
Code: pseudocode in the sense of the algorithmic, procedural parts of a program. Not just anything in a code block, and interfaces are excluded here.


## Strategies
The main ingredients I use for LLM programming are:
- style guides
- metalanguage
- code
- interfaces
- examples

### Style guide
Defines how the LLM should write the code, according to language.
This is the core part of this repo.

### Metalanguage
This is optional.
It is advantageous to define a metalanguage, a kind of command syntax or shorthand, for the commands you pass to the LLM.
For example, I could say: `js deps=lim: write an app that <...>", where the `deps=lim` parameter would point to the JS dependencies the LLM is allowed to use. These are defined elsewhere. Here it might mean that the LLM can use the dependencies `axios, lodash, yarn` from npm. 

### Code
If a task needs to be described algorithmically, in 90% of cases pseudocode does the trick.
However, in some cases you need to write either a code skeleton, or give it actual code, so the LLM can process the task correctly.
In that case, I favor giving it code that aligns with Javascript/Typescript principles.
I strongly believe ES6 Javascript syntax is more useful and elegant, and less ambigious and error-prone than that of Python. There is also no ambiguity whether a JS function passes by ref or by value.

### Interfaces
Interfaces writte in Typescript syntax. 
While effort is made to keep them legitimate Typescript code, in some cases a slightly alternate syntax is utilized. In that case they may be regarded as textual/non-graphical version of UML.

### Examples
While examples aren't the bread-and-butter tool, when they are used they usually are the most potent one. Nothing disambiguates a user instruction as much as well-provided example. As such, I bundle some if the examples are too long to be included in-line.