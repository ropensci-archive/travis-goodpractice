goodpractice setup for your .travis.yml file
============================================

As part of rOpenSci onboarding/software review we run the [goodpractice][] package on each submission. This is done manually right now, then we paste in the output into the issue. 

We have been wanting to automate running `goodpractice`. One approach is integrating with Travis-CI. This repo just has example `.travis.yml` files. In another repo(s) we'll have code for automating Travis builds with `goodpractice` and reporting the output back to the issue. 

## Simple example

See [.simple-travis.yml](.simple-travis.yml)

```
language: r
sudo: false
cache: packages
dist: trusty

r_github_packages:
  - MangoTheCat/goodpractice

after_success:
  - Rscript -e 'Sys.setenv(NOT_CRAN = "true"); goodpractice::gp()'
```

## Deploy example

To deploy the output of `goodpractice` on Travis, you can use something like that in See [.deploy-travis.yml](.deploy-travis.yml), where we change the `after-success` declaration to:

```
after_success:
  - Rscript -e 'Sys.setenv(NOT_CRAN = "true"); x = goodpractice::gp(); z = capture.output(x);
    pkg = x$package[[1]]; pkgver = x$description$get_version(); dt = gsub(":|-", "_",
    format(Sys.time(), format="%y-%h-%d_%T")); cat(z, file = sprintf("ropensci_check_%s_%s_%s.txt",
    pkg, pkgver, dt), sep = "\\n");  goodpractice::export_json(x, sprintf("ropensci_check_%s_%s_%s.json",
    pkg, pkgver, dt)); x'
```

And add an `addons` declaration to deploy to Amazon S3 (or somewhere else as you like) 

```
addons:
  artifacts:
    s3_region: us-west-2
    paths:
    - $(git ls-files -o | grep -e '.txt' -e '.json' | tr "\n" ":")
```

If you deploy you'll likely need keys. You can add those in the Travis web console, or as global env vars like

```
env:
  global:
  - secure: <your secure key 1>
  - secure: <your secure key 2>
```


[goodpractice]: https://github.com/MangoTheCat/goodpractice
