npm issue: peer dependencies dedupe

install packages at the root directly  ``npm install``

We've encountered a wrong behavior with npm dedupe functionality with peer dependencies. 
In this demo, there are 3 modules and their dependencies tree looks like this:


```js
// main-app
"package-a": "file:../package-a",
"package-b": "file:../package-b",
"graphql": "15.0.0"

// package=a
 "graphql": "16.5.0",
 "@graphql-tools/schema": "8.3.8", // peerDependency: graphql "^14.0.0 || ^15.0.0 || ^16.0.0 || ^17.0.0"
 "@graphql-tools/stitch": "8.6.12" // peerDependency: graphql "^14.0.0 || ^15.0.0 || ^16.0.0 || ^17.0.0"

// package-b
"graphql": "15.0.0"
```

**Expected behavior:**

```js
// node_modules (root)
│  "graphql": "15.0.0" // deduped 
│
└───project-a/node_modules
│   └───"graphql": "16.5.0"
│   └───"@graphql-tools/schema": "8.3.8" // should not be deduped as the project depends on graphql: 16.5.0
│   └───"@graphql-tools/stitch": "8.6.12" // should not be deduped as the project depends on graphql: 16.5.0
│   
└───project-b/node_modules
    └───"graphql": "15.0.0" // deduped (using root package)
```

**Current behavior:**

`@graphql-tools/schema` uses the wrong `graphql` version from the root project (`main-app`).
The version of `graphql` for `@graphql-tools/schema` should be decided by the actual consumer of `@graphql-tools/schema`, which in this case is `project-a`, not `main-app`. 

Deduplication is not taking this into consideration, and should consider the version of `graphql` installed in `project-a` to be the required version for another dependancies defining `graphql` as peer dependancy, like `@graphql-tools/schema` in this scenario. 

Alternativelly, packages with peer dependancies should not be deduped at all for version safety. 

```js
// node_modules (root)
│  "graphql": "15.0.0" // deduped 
│  "@graphql-tools/schema": "8.3.8"  // issue lays here where this package now references the root graphql which is a different version
│  "@graphql-tools/stitch": "8.6.12"
│
└───project-a/node_modules
│   └───"graphql": "16.5.0" // not deduped
│   
└───project-b/node_modules
    └───"graphql": "15.0.0" // deduped (using root package)
```
