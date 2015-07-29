---
title: Online household financial management tool
tags: Intermediate, Web, Parsing, Extensible
---

## Description

[AFAS Personal](http://www.afaspersonal.nl) is a simple financial management tool.
Its main features include:

* The ability to export your financial data from your online banking site into the tool via a [browser extension](https://chrome.google.com/webstore/detail/afas-personal-bijwerk-ass/fhdjnejhhklnclpkbnfmfimijnlmghfk?hl=nl)
* Supports several banks
* Automatic tagging of transactions by searching for similar transactions and through (user-definable) tagging rules
* Ability to add one or more labels to a transaction
* Ability to add attachments to a transaction
* Splitting up a transaction into two transactions
* Update or remove a transaction
* Revert a transaction to its original, imported state
* Budget overview with fancy charts to indicate what goes where on an per-account basis (or for all accounts)
* Overview of running contracts and automatic generation of letters of request for service cancellation
* Pay slip management
* Pension calculation (NL only)
* Multiple-access

The idea here is to recreate a subset of the aforementioned functionality, with the ability to extend it later on with more features.
Finally, have it run on a VM / Docker instance (or just on your local machine).

## What you will learn

A lot, actually.
This would take you through the entire cycle of thinking about program structure, development, testing / debugging and deployment.

### Things you will encounter:

* Parsing: It might be that your bank exports your data to a CSV file or a custom file format, in which case you'll have to learn about parsing / parsing combinators.
* Working out program architecture: For example, think about what the type of a function that splits up a transaction could look like. Perhaps something like `Transaction -> [Transaction]`? Hmm, we'll need some more info...
* Passing data to the browser and the UI: Lots of opportunity here to learn about [Fay](https://github.com/faylang/fay/wiki), [Elm](http://elm-lang.org/), [Haste](http://haste-lang.org/), [ghcjs](https://github.com/ghcjs/ghcjs) or `aeson` + your favourite JS templating framework.
* FRP: Perhaps you'd like it all to be reactive? Perhaps work it out without FRP first and then refactor it as a lesson in refactoring?
* Authentication and security: you'll want to take best practices into account here.
* Web frameworks / libraries: Perhaps you can do some of the heavy lifting in processing transactions on a Haskell server.
* Databases: Perhaps you'll have the client do everything and just use the server as a persistent store. 
* Saving functions: But what about the tagging rules? Perhaps simply write out the function to a file and save it to the persistent store, hot-loading it later? Or perhaps a simple embedded DSL that allows people to piece together new filter functions from existing (default and custom!) ones? Crowd-sourced filter functions, anyone?

You can always throw more stuff into the mix.
Want to use `lens` in an actual project? Go for it!
Want to create a simple data store that can be queried via a web API generated with `servant` and write the program logic in another language entirely? No problem.

## Rough implementation guide

Because this is a very broad project, the suggested steps are *very* rough.

1. You'll probably want to be able to get your transaction data from your bank first. 
This might involve a lot of hoops or the use of a browser extension to do it reliably.

2. Then, you'll want to be able to parse that raw data into a Haskell data structure to do stuff with it.

3. From here on out, you're on your own. Some high-level design decisions for you to contemplate:
 * Will I have the client do most of the calculations and just treat the server as a simple data store, or will the server do more stuff? Perhaps both and cross-compile with GHCJS?
 * Perhaps all data could be downloaded to client available for offline viewing.
 * How will I handle the UI?
 
## Bonus features

* A transaction can only be placed in a single category. 
  Why not more than one? A transaction could easily be a vacation expense and a dinner expense.
* Add an audit log that identifies the changes that were done (updating/splitting up a transaction,etc).
  Perhaps add the ability to roll back to a previous changeset? This opens up the fun idea of keeping your data in a git repository.
* Virtual accounts: based on *something* (originating account number, name, category, transaction note, etc), a transaction is placed in a virtual account.
  * Perhaps something like `type VirtualAccount = [Source]`, where `Source` is a list of filters on all transactions. For example,
    ```Haskell
    fortisFilter = AccountFilter (Fortis "0012020333")
    source = [fortisFilter]
    ```
* Make all data queryable via a(n) (secured) API.
* Make all data exportable to Excel / CSV / JSON / an SQL OLAP cube.
* Make it possible to import a data dump from AFAS Personal ;)

