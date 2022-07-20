npm issue: peer dependencies dedupe

install packages at the root directly 
> npm install

We've encounter a wrong behavior with npm dedupe functionality with peer dependencies. 
On this demo there are 3 modules and their dependencies tree looks like this:


```js
// main-app
"package-a": "file:../package-a",
"package-b": "file:../package-b",
"graphql": "15.0.0"

// package=a
 "graphql": "16.5.0",
 "@graphql-tools/schema": "8.3.8", // peerDependency: graphql
 "@graphql-tools/stitch": "8.6.12" // peerDependency: graphql

// package-b
"graphql": "15.0.0"
```

Expected behavior would be:
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

Current behavior which leeds to wrong `@graphql-tools/schema` to use the wrong graphql version

```js
// node_modules (root)
│  "graphql": "15.0.0" // deduped 
│  "@graphql-tools/schema": "8.3.8"  // issue lays here where this package now reference the root graphql which is different version
│  "@graphql-tools/stitch": "8.6.12"
│
└───project-a/node_modules
│   └───"graphql": "16.5.0" // not deduped
│   
└───project-b/node_modules
    └───"graphql": "15.0.0" // deduped (using root package)
```
