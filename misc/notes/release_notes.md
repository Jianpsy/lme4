# lme4 release notes, v 1.1

2013-09-10 16:34:21; update, concerning lme4.0, 2014-03-10




## Version numbering

As [previously announced on the lme4 mailing list][announce], we will shortly be releasing a new version of `lme4`, a descendant of the previous development version `lme4Eigen`. For users who do not access any internal structures, there will be few backward-incompatible changes. We uploaded version 1.0-4 to CRAN on 9 September 2013. We hope it will appear on CRAN shortly, but it will initially be posted *only for the development version of R*, to allow downstream package maintainers to upload forward-compatible versions of their packages before the package is released for R 3.0.1. 
In the mean time we've learned that a new CRAN policy does not allow the `lme4.0` version of the package to be hosted on CRAN. Instead we will host it separately in order to provide reproducible research and data analysis of previous work,
see [lme4's README](https://github.com/lme4/lme4/blob/master/README.md#installation-of-lme40) for installation.

[announce]: https://stat.ethz.ch/pipermail/r-sig-mixed-models/2012q1/014811.html

* the version of `lme4` currently on [https://github.com/lme4/lme4/](github) should be used for new projects if possible
 * the most recent version can be installed (from source: development toolchain including compiler etc. are required, and you may have to install `RcppEigen` from CRAN first) via 

```r
library("devtools")
install_github("lme4", user = "lme4")
```

This will install the `master` branch, version 1.1-x; if you want the release (1.0-x) branch, add the argument `ref="release"`.
 * periodically rebuilt binaries/source tarballs are available via

```r
install.packages("lme4", repos = c("http://lme4.r-forge.r-project.org/repos", 
    getOption("repos")))
```

(version 1.0-4 is available as of 2013-09-10 16:34:21)
* The binary versions from the r-forge repository (at least the Windows version) will (apparently) not work on R version 3.0.0; *either* an earlier (<=2.15.3) or a later (>=3.0.1) version of R is required.
* *Solaris/Solaris Studio compiler users*: the Eigen linear algebra library, on which the new version of `lme4` is built, is apparently incompatible with Sun compilers; it is unclear whether the fault lies with `Eigen` (which claims to be compatible with the C++1998 standard) or with the Sun compilers. If you need to build `lme4` with Sun compilers, and would be willing to help in sorting out the Sun/Eigen incompatibilities, please contact the package maintainers.
* We hope that the current CRAN version (0.999999-2) will be replaced by a nearly identical version called `lme4.0` (currently version 0.9999-2) (*this is currently under negotiation with the CRAN maintainers*).  `lme4.0` is a maintenance version and will only be changed to fix documented bugs. `mer` objects from older versions of `lme4` can be converted to `lme4.0`-usable form via `convert_old_lme4()` in the `lme4.0` package.
* all other versions (`lme4a`, `lme4b`, `lme4Eigen`) are deprecated.

Changes to `lme4` are listed in the [NEWS.Rd file](https://github.com/lme4/lme4/blob/master/inst/NEWS.Rd); the major points are reiterated below.

## For end-users

### Changes in behavior
* Because the internal computational machinery has changed, results from the newest version of `lme4` will not be numerically identical to those from previous versions.  For reasonably well-defined fits, they will be extremely close (within numerical tolerances of 1e-4 or so), but for unstable or poorly-defined fits the results may change, and very unstable fits may fail when they (apparently) succeeded with previous versions. Similarly, some fits may be slower with the new version, although on average the new version should be faster and more stable. There are more numerical tuning options available than before (see below); non-default settings may restore the speed and/or ability to fit a particular model without an error. If you notice significant or disturbing changes when fitting a model with the new version of `lme4`, **please notify the maintainers**.
* In the past, `lmer` automatically called `glmer` when `family` was specified. It still does so, but now warns the user that they should preferably use `glmer` directly.
* `VarCorr` returns its results in the same format as before (as a list of variance-covariance matrices with `correlation` and `stddev` attributes, plus a `sc` attribute giving the residual standard deviation/scale parameter when appropriate), but prints them in a different (nicer) way.
* by default `residuals` gives deviance (rather than Pearson) residuals when applied to `glmer` fits (a side effect of matching `glm` behaviour more closely).

### Other user-visible changes
* More use is made of S3 rather than S4 classes and methods: one side effect is that methods such as `fixef` no longer conflict with the `nlme` package.
* The internal optimizer has changed. `[gn]lmer` now has an `optimizer` argument; `"Nelder_Mead"` is the default for `[n]lmer`, while a combination of `"bobyqa"` (an alternative derivative-free method) and `"Nelder_Mead"` is the default for `glmer`; to use the `nlminb` optimizer as in the old version of `lme4`, you can use `optimizer="optimx"` with `control=list(method="nlminb")` (you will need the `optimx` package to be installed and loaded). See the help pages for details.
* Families in GLMMs are no longer restricted to built-in/hard-coded families; any family described in `?family`, or following that design, is usable (although there are some hard-coded families, which will be faster)
* `[gn]lmer` now produces objects of class `merMod` rather than class `mer` as before

### New features
* A general-purpose `getME()` accessor method has been used to allow extraction of a wide variety of components of a mixed-model fit; this has been backported to `lme4.0` for compatibility.
* `bootMer`, a framework for obtaining parameter confidence intervals by parametric bootstrapping.
* `plot` methods similar to those from the `nlme` package (although missing `augPred`).
* A `predict` method, allowing a choice of which random effects are included in the prediction.
* Likelihood profiling (and profile confidence intervals) for `lmer` and `glmer` results.
* `nAGQ=0`, an option to do fast (but inaccurate) fitting of GLMMs.
* Using `devFunOnly=TRUE` allows the user to extract a deviance function for the model, allowing further diagnostics/customization of model results.
* The internal structure of [gn]lmer is now more modular, allowing finer control of the different steps of argument checking; construction of design matrices and data structures; parameter estimation; and construction of the final `merMod` object (see `?modular`).
* Negative binomial models (still under development).

### Still non-existent features
* Automatic MCMC sampling based on the fit turns out to be very difficult to implement in a way that is really broadly reliable and robust; `mcmcsamp` will not be implemented in the near future. We recommend parametric boostrapping via `bootMer`, or the Kenward-Roger 
approximation implemented in the `pbkrtest` package and leveraged by the `lmerTest` package and the `Anova` function in the `car` package.
* "R-side" structures (within-block correlation and heteroscedasticity) are being implemented in an experimental version on Github, but with no definite release timetable

## Notes for package writers

[Current package compatibility test results][pkgtest]

[pkgtest]: http://htmlpreview.github.io/?https://github.com/lme4/lme4/blob/master/misc/pkgtests/lme4_compat_report.html

* `lme4`-old and `lme4.0` produce objects of class `mer`, `lme4`-new produces `merMod` objects, so any methods written for class `mer` will at least have to be copied to work with class `merMod`
* You can distinguish `lme4`-old from `lme4`-new via package version; the last old-style version of `lme4` on CRAN is 0.999999-2, so anything after that is `lme4`-new (the current version on <http://lme4.r-forge.r-project.org/repos> is 1.0-4).
* So you can test e.g. if `packageVersion("lme4")<="0.999999-2"` (yes, you do want the quotation marks; package versions are weird objects in R).
* For example:

```r
if (inherits(object, "merMod")) {
    ## code relevant to development version
} else if (inherits(object, "mer")) {
    ## stable version code if you need to differentiate between lme4.0 and lme4:
    pkg <- attr(class(object), "package")
    ## you can use if (pkg=='lme4.0') or, e.g.
    getFromNamespace("nobars", ns = pkg)
    ## to retrieve un-exported functions from the package
}
```

* `getME(.,.)` should often get the components you want
* As mentioned above, `lme4` now uses S3 rather than S4 methods in many places. That makes some things easier, but it also means (for example) that classes and methods do not get imported in the same way.

### Things that won't work

* Direct extraction of slots via `@` (please use `getME()` instead).
* Methods that depend on `lme4` producing objects of class `mer` (write new methods for class `merMod`; `lme4` now has `isLMM()`, `isGLMM()`, `isNLMM()` that should help you distinguish different types of model if you need to).
* The `method` argument is no longer used in `glmer` (`nAGQ=1` vs `nAGQ>1` specifies whether to use Laplace or AGQ).
* `expandSlash` no longer exists (although it does exist within the `findbars` function: could be reconstituted??).
* Because S4 methods are used less, and (S4) *reference* classes are used, considerably fewer methods are exported, but they are generally available as reference class method "slots" (if absolutely necessary: see the first bullet point above).
