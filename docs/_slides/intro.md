---
editor_options:
  chunk_output_type: console
---

## Objectives for this Lesson

- Observe characteristics of wildfire in remote sensing data
- Find "unusual" pixels in time-series of raster data

===

## Specific Achievements

- Distinguish "stacks" from "bricks"
- Use MODIS derived NDVI
- Execute efficient raster calculations
- Perform PCA on "bricks"

===

## Import Clarification

Functions from the [raster](){:.rlib} package dominate this lesson, so load all
its elements and use them without a pointer back to the library. Import
functions from other packages as needed using the [modules](){:.rlib} library,
to make their source clear for training purposes.


~~~r
library(raster)
library(modules)
import('magrittr', '%>%')
~~~
{:.text-document title="{{ site.handouts[0] }}"}
