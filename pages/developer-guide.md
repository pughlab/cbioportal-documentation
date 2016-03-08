---
layout: default
---

### [Home](../)

# Developer guide

## Tools required

Working on the development of cBioPortal requires the following tools:

* A Java 8 JDK
* Apache Maven 3.1+

I'd also recommend using Eclipse or another IDE, simply because editing Java without one can be something of a challenge.

## Useful commands

To run all the tests, package, and prepare the Maven reports:

    mvn clean test package site
