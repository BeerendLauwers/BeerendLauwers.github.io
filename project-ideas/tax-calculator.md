---
title: A simple, extensible tax calculator for your country
tags: Simple, Command-Line, GUI, Extensible
---

## What

Tax calculators tend to be these huge, bloated codebases that crudely try to model the flow of a bunch of values as they pass through a bunch of rules.

Creating a tax calculator for your country is a nice way to get acquianted with stateful programming in Haskell and allows you to start small and build on your previous efforts.

## What you will learn / Rough plan

Most countries have a few very generic stages in a tax calculation.
For example, [The Netherlands uses three income boxes](http://www.belastingdienst.nl/wps/wcm/connect/bldcontenten/belastingdienst/individuals/tax_arrangements/living_abroad/non_resident_taxpayer_status/income_in_multiple_boxes2/) to partition their tax calculation.

You could start with modelling that partitioning, fleshing parts out as you progress.

On a high level, a tax calculator is a list of transformation functions on a collection of input values.
Depending on the values, transformation functions can be added or removed from the list.

How do we pass around the values to each transformation function?
You could use the collection of Reader / Writer / State monads to make it easy to pass them around.

How could you make the data structure extensible?
Some new functionality might require more values, so we would have to update our data structure that models the values as we progress.
You could try out [Vinyl](https://github.com/VinylRecords/Vinyl) or a similar library to see if it makes adding values easier.

Perhaps write a simple desktop-based GUI for it?
There are several Linux libraries available, such as [Gtk2Hs](https://github.com/gtk2hs/gtk2hs).