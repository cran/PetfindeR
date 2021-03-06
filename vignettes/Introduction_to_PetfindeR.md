---
title: "Introduction to PetfindeR"
output: rmarkdown::html_vignette
vignette: >
  %\VignetteIndexEntry{Introduction to PetfindeR}
  %\VignetteEngine{knitr::rmarkdown}
  %\VignetteEncoding{UTF-8}
---

The following vignette introduces the `PetfindeR` package and its methods for interacting with the Petfinder API. The goal of the `PetfindeR` library is to enable other users to interact with the rich data available in the Petfinder database. More information on the Petfinder API itself can be found on the [API documentation page](https://www.petfinder.com/developers/v2/docs/).

# Table of Contents

* [Obtaining an API and Secret key](#api_key)
* [Installation](#installation)
    - [Animal Types](#types)
    - [Extracting breeds of animals available in the Petfinder database](#breeds)
    - [Finding animals](#animals)
    - [Finding shelters matching search criteria](#organizations)
* [Conclusion](#conclusion)

# Obtaining an API and Secret Key <a id='api_key'></a>

Before we can begin extracting data from the API, we first require an API and secret key to authenticate access. To receive an API and secret key, [create a free account with Petfinder](https://www.petfinder.com/developers/) on their developer page and request an API key.

The API and secret key received from Petfinder are what we will use to authenticate our connection to the Petfinder API with `PetfindeR`. Note authenication has a timeout of 3600 seconds, or one hour, after which the authentication to the API will need to be made again. For more information on how the Petfinder API authenication works, [visit the API documentation on the Petfinder developer page](https://www.petfinder.com/developers/v2/docs/#using-the-api).

Storing your keys received from APIs and other sensitive information in a secure file or as an environment variable is considered best practice to avoid any potential malicious activity. Therefore, we save the API and secret keys we received from Petfinder as environment variables in a specific R file named `.Renviron`. If you do not have an `.Renviron` file on your system, here is handy and short guide for creating one and adding environment variables for both Linux and Windows. If on a Mac, you can add the `.Renviron` file in your home directory. After saving the variables and restarting R/RStudio, we can access the environment variables with the [`Sys.getenv()`](https://stat.ethz.ch/R-manual/R-devel/library/base/html/Sys.getenv.html) function.


```r
key <- Sys.getenv('PETFINDER_KEY')
secret <- Sys.getenv('PETFINDER_SECRET_KEY')
```

# Installation <a id='installation'></a>

If not already installed, `PetfindeR` can be installed through the usual means:


```r
install.packages('PetfindeR')
```

The package can also be installed with [`devtools`](https://cran.r-project.org/package=devtools) for those wanting the most recent development version.


```r
install.packages('devtools') # if devtools is not already installed
devtools::install_github('aschleg/PetfindeR')
```

Once installed, we can load the library as usual:


```r
library(PetfindeR)
```

Now that `PetfindeR` is loaded, we can authenticate our connection to the API and begin extracting data! The authentication to the Petfinder API occurs when the `PetfindeR` class is initialized, which requires the API and secret keys we received in the previous step as parameters.


```r
pf = Petfinder(key = key, secret = secret)
```

The `pf` variable is the initialized Petfinder class with our given API and secret key. We can now use this instance to interact with and extract data from the Petfinder API.

# Getting Animal Types <a id='types'></a>

The `animal_types` method allows one to get information on all or specific animal types in the Petfinder database. Animal type data includes the type's species, color, coat and gender. Leaving the `types` parameter of the method empty will return all available animal types.


```r
animal_types = pf$animal_types()
```

We can get the names of all the animal types available in the Petfinder API by using the [`names()`](https://stat.ethz.ch/R-manual/R-devel/library/base/html/names.html) function


```r
names(animal_types)
```

Specific information for a given type can be accessed as usual:


```r
print(animal_types$Cat$coats)
print(animal_types$Cat$colors)
```

The `animal_types` method also accepts a single or multiple animal types.


```r
cats = pf$animal_types('cat')
```

We can access the type's data as we did before:


```r
print(cats$Cat$coats)
print(cats$Cat$colors)
```

Multiple specific animal types can also be passed.


```r
cat_dog <- pf$animal_types(c('cat', 'dog'))
print(cat_dog$Dog$coats)
print(cat_dog$Cat$coats)
```

# Finding Available Breeds of Animal Types <a id='breeds'></a>

Listed breeds for each animal type can be extracted using the `breeds()` method.


```r
all_breeds <- pf$breeds()
```

For brevity, let's print the animal types and the first three breeds of each type.


```r
for (breed in 1:length(all_breeds)) {
  print(names(all_breeds[breed]))
  print(all_breeds[breed][0:3])
}
```

Similar to the `animal_types()` method, the `breeds()` mehtod can accept a single or multiple animal types.


```r
cat_rabbit = pf$breeds(c('cat', 'rabbit'))

for (breed in 1:length(cat_rabbit)) {
  print(names(all_breeds[breed]))
  print(all_breeds[breed][0:3])
}
```

# Search for animals in the Petfinder database <a id='animals'></a>

The `animals` method can be used to perform a general search for animals based on specified criteria or returning data on specific animals based on their respective IDs. For example, let's say we want to get 100 adoptable female cats within a 10 mile distance of Seattle, WA.


```r
cats = pf$animals(animal_type = 'cat', gender = 'female', status = 'adoptable', location = 'Seattle, WA', distance = 10, results_per_page = 50, pages = 2)
```

Specific animal data can also be extracted by supplying the `animal_id` parameter in the `animals()` method. As the search by ID is a direct search, the general search queries such as `gender`, `age`, and the others are overridden. As an example, we can use some of the `animal_ids` returned from the `animals()` method. Although Petfinder `animal_ids` are integers, they can be supplied as strings.


```r
cat_ids = pf$animals(animal_id = c(animal_id1, animal_id2))
```

# Finding animal welfare organizations <a id='organizations'></a>

The `organizations()` method is similar to the `animals()` method described above but is used to perform general or specific searches by ID on animal welfare organizations listed in the Petfinder database. Again, similar to the first example above, we can find animal welfare organizations in a 50 mile radius of Seattle, WA. To see all of the possible search parameters, please see the [petpy documentation for the `organizations()` method](https://petpy.readthedocs.io/en/latest/api.html#get-animal-welfare-organization-data).


```r
wa_orgs = pf$organizations(location = 'Seattle, WA', distance = 50, sort = 'distance', pages = 4, results_per_page = 100)
```

We can also extract specific animal welfare organization data by supplying the `organizations()` method with Petfinder organization IDs. When performing a direct search for organizations, the general search parameters in the method are overridden and do not need to be supplied.


```r
wa_some_orgs = pf$organizations(organization_id = wa_orgs$page1$id[0:3])
```

# Conclusion <a id='conclusion'></a>

Hopefully this served as a useful introduction to the basic functionality of `PetfindeR` and some short examples to get started. The Petfinder database contains a wealth of adoptable animal information that is updated constantly so it is the goal of the `PetfindeR` wrapper library to make extracting and interacting with this data even easier whether you're building a web app or want to scrape the database for data analysis.
