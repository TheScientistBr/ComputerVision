MXNET
-----

Apache MXNet is an effort undergoing incubation at The Apache Software Foundation (ASF), sponsored by the Apache Incubator. Incubation is required of all newly accepted projects until a further review indicates that the infrastructure, communications, and decision making process have stabilized in a manner consistent with other successful ASF projects. While incubation status is not necessarily a reflection of the completeness or stability of the code, it does indicate that the project has yet to be fully endorsed by the ASF.

[source](https://mxnet.incubator.apache.org/versions/master/)

### Classify Images with a PreTrained Model

MXNet is a flexible and efficient deep learning framework. One of the interesting things that a deep learning algorithm can do is classify real world images.

In this tutorial, we show how to use a pre-trained Inception-BatchNorm network to predict the class of an image. For information about the network architecture.

The pre-trained Inception-BatchNorm network is able to be downloaded from this link This model gives the recent state-of-art prediction accuracy on image net dataset.

``` r
require(mxnet)
```

    ## Loading required package: mxnet

    ## Warning: package 'mxnet' was built under R version 3.4.4

### Including Plots

Now load the imager package to load and preprocess the images in R:

``` r
require(imager)
```

    ## Loading required package: imager

    ## Warning: package 'imager' was built under R version 3.4.4

    ## Loading required package: magrittr

    ## Warning: package 'magrittr' was built under R version 3.4.4

    ## 
    ## Attaching package: 'imager'

    ## The following object is masked from 'package:magrittr':
    ## 
    ##     add

    ## The following objects are masked from 'package:stats':
    ## 
    ##     convolve, spectrum

    ## The following object is masked from 'package:graphics':
    ## 
    ##     frame

    ## The following object is masked from 'package:base':
    ## 
    ##     save.image

Make sure you unzip the pre-trained model in the current folder. Use the model loading function to load the model into R:

``` r
model = mx.model.load("Inception/Inception_BN", iteration=39)
```

Load in the mean image, which is used for preprocessing using:

``` r
mean.img = as.array(mx.nd.load("Inception/mean_224.nd")[["mean_img"]])
```

### Load and Preprocess the Image

Now, we are ready to classify a real image. In this example, we simply take the parrots image from the imager package. You can use another image, if you prefer.

Load and plot the image:

``` r
im <- load.image("delermando4.jpg")
plot(im)
```

![](README_files/figure-markdown_github/load_image-1.png)

Before feeding the image to the deep network, we need to perform some preprocessing to make the image meet the deep network input requirements. Preprocessing includes cropping and subtracting the mean. Because MXNet is deeply integrated with R, we can do all the processing in an R function:

``` r
preproc.image <- function(im, mean.image) {
        # crop the image
        shape <- dim(im)
        short.edge <- min(shape[1:2])
        xx <- floor((shape[1] - short.edge) / 2)
        yy <- floor((shape[2] - short.edge) / 2)
        cropped <- crop.borders(im, xx, yy)
        # resize to 224 x 224, needed by input of the model.
        resized <- resize(cropped, 224, 224)
        # convert to array (x, y, channel)
        arr <- as.array(resized) * 255
        dim(arr) <- c(224, 224, 3)
        # subtract the mean
        normed <- arr - mean.img
        # Reshape to format needed by mxnet (width, height, channel, num)
        dim(normed) <- c(224, 224, 3, 1)
        return(normed)
}
```

Use the defined preprocessing function to get the normalized image:

``` r
normed <- preproc.image(im, mean.img)
```

### Classify the Image

Now we are ready to classify the image! Use the predict function to get the probability over classes:

``` r
prob <- predict(model, X=normed)
```

Use the max.col on the transpose of prob to get the class index:

``` r
max.idx <- max.col(t(prob))
```

The index doesn't make much sense, so let's see what it really means. Read the names of the classes from the following file:

``` r
synsets <- readLines("Inception/synset.txt")
```

Let's see what the image really is:

``` r
print(paste("Predicted Top-class:", synsets[[max.idx]]))
```

    ## [1] "Predicted Top-class: n02870880 bookcase"

### Trying again with others pictures - Coffee mug

``` r
im <- load.image("caneca.jpg")
plot(im)
```

![](README_files/figure-markdown_github/coffee-1.png)

``` r
normed <- preproc.image(im, mean.img)

prob <- predict(model, X=normed)
max.idx <- max.col(t(prob))
synsets <- readLines("Inception/synset.txt")
print(paste0("Predicted Top-class: ", synsets  [[max.idx]]))
```

    ## [1] "Predicted Top-class: n03063599 coffee mug"

### Trying again with others pictures - Book

``` r
im <- load.image("chave.jpg")
normed <- preproc.image(im, mean.img)
plot(im)
```

![](README_files/figure-markdown_github/error-1.png)

``` r
prob <- predict(model, X=normed)
max.idx <- max.col(t(prob))
synsets <- readLines("Inception/synset.txt")
print(paste0("Predicted Top-class: ", synsets  [[max.idx]]))
```

    ## [1] "Predicted Top-class: n03109150 corkscrew, bottle screw"

Not so good!
