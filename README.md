tfserve: Serve Tensorflow Models
================

Overview
--------

`tfserve` provides a [GoogleML](https://cloud.google.com/ml-engine/docs/prediction-overview) compatiable REST API for predictions to serve TensorFlow Models from R with ease.

<img src="tools/readme/swagger.png" width=500 />

Quick Start
-----------

For example, we can train MNIST as described by [MNIST For ML Beginners](https://tensorflow.rstudio.com/tensorflow/articles/tutorial_mnist_beginners.html) and then save using `SavedModelBuilder` and the right signature or, for conviniece, use a `tfserve` helper function as follows:

``` r
library(tfserve)
library(magrittr)

model_path <- "trained/tensorflow-mnist/1"
mnist_train_save(model_path)
```

    ## [1] "trained/tensorflow-mnist/1/saved_model.pb"

Then, we can serve this model with ease by running:

``` r
serve(model_path)
```

Instead of blocking R while running `serve()`, we can start a server with:

``` r
handle <- start_server(model_path)
```

We can make use of the `pixeldraw` HTMLWidget to manually collect a vector of pixels and pass them to the REST API from `tfserve` as follows:

``` r
devtools::install_github("javierluraschi/pixels")
library(pixels)

recognize_digit <- function() {
  pixels_bad <- get_pixels()
  httr::POST(
    url = "http://127.0.0.1:8089/api/predict_images/predict/",
    body = list(
      instances = list(
        matrix(as.double(pixels_bad), 28, 28, byrow = T) %>% as.vector()
      )
    ),
    encode = "json"
  ) %>%
    httr::content(as = "text") %>%
    jsonlite::fromJSON() %>%
    as.vector() %>%
    round(digits = 1) %>%
    as.logical() %>%
    which() - 1
}
```

Then, running `recognize_digit()` we can capture manually a few instances:

<img src="tools/readme/mnist-digits.gif" width=400 />

Finally, we close the server using it's `handle`:

``` r
stop_server(handle)
```
