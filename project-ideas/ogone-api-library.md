---
title: Ogone API library
tags: Simple, API, Web
---

## What

[Ogone](http://payment-services.ingenico.com/gb/en/online-payment-services-solutions/e-commerce/online-payments) is a payment processing system popular in the Netherlands and Belgium.

[A comprehensive implementation guide is available here](https://secure.ogone.com/ncol/Ogone_e-Com-ADV_EN.pdf).

The idea is to be able to construct a data structure with all the necessary attributes to send a payment request to the Ogone payment processing system.
Expect lots of `data` declarations and `newtype`s, but also some simple business logic (generating a SHA from all attributes, etc).

Once implemented, you can try it out yourself with a Haskell web framework.
Test accounts for Ogone are free, although they take a day or two to be approved.

## What you will learn

This project will make you think on how to structure a program with types.

* For example, the Ogone payment system expects a number of required attributes, and a number of optional attributes.
What could you do to ensure that the required attributes are *always* filled in?
Perhaps by using [smart constructors](https://wiki.haskell.org/Smart_constructors).
Or maybe some requirement checks can be made type errors.

* In the end, a POST request is sent to the payment processing system with all of the attributes included.
How can we prevent invalid attribute names being generated?

* Once processed, a payment can be in three different states:
    * Paid: payment was successful.
    * Not paid: payment was declined.
    * Uncertain: the request to the acquirer timed out, and will be tried at a later time to not keep the customer waiting.  

Ogone is able to send a HTTP request to the originating website when one of these states is reached.
Accepting and parsing these requests is a good exercise to get acquainted with Haskell web frameworks such as [Scotty](https://github.com/scotty-web/scotty), [Snap](http://snapframework.com/) and [Yesod](http://www.yesodweb.com/).

You can also test out the rest of the functionality on a simple Haskell web framework. Bonus points if you include some kind of form redirect generation functionality.
