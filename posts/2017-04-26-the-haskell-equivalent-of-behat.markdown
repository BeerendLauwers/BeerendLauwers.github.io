---
title: The Haskell equivalent of Behat
---

[Behat](http://behat.org/en/latest/) is a BDD framework for PHP.
It uses a [Gherkin](https://github.com/cucumber/cucumber/tree/master/gherkin)-like structure to write human-readable tests, like these:

<pre>
Scenario: Finding some cheese
  Given I am on the Google search page
  When I search for "Cheese!"
  Then the page title should start with "cheese"
</pre>

(A bunch of scenarios are put in a `.feature`file.)

While you can express tests for anything (for example, the output of a program or function), it is often used in the context of automated web application testing:
write a test suite that goes through the flow of your web application (say, a simple webshop), and if all the tests are green, that means you're in the clear! (Or you just haven't written enough tests yet.)

Obviously, such a thing would be very nice to have for Haskell!

## Old attempts

A rough battle plan was written out here (https://github.com/sol/cucumber-haskell), and some related repositories are mentioned.

Unfortunately, the majority of these packages are all very old (>3 years), and will not have been kept up to spec with the Gherkin grammar.

On top of that, there are many elements that allow us to write a few tests and have them executed automatically on a (headless) browser, which the battle plan does not cover.

Here's how Behat does it:

## Behat overview

* [Behat](https://github.com/Behat/Behat) does two things:
  * It is the parser for the feature files.
  * It is the framework for taking such a parsed feature file, and running them as steps. The steps can be defined in extensions (called "Contexts") and distributed, and you can also write your own Context.
* The [Mink Extension](https://github.com/Behat/MinkExtension) is an integration layer Behat and Mink. 
* [Mink](https://github.com/minkphp/Mink) is a library for communicating with an abstract browser (also called a "driver" or "web driver").
  * There's also implementations for a few web drivers, of which the  [Selenium2 driver](https://github.com/minkphp/MinkSelenium2Driver) is the most important.
* Now all that remains is sending the actual commands to the web driver. In the case of the Selenium2 web driver, that's a fork of [php-webdriver](https://github.com/instaclick/php-webdriver).

Now let's look at each element in detail, but starting from the bottom.

### php-webdriver

This library establishes a connection with a Selenium2 server, opens a session, and sends commands to it.
These are all just `curl` calls.

**Do we have a Haskell equivalent?** Most certainly! The [webdriver](https://github.com/kallisti-dev/hs-webdriver) package is well-maintained, and seems to support everything.

### Mink

Mink defines a [base class](https://github.com/minkphp/Mink/blob/master/src/Driver/CoreDriver.php) for a web driver and a bunch of utility functions, 
such as converters from a CSS class selector ("Click the element with CSS class *'submit'*"), 
a named selector ("Click the button labeled *'Start'*"), etc., 
to an XPath selector, which most web drivers understand.
  
It also provides a few implementations of web drivers to fit into its system.
Most of these implementations also expose the driver-specific capabilities or functionality.

**Do we have a Haskell equivalent?** No. Such a library would provide an interface for arbitrary web drivers. 
Mink, by itself, does not handle the translation of Behat steps to web driver commands. 
The Mink extension does that.

In my opinion, this abstraction layer is not required to get something Behatty running for Haskell.
We can add it later and just have Selenium2 as the only concrete web driver.

### MinkExtension

This provides a few [Contexts](https://github.com/Behat/MinkExtension/tree/master/src/Behat/MinkExtension/Context) that allow Behat to write steps that should send commands to a web driver.

For instance, the `RawMinkContext` has a function for visiting a page:

```php
public function visitPath($path, $sessionName = null)
{
    $this->getSession($sessionName)->visit($this->locatePath($path));
}
```

`getSession` being defined as:

```php
public function getSession($name = null)
{
    return $this->getMink()->getSession($name);
}
```

`getMink()` fetches the web driver instance, and `getSession()` fetches the active web driver session.

With these kinds of building blocks, you can then write your own custom steps, which you can then refer to in your scenario.

**Do we have a Haskell equivalent?** No. And seeing as we'll probably skip Mink for the moment, it's not yet required.

### Behat

Apart from parsing files and running the functions that map to each parsed step, 
Behat also handles configuration of the web driver (Which browser? Which version? Javascript on or not?) via the `behat.yml` file.

**Do we have a Haskell equivalent?** Kinda.

For the parser part, we have the [abacate](https://hackage.haskell.org/package/abacate) package, but it is very old, 
and the BNF grammar to parse the Gherkin language is probably out of date.

#### Tangent: Gherkin, Cucumber, and Berp

The Gherkin language is essentially the language to write those human-readable scenarios we mentioned earlier.
It's managed by the [Cucumber](https://cucumber.io) team, which is something like Behat.

The [Gherkin language grammar](https://github.com/cucumber/cucumber/blob/master/gherkin/gherkin.berp) itself is written in [Berp](https://github.com/gasparnagy/berp),
which is a cross-language parser generator.

Here's how Cucumber describes its workings:

> Berp takes a grammar file (`gherkin.berp`) and a template file (`gherkin-X.razor`) as input
and outputs a parser in language *X*:
> 
>     ╔════════════╗   ┌────────┐   ╔═══════════════╗
>     ║gherkin.berp║──>│berp.exe│<──║gherkin-X.razor║
>     ╚════════════╝   └────────┘   ╚═══════════════╝
>                           │
>                           V
>                      ╔════════╗
>                      ║Parser.x║
>                      ╚════════╝
> 

So, essentially, you would put the Haskell code to generate a Haskell-based parser in the `gherkin-X.razor` file, and out would come `Parser.hs`.
Then, `Parser.hs` would be used in the rest of the Cucumber workflow:

> The following diagram outlines the architecture:
> 
>     ╔════════════╗   ┌───────┐   ╔══════╗   ┌──────┐   ╔═══╗
>     ║Feature file║──>│Scanner│──>║Tokens║──>│Parser│──>║AST║
>     ╚════════════╝   └───────┘   ╚══════╝   └──────┘   ╚═══╝
> 
> The *scanner* reads a gherkin doc (typically read from a `.feature` file) and creates
> a *token* for each line. The tokens are passed to the *parser*, which outputs an *AST*
> (Abstract Syntax Tree).

So, `Parser.hs` would have to output an AST (in JSON.). 

Finally, we need to "compile" the AST to a "pickle":

> The AST isn't suitable for execution by Cucumber. It needs further processing into a simpler form called Pickles.
> 
> The compiler compiles the AST produced by the parser into pickles:
> 
>     ╔═══╗   ┌────────┐   ╔═══════╗
>     ║AST║──>│Compiler│──>║Pickles║
>     ╚═══╝   └────────┘   ╚═══════╝

(This also outputs JSON.)

When that's all done, you would have a Cucumber-compliant implementation.

#### End of tangent

Of course, Haskell is quite good in parsing.
Generating a parser (which would have to consume a bunch of tokens) and then writing a simple compiler to simplify the AST should be quite easy.

However, it seems easier to have a 100% Haskell package first that can produce an executable from a bunch of steps, which is able to:

* Read a feature file and parse it
* Validate the AST against the loaded steps and ensure it is executable against a web driver
* Execute a valid AST against a web driver
* Puke out the corresponding output in the console

With this proof-of-concept package, full Cucumber compliance could be attained at a later stage.

So, for the non-parsing part, there are no Haskell packages available.
