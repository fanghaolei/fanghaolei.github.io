# Organizing Projects with S3 Methods in R 

Unless you are developing your own R packages with complicated structures, object oriented programming in R is often overlooked.  Not until very recently, I started using OO programming, or more specifically speaking -- S3 methods, in situations other than package development. 

If you aren't familiar with S3 methods in R, here is a guide by Hadley: [OO field guide in Advanced R](http://adv-r.had.co.nz/OO-essentials.html).

So, let's cut into the point. 

One big challenge in data science is actually organizing and reusing the code across multiple projects. Imaging that you have a general data wrangling procedure that loosely applies to all your projects but with some custom variants.

Obviously, one way to reuse your code is simply by copying and pasting functions or scripts that are relatively generic from one project to another and then refactor accordingly. This is a very quick and effective way to get the job done; however, you will get confused over time with code accumulation and puzzled by the subtle differences in each project. Also if you found a bug in one place, you have to fix it in multiple places, which is quite a pain. 

 The S3 approach to handle a situation like this is fairly straight forward:

1. make an s3 object that defines your project, note that an s3 object is simply list with attributes:

   ```R
   create_project <- function(project_name, options){
       project <- structure(
           list(
               project_name = project_name,
               
               # Note that you can wrap all project 
               # specific params into an options list 
               # and unpack it later in the S3 methods.
           	options = options
           ),
      
           # class name is simply the project_name, you 
           # can also assign additonal parent classes to
           # set up an inheritance structure, which is 
           # particularly useful if you have layers of 
           # projects that follow one generic procedure
       
           # E.g., class = c(project_name, "my_project")
           # all projects are also "my_project"
           class = project_name
       )
       return(project)
   }
   ```

2. create a generic function that calls project specific s3 methods:

   ```R
   process_data <- function(obj, data){
       UseMethod("process_data", obj)
   }
   ```

   you may also want a default method that applies to all projects:

   ```R
   process_data.default <- function(obj, data){
       # do something that applies to every project
       return(data)
   }
   ```

   or, if you have a parent class defined in 1, you can also do:

   ```R
   process_data.my_project  <- function(obj, data){
       # do something that applies to all "my project"
       return(data)
   }
   ```

3. create project specific s3 methods:

   ```R
   # For project alpha
   process_data.project_alpha <- function(obj, data){
       # unpack options list from obj
       options <- obj$options
       ...
       
       # do something that applies to project alpha
       ...
       
       # either call the default method using 
       # NextMethod or simply return the data
       NextMethod("process_data")
   }
   ```

   ```R
   # For project beta (same thing)
   process_data.project_beta <- function(obj, data){
       # unpack options list from obj
       options <- obj$options
       ...
       
       # do something that applies to project beta
       ...
       
       # either call the default method using 
       # NextMethod or simply return the data
       NextMethod("process_data")
   }
   ```

4. then wrap up your code in an execution function / script and run:

   ```R
   run_data_process <- function(name, options,data){
       
       project <- create_project(name, options)
   	processed_data <- process_data(
           project, 
           data = data
       )
       return(processed_data)
   }
   
   # execute!
   alpha_processed <- run_data_process(
       "alpha", list(alpha = "good"), 
       alpha_data
   )
   beta_processed <- run_data_process(
       "beta", list(beta = "better"), 
       beta_data
   )
   ```

   You can see that with the S3 methods, we have the ability to scale our core functions to many projects with following advantages:

   1. No more worries about human error or hidden bugs from copy & paste.  

   2. Easy bug fixes. If there's a bug in the core generic function, fixing it in one place means all projects are also fixed. 

   3. Flexibility to extend your object with even more methods. This maximizes your code reusability. 








