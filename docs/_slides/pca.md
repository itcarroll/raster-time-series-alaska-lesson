---
---

## Eliminating Time

Because changes to NDVI at each pixel follow a similar pattern over the course
of a year, the slices are highly correlated. Consider representing the NDVI
values as a simple matrix with

- each time slice as a variable
- each pixel as an observation

PCA is a technique for reducing dimensionality of a dataset based on correlation
between variables. The method proceeds either by eigenvalue decomposition of a
covariance matrix or singular-value decomposition of the entire dataset.
{:.notes}

===

To perform PCA on raster data, it's efficient to use specialized tools that
calculate a covariance matrix without reading in that big data matrix.



~~~r
ndvi_lS <- layerStats(
  ndvi, 'cov', na.rm = TRUE)
ndvi_mean <- ndvi_lS[['mean']]
ndvi_cov <- ndvi_lS[['covariance']]
ndvi_cor <- cov2cor(ndvi_cov)
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


Here, we use `layerStats` to calculate the covariance between each pair of
layers (dates) in our NDVI brick, and the mean of each date. 
`layerStats` calculates summary statistics 
across multiple layers of a `RasterStack` or `RasterBrick`, analogous to how 
`cellStats` calculates summary statistics for each layer separately. After
calculating the summary statistics we extract each one to a separate object 
and then scale the covariance matrix using `cov2cor`.
{:.notes}

===

The `layerStats` function only evaluates standard statistical summaries. The
`calc` function, however, can apply user-defined functions over or across raster
layers.



~~~r
ndvi_std <- sqrt(diag(ndvi_cov))
ndvi_stdz <- calc(ndvi,
  function(x) (x - ndvi_mean) / ndvi_std,
  filename = file.path(out, 'ndvi_stdz.grd'),
  overwrite = TRUE)
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


Here we find the standard deviation of each NDVI time slice by pulling out
the diagonal of the covariance matrix with `diag()`, which contains the variance of
each time slice, then taking the square root of the variance. Next, inside `calc()`,
we define a function inline to standardize a value by subtracting the mean then
dividing by the standard deviation, then apply it
to each pixel in each layer of `ndvi`. We use `filename` to write the output
directly to disk.
{:.notes}

===

Standardizing the data removes the large seasonal swing, but not the correlation
between "variables," i.e., between pixels in different time slices. Only the
correlation matters for PCA.



~~~r
> animate(ndvi_stdz, pause = 0.5, n = 1)
~~~
{:title="Console" .no-eval .input}


![plot of ndvi_stdz_animation]({% include asset.html path="images/ndvi_stdz_animation.gif" %})
{:.captioned}

===

Now, we use `princomp` to calculate the principal components of the NDVI correlations,
which is just a 23-by-23 matrix of pairwise correlations between the 23 time
slices. The plot method of the output shows the variance among pixels, not at
each time slice, but on each principal component.



~~~r
pca <- princomp(covmat = ndvi_cor)
plot(pca)
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}
![ ]({% include asset.html path="images/pca/unnamed-chunk-5-1.png" %})
{:.captioned}

===

Principal component "loadings" correspond to the weight each time slice
contributes to each component.



~~~r
npc <- 4
loading <- data.frame(
  Date = rep(dates, npc), 
  PC = factor(
    rep(1:npc, each = length(dates))
  ),
  Loading = c(pca$loadings[, 1:npc])
)
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


Here we manually reshape the output of `princomp` into a data frame for plotting.
{:.notes}

===

The first principal component is a more-or-less equally weighted combination of
all time slices, like an average.

In contrast, components 2 through 4 show trends over time. PC2 has a broad downward trend
centered around July 2005, while PC3 has a sharp downward trend centered around 
April 2005.
{:.notes}



~~~r
ggplot(loading,
       aes(x = Date, y = Loading,
           col = PC)) +
  geom_line()
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}
![ ]({% include asset.html path="images/pca/unnamed-chunk-7-1.png" %})
{:.captioned}

===

The principal component scores are projections of the NDVI values at each time
point onto the components. Memory limitation may foil a straightforward attempt at this calculation, but the [raster](){:.rlib}
package `predict` wrapper carries the princomp `predict` method through to
the time series for each pixel.



~~~r
pca$center <- pca$scale * 0
ndvi_scores <- predict(
  ndvi_stdz, pca,
  index = 1:npc,
  filename = file.path(out, 'ndvi_scores.grd'),
  overwrite = TRUE)
plot(ndvi_scores)
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}
![ ]({% include asset.html path="images/pca/unnamed-chunk-8-1.png" %})
{:.captioned}

A complication in here is that the `pca` object does not know how the original
data were centered, because we didn't give it the original data. The `predict`
function will behave as if we performed PCA on `ndvi_stdz[]` if we set the
centering vector to zeros. We specify `index = 1:npc` to restrict the 
prediction to the first four principal component axes. 
This returns a standardized value for each pixel on each of the four axes.
{:.notes}

===

The first several principal components account for most of the variance in the
data, so approximate the NDVI time series by "un-projecting" the scores.

Mathematically, the calculation for this approximation at each time slice,
$$\mathbf{X_t}$$, is a linear combination of each score "map", $$\mathbf{T}_i$$, with
time-varying loadings, $$W_{i,t}$$.
{:.notes}

$$
\mathbf{X}_t \approx W_{1,t} \mathbf{T}_1 + W_{2,t} \mathbf{T}_2 + W_{3,t} \mathbf{T}_3 + \dots
$$

===

The flexible `overlay` function allows you to pass a custom function for
pixel-wise calculations on one or more of the main raster objects.



~~~r
ndvi_dev <- overlay(
  ndvi_stdz, ndvi_scores,
  fun = function(x, y) {
    x - y %*% t(pca$loadings[, 1:npc])
  },
  filename = file.path(out, 'ndvi_dev.grd'),
  overwrite = TRUE)
names(ndvi_dev) <- names(ndvi)
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


Here we define a function `fun` that takes two arguments, `x` and `y`. Those
are the first two arguments we pass to `overlay`. The `overlay` function then 
"overlays" raster brick `x`, the standardized NDVI time series with 23 layers, on raster brick `y`,
the first 4 PCA scores for that time series, and applies `fun` to each pixel 
of those two raster bricks. `fun` performs matrix multiplication of one pixel of `y` with
the first 4 PCA loadings and then subtracts the result from `x`, resulting in the 
difference between the observed standardized pixel values and the values 
predicted by the principal components.
{:.notes}

===

Verify that the deviations just calculated are never very large,
(that is, the PCA predicts the true values fairly well),
then try the same approximation using even fewer principal components.



~~~r
> animate(ndvi_dev, pause = 0.5, n = 1)
~~~
{:title="Console" .no-eval .input}


![plot of ndvi_dev_animation]({% include asset.html path="images/ndvi_dev_animation.gif" %})
{:.captioned}

===

Based on the time variation in the loadings for principal components 2 and 3,
as we saw in the graph of loadings versus time we made earlier, we
might guess that they correspond to one longer-term and one shorter-term
departure from the seasonal NDVI variation within this extent.



~~~r
plot(
  ndvi_scores[[2]] < -2 |
  ndvi_scores[[3]] < -2)
plot(st_geometry(scar), add = TRUE)
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}
![ ]({% include asset.html path="images/pca/unnamed-chunk-12-1.png" %})
{:.captioned}
