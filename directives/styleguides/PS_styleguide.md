# Powershell styleguide



## General style
- write where feasible more smaller, atomic functions instead of fewer, larger ones.
- don't do anonymous direct `return {...rest of object}`. First create an output object with some descriptive name, then return that like `return <output>;`
- where "magic strings" are repeatedly used, consider making an enum.
- you might define Registry classes (based on `Map`s) that store e.g. commands or plugins.
- strongly consider splitting up `if` condition it into several statements on separate lines that create named variables. Example of worse code:
```
if !config.HasProp("hotkeys") || !IsObject(config.hotkeys) || config.hotkeys.Length() = 0 {
    return false }
```
Example of better code:
```
hksMissing := !config.HasProp("hotkeys") 
hksNotAnObject := !IsObject(config.hotkeys)
hksEmpty := config.hotkeys.Length() = 0
if (hksMissing || hksNotAnObject || hksEmpty){
    return false }
```



### Error/exception handling
- prefer early fails/returns.
- don't just wrap longer functions (more than ~10 lines long) in one single `try ... catch` block. Instead, Use multiple `try ... catch` blocks that clearly show the error reason.
- in `catch`, don't `MsgBox` with the info. Instead:
- in the `catch` part: `catch Error as e { ...` prepend a detailed error description to the `.Message` prop (separated by newline then twice a tab), then throw that `e`


### Naming
- IMPORTANT: NEVER create variable names that differ from class or function names only in the casing. Bad example: `windowInfo := WindowInfo()`. Good example: `wInfo := WindowInfo()`
- functions that are intended to be used internally in the script use regular camelCase
- your function names should always start lowercase. Try to start them with a verb. Try not to use noun-only as function or method names.
- Never use class names that contain `Manager`. Always be more creative.
- use camelCase instead of PascalCase for non-static methods; PascalCase for `static` methods
- don't use "input" as a name of a func/method parameter, because it is already a function name in AHK.

### AutoHotkey syntax notes
VERY IMPORTANT: you need to STRICTLY follow all of the following points in this subcategory.
- pay extra attention to props/fields you might hallucinate. E.g. strings don't have the `.Length` property in AHK, only arrays do. You need to call the Function `StrLen`.
- AHK new objects from classes aren't generated with `new`. Wrong syntax: `tc := new TheClass(1)`. Correct: `tc := TheClass(1)`.
-  AHK v2 cannot handle multi-line arrow funcs. So always create a named local func if you'd otherwise try to write a multi-line arrow function.
- `global` as fields inside classes is a syntax error in AHK v2. 
- objects defined like `example := {file: "--file", stdin: "--string"}` (object literals) MUST be written on one single line.
- comment blocks: `/*` and `*/` symbols can be used to comment out an entire section, but only if the symbols appear at the beginning of a line. So `/* <...> */` is wrong, it should be 
```
/*
<...>
*/
```
- if inside a method/function a class is called, we need to refer to the global scope. At the start of the method/function, use the `global` (standalone) declaration (not `global <name>`)
- IMPORTANT: ALWAYS keep the limitations about scope in AutoHotkey v2 in mind -- i.e. assumed-local: inside the body you might not have access to the global scope. So you'd need to declare `global` first. Only do this grouping into classes if you are *absolutely* sure nothing will break. Remember that AHK doesn't have hoisting. mind the scope limitations. position in file matters.


### Handling strings
- IMPORTANT: When a string has variable parts or you want to concatenate some to insert a value in the middle, instead use a "continuation section".
- use this together with the `Format(` function.
Especially if you need to use quote marks (") or backticks (`) you MUST use these continuation sections.
Bad example:
```
; Use backticks for string values in maps
if IsString(value) {
    preview .= key ": `" value "`"
} else {
    preview .= key ": " value
}
```
Good Example:

```
; Use backticks for string values in maps
if IsString(value) {
    preview .= key Format("
    (
    : ``{}``
    )", value)
} else {
    preview .= key ": " value
}
```


- The ONLY multiline strings supported in AHK are the continuation sections. These are defined like so:
```
Var := "
(
A line of text.
By default, the hard carriage return (Enter) between the previous line and this one will be stored.
	This line is indented with a tab; by default, that tab will also be stored.
Additionally, "quote marks" are automatically escaped when appropriate.
)"
/*
i.e. quote mark \newline Parentheses open \newline The 1st line ... The last line \newline Parenth close quote mark
In the examples above, a series of lines is bounded at the top and bottom by a pair of parentheses. This is known as a continuation section. Notice that any code after the closing parenthesis is also joined with the other lines (without any delimiter), but the opening and closing parentheses are not included.

If the line above the continuation section ends with a name character and the section does not start inside a quoted string, a single space is automatically inserted to separate the name from the contents of the continuation section.

Quote marks are automatically escaped (i.e. they are interpreted as literal characters) if the continuation section starts inside a quoted string, as in the examples above. Otherwise, quote marks act as they do normally; that is, continuation sections can contain expressions with quoted strings.
*/
```
- When you have strings with variable content, you should make use of the continuation section syntax. You can then format variable input like this:
```
Var4 := Format("
(
A line of text.
First var {1}
Second var "{2}"
)", var5, var6)
```
or
```
 Var2 := "
(
A line of text.
First var {1}
Second var "{2}"
)"
Var3 := Format(Var2, "1st var", "2nd var")
```


### Comments
- If I have given you pseudocode/schematic code/skeletons: you MUST take the comments I have already written, fix them up a bit WITHOUT making them significantly more terse, but also do not add superfluous wording. You CANNOT leave these pre-existing comments out at the spots where I have inserted them. They are documentation. Always insert them at the fitting place.
- before each function as well as most methods, add a closely JSDoc-inspired block denoting the types. However, unlike JSDoc, use AHK types; and also, expand the output object inline. IMPORTANT: remember that AHK uses a different comment syntax than JavaScript.
- prepend comment blocks according to the following schema before each function as well as most methods
```
/*
 * <OPTIONAL: perhaps some explanation if apt>
 * @params      (a: number, b: number)      <OPTIONAL: comment/explanation, separated by 4 tabs>
 * @returns     {number}                    <OPTIONAL: comment/explanation, separated by 4 tabs>
 */
```
 before `addNumbers(a, b)`, or in the case of returned objects
```
/*
 * <perhaps some explanation if apt>
 * @params      (a: number, b: number)
 * @returns     {object}        {     <for a plain object put "object", otherwise the class name in curly braces>. Then, write out the interface>
 *                  name: string        <if it's not obvious/self-evident/self-documenting, write a comment with a description of the prop here>
 *                  id: number
 *                  <...if more than 3 entries or if long prop comments>
 *                   <align so everything is on the same vertical align>
 *              }
 */
 ```
 - in the case of classes like `Map` (or `ValueError` etc.):
 ```
 /*
 * <perhaps some explanation if apt>
 * @params      (a: number, b: number)
 * @returns     {Map}
 */
 ```
 - if you write an example, tag it like 
```
 * <... rest ... next line is supposed to be an empty comment line>
 * 
 * @example
 * <... the example ...>
 */
```
- By contrast: refrain from littering the script with one-liner comments describing what the next line does. Analyze whether the comment you would be including is "obvious" or "low info", and if yes, then don't include it.
 - Be informative and thoughtful with the comments. 
 AD comment example:
```
; Error handler function
ErrorHandler(exception, mode) {
```
Use your brain. You see that the example comment above doesn't actually explain anything because it just restated the function name/type.


### File operations
- beforehand check if path even exists
- ALWAYS strictly non-overwriting. If a file already exists, write to an incremented filename instead.



# Task requirements 
IF my directives passed directly into your chat box or a different markdown than these inside this "AHK_style_guide.md" markdown mention that you should create an AHK app, or if they mentions tests, then it means the code you are generating must additionally be wrapped in sections to enable it to be ran directly.
If that is the case, then:
- Prepend this to the code you will be creating
```
#Requires AutoHotkey v2.0
#Include ./utils.ahk


#SingleInstance force
#Warn All, Off    
OnError(handleExceptions)
```
(your code is in between)
- write a main function called `main` and then execute it with `main();`. Put it at the end of the file.
-  At its beginning, log to stdout with the current ISO time formatted as "hh:mm:ss" plus ` -- <<${a meaningful project name}>>`
- create some tests in form of a class with some relevant name, and run them. Run them by calling `<RelevantClassName>.runTests()`. Colorize this output/test results. E.g. use green checkmarks and red Xs. 


if I tell you to specifically create an AHK CLI app:

- prepend to the code
``` 
#SingleInstance force
#Warn All, Off    
OnError(handleExceptions)
```
- append at the very end of file:
```
handleExceptions(exception, mode) {
    Stdout.write("[ERROR]:    " exception.Message)
    Stdout.write("    File: " exception.File)
    Stdout.write("    Line: " exception.Line)
    Stdout.write("    Extra: " exception.Extra)
    ;ExitApp(1)
}
```


===========
Utilities: (you can use the code in this)
utils.ahk:
```autohotkey

; Function to resolve and normalize a path relative to a base directory
ResolvePath(baseDir, targetFile) {
    ; Remove trailing slashes from baseDir
    baseDir := RTrim(baseDir, "\/")
    
    ; Normalize the target file path (replace / with \ and remove ./)
    targetFile := StrReplace(targetFile, "/", "\")
    targetFile := StrReplace(targetFile, ".\", "\") ; Remove .\ if present
    
    ; Join the base directory and target file
    resolvedPath := baseDir "\" targetFile
    
    ; Normalize the path (replace multiple slashes with a single slash)
    resolvedPath := RegExReplace(resolvedPath, "\\+", "\")
    
    return resolvedPath
}





/*
 * Converts array to string with specified delimiter
 * in AHK, arrays start at 1.
 * @params      (arr: array, delimiter: string)
 * @returns     string    Joined array elements
 */
Array_Join(arr, delimiter := "`n") {
    if !IsArray(arr) {
        throw ValueError("Input must be an array", -1)
    }

    if arr.Length = 0 {
        return ""
    }

    try {
        result := ""
        for index, value in arr {

            result .= value

            if (index < arr.Length) {

                result .= delimiter

            }

        }

        return result

    } catch Error as e {

        throw ValueError("Failed to join array: " e.Message, -1)

    }

}





/*
 * Checks if the input parameter is a Map object.
 * @params      (input: any) The value to check
 * @returns     boolean      True if the input is a Map, false otherwise
 */
IsMap(input) {
    ; Early return if input is not an object
    if !IsObject(input) {
        return false
    }

    ; Check if the input has the base of a Map
    try {
        return input.HasBase(Map.Prototype) || input.HasBase(Map)
    } catch {
        return false
    }
}




; Returns true if the input is a boolean value, false otherwise
IsBool(value) {
    ; In AHK v2, true is exactly 1 and false is exactly 0
    return value = true || value = false
}



; Function to check if a variable is a string
IsString(value) {
    ; 1. Use the Type() function to identify the type of the variable.
    ;    Type() returns "String", "Integer", "Float", "Object", "Func", etc.
    if (Type(value) = "String") {
        return true ; The variable is a string.
    }

    ; 2. Check for edge cases:
    ;    a. If the value is an object, it is not a string.
    if IsObject(value) {
        return false
    }

    ;    b. If the value is numeric, it is not a string even if quoted.
    ;       (Numeric strings like "123" will still pass as strings, which is intentional.)
    if IsNumber(value) {
        return false
    }

    ; 3. If all checks fail, return false.
    return false
}







; IsArray() - Checks if a value is an array
; Returns true if the input is an array, false otherwise
IsArray(value) {
    try {
        ; Try to access array methods/properties
        ; If these succeed, it's likely an array
        length := value.Length
        capacity := value.Capacity
        return true
    } catch {
        return false
    }
}


/*
 * Checks if a value is null/undefined or empty.
 * For strings, checks if length is 0.
 * For arrays, checks if array is empty.
 * @params      (x: any) The value to check
 * @returns     boolean  True if value is null, undefined, or empty
 * @throws      ValueError if x is of an unsupported type
 */
isNullOrEmpty(x) {
    global
    ; If x is not set, consider it null/empty
    if !IsSet(x) {
        return true
    }

    ; Check for array
    if IsArray(x) {
        return x.Length = 0
    }

    ; Check for string
    if IsString(x) {
        return x = ""
    }

    ; If x is of an unsupported type, throw a ValueError
    throw Error(Format("ValueError: Unsupported type. Expected Array or String. Got: {}", Type(x)), -1, x)
}





/*
 * Checks if a value is callable (a function or a functor).
 * @params      (value: any)
 * @returns     boolean
 */
IsCallable(value) {
    ; Check if the value is a function
    if (Type(value) = "Func") {
        return true
    }

    ; Check if the value is an object with a Call method
    if (IsObject(value) && value.HasMethod("Call")) {
        return true
    }

    ; Otherwise, the value is not callable
    return false
}





/*
 * Enum
 * Represents primitive and complex types in the system.
 */
class Types {
    static String := "string"
    static Boolean := "boolean"
    static Number := "number"
    static Array := "array"
    static Object := "object"
    static Undefined := "undefined"
}




/*
 * Determines the type of a given value according to the Types enum.
 * Provides consistent type checking across the application.
 * @params      (x: any) The value to check
 * @returns     Types    The type of the value as defined in the Types enum
 */
getType(x) {
    global
    if !IsSet(x) {
        return Types.Undefined
    }
    if IsArray(x) {
        return Types.Array
    }
    if IsObject(x) {
        return Types.Object
    }
    if IsNumber(x) {
        return Types.Number
    }
    if IsBool(x) {
        return Types.Boolean
    }
    return Types.String
}



/*
 * Enum
 * Constants for callback execution order.
 */
class Pos {
    static After := 1    ; Call the callback after any previously registered callbacks
    static Before := -1  ; Call the callback before any previously registered callbacks
    static Noop := 0     ; Do not call the callback
}

/*
 * Enum
 * Defines different error handling modes for the system.
 */
class ErrorMode {
    static Return := "Return"     ; Continuable runtime error
    static Exit := "Exit"         ; Non-continuable runtime error
    static ExitApp := "ExitApp"   ; Critical runtime error
}






/*
 * Custom assertion function for testing
 * @params      (condition: boolean, message: string)
 * @returns     void
 * @throws      AssertionError if condition is false
 * @example
 *   assert(1 + 1 = 2, "Basic math failed")  ; Passes
 *   assert(1 + 1 = 3, "Bad math")          ; Throws AssertionError
 */
assert(condition, message := "Assertion failed") {
    if (!condition) {
        e := Error("AssertionError", -1)
        e.Message := message
        throw e
    }
}

/*
 * Converts a Map to an Array by extracting its values
 * @params      (map: Map)
 * @returns     {Array}
 * @throws      ValueError if input is not a Map or conversion fails
 * @example
 *   testMap := Map("a", 1, "b", 2)
 *   arr := mapToArray(testMap)  ; Returns [1, 2]
 */
mapToArray(map) {
    if !IsMap(map) {
        throw ValueError("Input must be a Map object", -1)
    }

    try {
        output := []
        for _, value in map {  ; In AHK v2, we iterate over the Map directly
            output.Push(value)
        }
        return output
    } catch Error as e {
        throw ValueError("Failed to convert Map to Array: " e.Message, -1)
    }
}

/*
 * Converts array or map to string with advanced options
 * @params      (input: Array|Map, opts: Map) {
 *                  input: Array|Map       The array or map to convert
 *                  opts: Map {            Optional settings
 *                      delimiter: string  Custom delimiter (default: space)
 *                      trim: boolean      Trim items (default: true)
 *                      skipEmpty: boolean Skip empty items (default: true)
 *                      sort: boolean      Sort items (default: false)
 *                  }
 *              }
 * @returns     {string}    Joined elements
 * @throws      ValueError for invalid inputs
 * @example
 *   arr := ["a", "b", " c "]
 *   opts := Map("delimiter", "|", "trim", true)
 *   str := arrayToString(arr, opts)  ; Returns "a|b|c"
 */
arrayToString(input, opts := 0) {
    ; Default options
    defaultOpts := Map(
        "delimiter", "",
        "trim", true,
        "skipEmpty", true,
        "sort", false
    )

    ; Merge provided options with defaults
    if IsMap(opts) {
        for key, value in defaultOpts {
            if !opts.Has(key) {
                opts[key] := value
            }
        }
    } else {
        opts := defaultOpts
    }

    ; Handle Map input
    if IsMap(input) {
        input := mapToArray(input)
    }

    ; Validate input is array
    if !IsArray(input) {
        throw ValueError("Input must be an Array or Map", -1)
    }

    ; Handle empty array
    if input.Length = 0 {
        return ""
    }

    try {
        ; Process array items
        processedItems := []
        for item in input {
            ; Convert item to string if needed
            strItem := IsObject(item) ? JSON_stringify(objectToMap(item)) : String(item)
            
            ; Apply trimming if enabled
            if opts["trim"] {
                strItem := Trim(strItem)
            }
            
            ; Skip empty items if enabled
            if opts["skipEmpty"] && strItem = "" {
                continue
            }
            
            processedItems.Push(strItem)
        }

        ; Sort if enabled
        if opts["sort"] {
            processedItems.Sort()
        }

        ; Join items with delimiter
        return Array_Join(processedItems, opts["delimiter"])
    } catch Error as e {
        throw ValueError("Failed to convert to string: " e.Message, -1)
    }
}



/*
 * Recursively converts an object (plain object, Map, Array, etc.) into a Map with only Array or Map as children.
 * @params      (obj: any, errors: Array := [], path: string := "")
 * @returns     Map
 * @throws      ValueError if conversion fails, with a digest of errors
 */
objectToMap(obj, errors := [], path := "") {
    ; Log the current object being processed
    ;Log_debug(Format("Processing object at path: {1}", path))

    ; Initialize the output Map
    output := Map()

    ; Handle null or undefined objects
    if (!IsSet(obj) || obj == "") {
        Log_debug(Format("Object at path {1} is null or undefined", path))
        return output
    }

    ; Handle primitive types (string, number, boolean)
    if (!IsObject(obj)) {
        Log_debug(Format("Object at path {1} is a primitive: {2}", path, obj))
        return obj
    }

    ; Handle Arrays
    if (IsArray(obj)) {
        Log_debug(Format("Object at path {1} is an Array", path))
        output := Array()
        for index, value in obj {
            try {
                output.Push(objectToMap(value, errors, path "[" index "]"))
            } catch Error as e {
                errors.Push(Format("Failed to convert array element at {1}[{2}]: {3}", path, index, e.Message))
            }
        }
        return output
    }

    ; Handle Maps
    if (IsMap(obj)) {
        Log_debug(Format("Object at path {1} is a Map", path))
        for key, value in obj {
            try {
                output[key] := objectToMap(value, errors, path "." key)
            } catch Error as e {
                errors.Push(Format("Failed to convert Map entry at {1}.{2}: {3}", path, key, e.Message))
            }
        }
        return output
    }

    ; Handle plain objects
    if (Type(obj) = "Object") {
        Log_debug(Format("Object at path {1} is a plain object", path))
        for key, value in obj.OwnProps() {
            try {
                output[key] := objectToMap(value, errors, path "." key)
            } catch Error as e {
                errors.Push(Format("Failed to convert property at {1}.{2}: {3}", path, key, e.Message))
            }
        }
        return output
    }

    ; Handle unsupported types
    errors.Push(Format("Unsupported type at path {1}: {2}", path, Type(obj)))
    return output
}






/*
 * Handles all stdout operations with proper error handling
 * and formatting capabilities.
 */
class Stdout {
    static write(x) {
        try {
            FileAppend x "`n", "*"  ; Standard output
        } catch Error as e {
            FileAppend e.Message "`n", "*"
            throw ValueError("Failed to write to stdout", -1)
        }
    }

    static send(x) {
        if !IsSet(x) || x = "" {
            throw ValueError("Cannot send null or empty value to stdout", -1)
        }
        Stdout.write(x)
    }


    static getConsoleHandle() {
        ; Get a handle to the console (stdout)
        return DllCall("GetStdHandle", "Int", -11, "Ptr")
    }

}


/*
 * Logs debug messages to stdout if the AHK_DEBUG environment variable is set to "true", true, or 1.
 * @params      (message: string)
 * @returns     void
 */
Log_debug(message) {
    ; Retrieve the AHK_DEBUG environment variable
    ahkDebug := EnvGet("AHK_DEBUG")

    ; Check if AHK_DEBUG is "true", true, or 1
    if (ahkDebug = "true" || ahkDebug = true || ahkDebug = 1) {
        Stdout.write("[DEBUG] " message)
    }
}


```

## AHK v2 SYTNTAX NOTES
Objects:
There is now a distinction between properties accessed with . and data (items, array or map elements) accessed with []. For example, dictionary["Count"] can return the definition of "Count" while dictionary.Count returns the number of words contained within. User-defined objects can utilize this by defining an __Item property.

When the name of a property or method is not known in advance, it can (and must) be accessed by using percent signs. For example, obj.%varname%() is the v2 equivalent of obj[varname](). The use of [] is reserved for data (such as array elements).

The literal syntax for constructing an ad hoc object is still basically {name: value}, but since plain objects now only have "properties" and not "array elements", the rules have changed slightly for consistency with how properties are accessed in other contexts:
•o := {a: b} uses the name "a", as before.
•o := {%a%: b} uses the property name in a, instead of taking that as a variable name, performing a double-deref, and using the contents of the resulting variable. In other words, it has the same effect as o := {}, o.%a% := b.
•Any other kind of expression to the left of : is illegal. For instance, {(a): b} or {an error: 1}.



