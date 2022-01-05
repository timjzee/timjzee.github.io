---
layout: post
title:  "Installing RStan on 2021 Macbook Pro running macOS Monterey"
date:   2022-1-5 15:00:00 +0100
categories: rstan macos apple m1
---

> Disclaimer: I claim no deep understanding of how this all works. I just know that it works, and that I could not find any resource that described each step.

This description assumes a clean install of Monterey (no previous install of rstan or the C++ Toolchain for rstan).
It borrows extensively from [https://thecoatlessprofessor.com/programming/cpp/r-compiler-tools-for-rcpp-on-macos/](https://thecoatlessprofessor.com/programming/cpp/r-compiler-tools-for-rcpp-on-macos/)
If you do have a previous install of the C++ Toolchain for rstan, see [this post](https://thecoatlessprofessor.com/programming/r/uninstalling-the-r-development-toolchain-on-macos/) to uninstall relevant files.

## Install Command Line Tools
- open Terminal (located in Applications/Utilities)
- type `gcc`
- you will get a prompt to install Command Line Tools if you don't have them

## Install the latest version of R:

First install Xquartz
- download the installer on [https://www.xquartz.org](https://www.xquartz.org)
- follow the instructions

Then install R:
- go to [https://cran.r-project.org/bin/macosx/](https://cran.r-project.org/bin/macosx/)
- make sure to download the Apple silicon arm64 version "R-4.1.2-arm64.pkg"
- follow the instructions

Note that it says "for macOS 11 (Big Sur) and higher", this is something we need to take into account during the configuration of the C++ toolchain.

## Configuring C++ Toolchain for rstan

### Install gfortran for macOS Big Sur:

We need the gfortran version for Big Sur (not the one for Monterey), because the installed R version was compiled for Big Sur.

- go to [https://github.com/fxcoudert/gfortran-for-macOS/releases/tag/11-arm-alpha2](https://github.com/fxcoudert/gfortran-for-macOS/releases/tag/11-arm-alpha2)
- download the "gfortran-ARM-11.0-BigSur.pkg" installer
- follow the instructions

This installs gfortran in `/usr/local/`. However, R needs it to be in `/opt/R/arm64/`. So we just make a symlink to `/usr/local/gfortran` in `/opt/R/arm64/`.

- open a Terminal window
- type `sudo ln -s /usr/local/gfortran /opt/R/arm64/gfortran` and enter your admin password

### Check if you can compile C++ code from R

Borrowed from: [https://thecoatlessprofessor.com/programming/cpp/r-compiler-tools-for-rcpp-on-macos/](https://thecoatlessprofessor.com/programming/cpp/r-compiler-tools-for-rcpp-on-macos/)

First, install Rcpp and RcppArmadillo within R.

```
install.packages(c('Rcpp', 'RcppArmadillo'))
```

Create a new file and name it: helloworld.cpp

By adding the .cpp extension, the file is interpreted as C++ code.

Within the file write:

```
#include <RcppArmadillo.h>

// [[Rcpp::depends(RcppArmadillo)]]

// [[Rcpp::export]]
void hello_world() {
  Rcpp::Rcout << "Hello World!" << std::endl;
}

// After compile, this function will be immediately called using
// the below snippet and results will be sent to the R console.

/*** R
hello_world()
*/
```

Then in the R console type:

```
Rcpp::sourceCpp('path/to/file/helloworld.cpp')
```

where `path/to/file/` is the location containing `helloworld.cpp`

If everything is installed appropriately, then you should see the following in the console:

```
> hello_world()

Hello World!
```

### Optimize compiler (optional)

Borrowed from [https://github.com/stan-dev/rstan/wiki/Configuring-C---Toolchain-for-Mac](https://github.com/stan-dev/rstan/wiki/Configuring-C---Toolchain-for-Mac)
Optionally you can optimize the compiler by running the following lines in R:

```
dotR <- file.path(Sys.getenv("HOME"), ".R")
if (!file.exists(dotR)) dir.create(dotR)
M <- file.path(dotR, "Makevars")
if (!file.exists(M)) file.create(M)
arch <- ifelse(R.version$arch == "aarch64", "arm64e", "x86_64")
cat(paste("\nCXX14FLAGS += -O3 -mtune=native -arch", arch, "-ftemplate-depth-256"),
    file = M, sep = "\n", append = FALSE)
```

## Install rstan

Run the following lines in R.

```
Sys.setenv(DOWNLOAD_STATIC_LIBV8 = 1)
install.packages("rstan", repos = "https://cloud.r-project.org/", dependencies = TRUE)
```

To verify your installation, you can run the rstan example/test model:

```
example(stan_model, package = "rstan", run.dontrun = TRUE)
```

If the chains run without errors, everything worked!
