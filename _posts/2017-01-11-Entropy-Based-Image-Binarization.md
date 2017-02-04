---
layout: post
title: Entropy Based Image Binarization with imager and FSelectorRcpp
comments: true
published: true
date: 2017-01-08 11:00:00
author: Marcin Kosiński
categories: [Image Processing, Entropy]
output:
  html_document:
    mathjax:  default
    fig_caption:  true
    toc: true
    section_numbering: true
    keep_md: true
---

<img src="/images/fulls/binarization.png" class="fit image"> The image processing and the computer vision have gained a significant interest in last 2 decades. The image analysis can be used to detect items or people on images and videos. It is widely used in the medicine to detect cancer tissues and to improve brain, lungs and heart diseases diagnostic. The computer automation enabled analyzing terabytes of an image data, based on which we improve our life status and get insights for business decisions. In this post I present basic operations that can be applied to a simple image, all thanks to [imager](https://cran.r-project.org/web/packages/imager/index.html) package by which I am truly impressed. I also present a quick entropy approach to the image binarization, which applied to images on a greyscale transforms them to the binarized black-and-white output.


*Warning: you will probably not be able to bear my face anymore after this post.*

* imager
* Basic image operations
* Entropy based image binarization
{:toc}


# imager

Image processing library based on `CImg` - this is the title of `imager` package on [CRAN](https://cran.r-project.org/web/packages/imager/index.html). From the package's overview, it's [website](http://dahtah.github.io/imager/) and a GREAT [vignette: getting started with imager](https://cran.r-project.org/web/packages/imager/vignettes/gettingstarted.html) we can learn that it

> contains a large array of functions for working with image data, with most of these functions coming from the CImg library by David Tschumperle.

[CImg](http://cimg.eu) is a simple, modern C++ library for image processing - `imager` is a glue between the R interface and this library that works under the hood.

I must admit I read a lot of R vignettes, but I haven't earlier seen such an accurate and comprehensive one before! Maybe [RSelenium](https://cran.r-project.org/web/packages/RSelenium/) vignettes (about which I wrote in my previous post - [Controlling Expenses on Ali Express with RSelenium](http://r-addict.com/2017/01/08/RSelenium-at-TriCity-and-AliExpress.html)) are written with as high sacrifice as those from `imager`.

# Basic image operations

Package provides a basic functionality allowing to load an image to R and to plot it with


{% highlight r %}
library(imager)
my_photo <- load.image('Z1PTg1t1_400x400.jpg')
layout(t(c(1,2)))
plot(my_photo, main = "My visa photo")
plot(grayscale(my_photo), main = "My photo in a grayscale")
{% endhighlight %}

![plot of chunk unnamed-chunk-2](/figure/source/2017-01-11-Entropy-Based-Image-Binarization/unnamed-chunk-2-1.png)

Photos are stored as `cimg` objects and are a special representation of a 4D array with dimensions called: x,y,z,c where x,y corresponds to spatial dimensions, z usually correspond to depth or time, and c is a colour. The third dimension is typically used for videos. If you are operating on a photo from a grayscale you will not need to pay the attention to the last 2 dimensions.


{% highlight r %}
class(my_photo)
{% endhighlight %}



{% highlight text %}
[1] "cimg"
{% endhighlight %}



{% highlight r %}
my_photo
{% endhighlight %}



{% highlight text %}
Image. Width: 400 pix Height: 400 pix Depth: 1 Colour channels: 3 
{% endhighlight %}



{% highlight r %}
grayscale(my_photo)
{% endhighlight %}



{% highlight text %}
Image. Width: 400 pix Height: 400 pix Depth: 1 Colour channels: 1 
{% endhighlight %}

The advantage of storing images as arrays is that we can simply extract values corresponding to pixels to perform operations like arythemtic transformations or plotting the histogram of pixels' intensity.


{% highlight r %}
plot(sqrt(my_photo)+1); plot(log(grayscale(my_photo)+2)*0.95, rescale = FALSE)
{% endhighlight %}

![plot of chunk unnamed-chunk-4](/figure/source/2017-01-11-Entropy-Based-Image-Binarization/unnamed-chunk-4-1.png)

The `rescale` parameter stands for whether you would like to store image in `0-255` or `[0,1]` range, where for `0-255` picture are always rescaled to the maximum range. So to visualize the linear operations you need to set `rescale=FALSE`.


{% highlight r %}
layout(t(c(1,2,3)))
plot(my_photo)
plot(my_photo/2) # nothing happens on a plot 
plot(my_photo/2, rescale = FALSE)
{% endhighlight %}

![plot of chunk unnamed-chunk-5](/figure/source/2017-01-11-Entropy-Based-Image-Binarization/unnamed-chunk-5-1.png)

> Note the y axis running downwards: the origin is at the top-left corner, which is the traditional coordinate system for images. imager uses this coordinate system consistently

## Image histogram and it's equalisation

From [wikipedia](https://en.wikipedia.org/wiki/Image_histogram)

> An image histogram is a type of histogram that acts as a graphical representation of the tonal distribution in a digital image. It plots the number of pixels for each tonal value. By looking at the histogram for a specific image a viewer will be able to judge the entire tonal distribution at a glance.

It can be simply plotted for a grayscaled image with:


{% highlight r %}
layout(t(1))
# %>% operator means for
# f(a) %>% g(par1, par2) == g(f(a), par1, par2)
# and simplifies pipelines
grayscale(my_photo) %>% hist(main="Luminance values \nin my photo")
{% endhighlight %}

![plot of chunk unnamed-chunk-6](/figure/source/2017-01-11-Entropy-Based-Image-Binarization/unnamed-chunk-6-1.png)

Another approach is to turn the image into a data.frame, and use [ggplot2](https://github.com/tidyverse/ggplot2) to view all colour channels at once:


{% highlight r %}
library(ggplot2)
mdf <- as.data.frame(my_photo)
head(mdf, 3)
{% endhighlight %}



{% highlight text %}
  x y cc     value
1 1 1  1 0.9019608
2 2 1  1 0.9019608
3 3 1  1 0.9019608
{% endhighlight %}



{% highlight r %}
mdf <- plyr::mutate(mdf, channel = factor(cc, labels = c('R', 'G', 'B')))
ggplot(mdf, aes(value, col = channel)) +
   geom_histogram(bins = 30) + 
   facet_wrap(~ channel)
{% endhighlight %}

![plot of chunk unnamed-chunk-7](/figure/source/2017-01-11-Entropy-Based-Image-Binarization/unnamed-chunk-7-1.png)

Histogram equalisation makes histograms flat: each pixel's value is replaced by its rank, which is equivalent to running the data through their empirical cdf, which mainly solves the problem of over-representation for some pixel intensities.


{% highlight r %}
f <- ecdf(grayscale(my_photo))
f(grayscale(my_photo)) %>% hist(main="Transformed luminance values")
{% endhighlight %}

![plot of chunk unnamed-chunk-8](/figure/source/2017-01-11-Entropy-Based-Image-Binarization/unnamed-chunk-8-1.png)

Check the below comparison of an original photo, a grayscaled photo and a grayscaled photo after the histogram equalisation.


{% highlight r %}
layout(t(1:3))
plot(my_photo, main = "Original photo")
plot(grayscale(my_photo), main = "In a grayscale")
f(grayscale(my_photo)) %>% 
   as.cimg(dim=dim(grayscale(my_photo))) %>% 
   plot(main="With histogram equalisation")
{% endhighlight %}

![plot of chunk unnamed-chunk-9](/figure/source/2017-01-11-Entropy-Based-Image-Binarization/unnamed-chunk-9-1.png)

## Plotting with ggplot2

By converting `cimg` object to a `data.frame` with `as.data.frame` (mdf object) we can use following format to plot the image with `ggplot2`. See following code with comments:


{% highlight r %}
f <- ecdf(grayscale(my_photo))
mdf <- f(grayscale(my_photo)) %>% 
         as.cimg(dim=dim(grayscale(my_photo))) %>%
         as.data.frame()
head(mdf, 4)
{% endhighlight %}



{% highlight text %}
  x y     value
1 1 1 0.9769000
2 2 1 0.9769000
3 3 1 0.9769000
4 4 1 0.9566875
{% endhighlight %}

Note: I am using `as.cimg` since the result of `f` function is a regular vector, that needs to be converted  to `cimg` object for further usage.


{% highlight r %}
ggplot(mdf, aes(x, y)) + # create ggplot object
   geom_raster(aes(fill = value)) + # use raster geometry 
   scale_x_continuous(expand = c(0,0)) + # remove margins on X axis
   scale_y_continuous(expand = c(0,0), # remove margins on Y axis
      trans = scales::reverse_trans()) + # reverse Y axis
   scale_fill_gradient(low = "black", high = "white") # scale in grayscale - the default scale for ggplot is blue
{% endhighlight %}

![plot of chunk unnamed-chunk-11](/figure/source/2017-01-11-Entropy-Based-Image-Binarization/unnamed-chunk-11-1.png)


## Treshold binarization

For such a transformed image (a grayscaled photo after the histogram equalisation) next natural step in the image analysis is performing the image binarization. This is helpful in detecting the image's background and foreground. In this process we will substitute pixel values with 0 and 1 so that in the end the output image will be only black and white. This format is also convenient for detecting edges or blob extractions.

With `imager` we can perform the threshold binarization with `treshold()` function that can provide a threshold with quantiles or with numeric values (or somehow automatic). Below is only example which is an introduction for the next chapter: `entropy based image binarization`.


{% highlight r %}
my_photo_he <- f(grayscale(my_photo)) %>% 
         as.cimg(dim=dim(grayscale(my_photo)))
layout(t(matrix(1:6, ncol = 2, nrow = 3)))
threshold(my_photo_he, thr = 0.2) %>% plot(main = "Determinant: \nover 0.2 value")
threshold(my_photo_he, thr = 0.3) %>% plot(main = "Determinant: \nover 0.3 value")
threshold(my_photo_he, thr = 0.4) %>% plot(main = "Determinant: \nover 0.4 value")
threshold(my_photo_he, thr = "25%") %>% plot(main = "Determinant: \n75% highest values")
threshold(my_photo_he, thr = "50%") %>% plot(main = "Determinant: \n50% highest values")
threshold(my_photo_he, thr = "auto") %>% plot(main = "auto")
{% endhighlight %}

![plot of chunk unnamed-chunk-12](/figure/source/2017-01-11-Entropy-Based-Image-Binarization/unnamed-chunk-12-1.png)

> thr	- a threshold, either numeric, or "auto", or a string for quantiles

# Entropy based image binarization 

## FSelectorRcpp

Last year Zygmunt Zawadzki ([zzawadz](http://github.com/zzawadz)) developed [FSelectorRcpp](http://mi2-warsaw.github.io/FSelectorRcpp/) package (still only available at [GitHub](http://github.com/mi2-warsaw/FSelectorRcpp) - as I am still testing it, mea culpa...) which is a Rcpp (free of Java/Weka) implementation of [FSelector](https://cran.r-project.org/web/packages/FSelector/index.html) entropy-based feature selection algorithms with sparse matrix support. It has a functionality allowing to calculate the conditional entropy for a variable, knowing values of another feature. This is a good measure of how one variable explains another one and can be used in the feature selection to reduce `the curse of a dimensionality` or can be used in selecting the threshold value in the image binarization (my contrived idea).


{% highlight r %}
# devtools::install_github('mi2-warsaw/FSelectorRcpp')
library(FSelectorRcpp)
{% endhighlight %}

## Entropy and information gain

For the binarization I would choose the threshold that maximizes the information gain (based on the entropy) of binarized images based on the information gathered from the image before the binarization (a grayscaled photo after the histogram equalisation).

The information gain is defined as 

$$H(binarized) + H(pre_binarized) - H(binarized, pre_binarized),$$

where $$H(X)$$ states for the Shannon's entropy:

$$H(X) = - \sum_{i=1}^{n} P(x_i)\log_bP(x_i),$$

where $$b$$ is the base of the logarithm used (common values of $$b$$ are 2), and $$X(A,B)$$ is the conditional entropy for $$A$$ with a condition to $$B$$.

See an example of an information gain extraction for the binarized image with the threshold = 0.1.

{% highlight r %}
my_photo_for_entropy <- data.frame(
   pixels = as.data.frame(my_photo_he)[, 3],
   binarized = as.factor(as.data.frame(threshold(my_photo_he, thr = 0.1))[, 3]))
tail(my_photo_for_entropy, 3)
{% endhighlight %}



{% highlight text %}
          pixels binarized
159998 0.5980688         1
159999 0.5980688         1
160000 0.5980688         1
{% endhighlight %}



{% highlight r %}
information_gain(formula = binarized ~ pixels, 
                 data = my_photo_for_entropy)
{% endhighlight %}



{% highlight text %}
       importance
pixels   0.324038
{% endhighlight %}

## Final binarization - threshold selection

The final step is to check all the possible thresholds (let's assume that those are values 0-255 divided by 255). Since every threshold can be checked independently, I am using `parallel` library and `mclapply` function to apply the procedure to multiple cores.


{% highlight r %}
library(parallel)
mclapply(0:255/255, function(thresh){
   my_photo_for_entropy <- data.frame(
      pixels = as.data.frame(my_photo_he)[, 3],
      binarized = as.factor(as.data.frame(threshold(my_photo_he, thr = thresh))[, 3]))
   information_gain(formula = binarized ~ pixels, 
                    data = my_photo_for_entropy)[1,1] 
}) -> info_gains
{% endhighlight %}

Then I present the plot of calculated information gain vs the specified threshold. It's surprising for me that this is almost symmetric over 0.5. Maybe this is due to the histogram equalisation. In my opinion this is the great tool to detect the background and the foreground of the image.


{% highlight r %}
library(ggthemr)
ggthemr("fresh")
ggplot(data.frame(threshold = 0:255/255, 
                  information_gain = unlist(info_gains)),
       aes(threshold, information_gain)) +
   geom_line() +
   labs(title = "Entropy Based Image Binarization \nwith imager and FSelectorRcpp",
        caption = "code: r-addict.com")
{% endhighlight %}

![plot of chunk unnamed-chunk-16](/figure/source/2017-01-11-Entropy-Based-Image-Binarization/unnamed-chunk-16-1.png)

To sum up, I present my photo in a grayscale after the histogram equalisation and after the binarization based on the information gain optimization.


{% highlight r %}
layout(t(1:2))
plot(my_photo_he, main = "My photo in a grayscale \nafter the histogram equalisation")
threshold(my_photo_he, 0.5) %>%
   plot(main = "The binarized photo \nbased on the entropy optimization")
{% endhighlight %}

![plot of chunk unnamed-chunk-17](/figure/source/2017-01-11-Entropy-Based-Image-Binarization/unnamed-chunk-17-1.png)

