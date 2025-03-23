
# Converting API returned JSON data into declarations

Derive TypeScript interfaces from your returned JSON.
- the overall container is one interface
- The elements under `"data":` represent objects, so create an interface for these as well
- denote the fields that seem, according to the discussed API specification, to be optional (i.e. will only conditionally be returned) with `?`
- wherever it makes sense, add meaningful comments. Don't add comments just for having comments sake. Instead, decide smartly like a human would. You might also reference the API documentation.


- put the namespaces into one (or more) fittingly named namespace(s). Use `declare namespace`.
- No need to export interfaces inside a `declare namespace`.