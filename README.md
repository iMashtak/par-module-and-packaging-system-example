# Par Lang Module and Packaging System

> Inspired by Cargo and Maven

## Packages

Par building unit is *package*. Packages has three main attributes: *group* (to distinguish author or vendor of package), *name* and *version*. In `Par.toml` we must write the following:

```toml
[package]
group = "org"
name = "Example"
version = "1.0.0"
```

Group identifiers may be of regex: `[A-z][A-z0-9-_]*`. Name identifiers must start with capital letter and cannot contain special symbols: `[A-Z][A-z0-9]*`. Version identifiers are semantic versioning things.

Each package can have a *dependencies*. They are listed as prefixed with dot and group of which it is going:

```toml
[dependencies]
org1.Dep1 = { version = "*" }
org2.Dep2 = "2.4.6"
org2.Dep1 = { version = "*", as = "Dep3" }
```

There we defined `Dep1` twice. It is ok, we allow that cause we aliased the second dependency as `Dep3`, which is not in the list of dependencies names. All names in dependencies must be unique after alias resolution.

After that we must have either `src/Lib.par` or `src/Main.par` files. Existence of the first one will tell us that this package is a library, the second one - this package is a executable. Anyway both files will be *root modules* of program.

## Modules

All definitions in Par may contain dot `.` character in their names, which may be (ab)used to create module system. Lets say that any Par program have global space of definitions and each of them should start with capital letter and may contain dots. It may be quite inconvenient to write down all prefixes each time we want to create definition so module is just a syntax sugar above this verbose definition naming.

Root module of executable must contain definition with name `Main`.

Keyword `use` allows us to create aliases of global names that we have from definitions of module and dependencies.

```par
use Dep1.Def1
```

The root module of any package is referenced by the name of this package, group is not considered.

We can write complex things in `use`:

```par
use Dep2.{Def3, Def4}
use Std.Collections.*
```

Standard library is accessible with root module `Std`.

We can declare non-root module with `module` keyword:

```par
module MyMod {
    type MyList = List<Def1>
}
```

This would be equivalent to just simply write:

```par
type MyMod.MyList = List<Def1>
```

But that syntax becomes invalid for convenience.

We can also create another file and it itself becomes module:

```par
// FileMod.par

type MyString = String
```

It must be registered in parent module (root module in our simple case) with `module` keyword:

```par
// Lib.par

module FileMod

type MyListOfString = List<FileMod.MyString>
```

If we are creating a library, we must somehow make our definitions accessible, but not all of them, only a certain. Keyword `export` allows us to mark a definition as accessible from other modules.

```par
export def MyNum = 7

export module FileMod
```

## Conclusion

As can be seen, I do not suggest to reinvent the wheel. Cargo is nice tool and Rust crates system is also good. But it lacks vendoring mechanic (like in Maven), so I think it is ok to add this from the start. Many other features of Cargo may be of use like `dev-dependencies`, project structure (examples, tests, etc), version selection for dependencies, feature flags and so on.

Have seen an example from book about encapsulation, but think that modules should be easier. I considered that there are two ways but intensionally have chosen the simple one.
