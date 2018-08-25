# A Clean Framework to Interact with Rstan 

[Stan](http://mc-stan.org/) is a fantastic language for statistical computing with its extraordinary ability in performing probabilistic modeling and bayesian data analysis 

Here I want to talk about a clean way I discovered to interact with stan, essentially, the way to build a clean, easy-to-use framework to avoid code duplication and reuse your code as much as possible. Even though the title says Rstan, but I think the general structure applies to Pystan as well since they share a pretty similar API.

## 1. Where to store the stan code?

This is the straight-up problem we are facing. Where to put all our fancy stan model code? Since raw stan code needs to be compiled to run, we definitely want to avoid the time consuming re-compiling process. Depending on whether you want to put all your code into an R package, this can be achieved in two different ways. 

1. The project is in a single repo or too small to require a full-blown package. 

   In this scenario, you want put all your `.stan` code into one dedicated directory, when your model runs, the compiled binaries will be saved in the same directory. Of course you will need a function to save the binaries as `.rds` files, I will talk about that in the following section. 

2. The project has a relative large code base.

   Now you should be considering a package. Having a stand-out package repo is very helpful in separating modeling code and other helpers you created for processing or exploring data. From my experience, the core modeling code will need far less maintenance or be much easier to maintain once your model is mature. In this case, it's cleaner to store all `.stan` code under an `inst/` directory in the package repo. Similarly this requires a few helpers to optimize the workflow. 

## 2. Build around the stan API - Stan helpers. 

From reading stan's API manual, we can see that the ultimate function we call is `sampling()`, and the key arguments are:

1. object: a compiled stan model. 
2. data: a named list (dict in python) providing all input specified in your stan model
3. pars: parameters to keep in the stan fit
4. chains, iter, warmup, thin, seed: these are all core options for the sampler
5. algorithms: the sampling method, I think most of the case we use 'NUTS'

So, what we need are pretty obvious. 

1.   A pre-compiled model. The `stan_model()` function in `rstan` package does exactly that. However, we also want to be able to reuse our compiled binaries, so we need a wrapper to `stan_model`, something like this:

     ```
     compile_save_model <- function(stan_code) {
         binary <- stan_model(stan_code)
         saveRDS(binary, binary_file_dir)
     }
     ```

      to not only compile but also save the binary into a `.rds` or `.RData` file. In python, this can be a `.pkl` file. We also need a `load_compiled_model()` helper to load the saved binary. In R this can be a simple wrapper to `readRDS()` or `read()` with your file directory. 

2.   A data list. We need a helper function 

3.   A model option list. 

4.   A wrapper 

## 3. A Object Oriented Model Class Design.

