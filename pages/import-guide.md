---
layout: default
---

### [Home](../)

# Import Guide

## Overview

For cBioPortal at UHN, we use a simplified import process. Using a configuration file, the [import wrapper](import-wrapper.html) is used to translate a set of pipeline data files into a directory of data which can quickly and correctly be imported into cBioPortal. Generally speaking, this is designed to fail early if the data is incorrect.

When this is done, the [import runner](import-runner.html) can take this directory of data, and update a study within cBioPortal.

## Table of contents

1. [The import wrapper](import-wrapper.html)
2. [The import runner](import-runner.html)
3. [Import data raw format](import-data.html)
4. [Annotated example import file](import-example.html)
