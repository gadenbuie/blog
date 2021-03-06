---
slug: checking-your-r-package-on-solaris
title: Checking your R package on Solaris
authors:
- Gábor Csárdi
date: "2020-05-14"
tags:
- package development
- solaris
output:
  html_document:
    keep_md: true
---



## TL;DR

1. To check your package on Solaris, call [`rhub::check()`](
   https://r-hub.github.io/rhub/reference/check.html) as usual and
   choose one of our Solaris builders.

2. Bookmark this page, in case you get an email from CRAN about your
   package failing on Solaris.

## Oracle Solaris

[Oracle Solaris](https://en.wikipedia.org/wiki/Solaris_\(operating_system\))
is a non-free Unix operating system. CRAN regularly
[tests R packages](
https://blog.r-hub.io/2019/04/25/r-devel-linux-x86-64-debian-clang/)
on Solaris 10. See [this talk by Uwe Ligges](
https://channel9.msdn.com/Events/useR-international-R-User-conferences/useR-International-R-User-2017-Conference/KEYNOTE-20-years-of-CRAN)
if you want to know more about the motivation for this, at 32:45.

This is a post about checking that your R package on Solaris, using R-hub,
and how you can create your own Solaris virtual
machine to debug your R code.

## OpenCSW

[OpenCSW](https://www.opencsw.org/about/) is a repository of open source
software packages. It uses a package manager tool called `pkgutil`.
`pkgutil` is similar to `apt`, `yum`, etc.

OpenCSW greatly varies in terms of how current its various packages are.
By default OpenCSW uses its "testing" distribution, which has a rolling
release model. The most recently added packages are in the "unstable"
distribution. OpenCSW unstable has now a new R 4.0.0 build, called
`r_base`. It will move to the "testing" distibution in a couple of days.
OpenCSW `r_base` 4.0.0 is one of the R builds used on R-hub's Solaris
builder.

R-hub also has a repository (catalog) of OpenCSW packages,
which we use for system requirements that are not (yet) updated in the
official OpenCSW catalog. The R-hub catalog is at
https://files.r-hub.io/opencsw/. (This is the URL you need to use in
the `pkgutil.conf` file or with `pkgutil -t`.)

## CRAN's Solaris builder

CRAN uses two sets of compilers for their R package checks.
[Oracle Developer Studio](https://www.oracle.com/tools/developerstudio/)
(ODS) is a commercial product that supports Oracle Solaris 10. In addition,
OpenCSW packages GCC, [version 5.5.0 currently](
https://www.opencsw.org/packages/CSWgcc5core/).

CRAN compiles R packages with ODS by default. If ODS is not able to
compile a package, they use GCC instead. GCC should be able to compile
most (all?) CRAN R packages. Notably, Rcpp and all packages linking to it
are compiled with GCC.

The [CRAN package check flavours page](
https://cran.r-project.org/web/checks/check_flavors.html)
links to the [details of the Solaris configuration](
https://www.stats.ox.ac.uk/pub/bdr/Rconfig/r-patched-solaris-x86).

## R-hub's Solaris builder

R-hub's setup is similar. Our Solaris virtual machine has three R
versions:
* 32-bit GCC build at `/opt/csw/bin/R`, from the OpenCSW `r_base` package.
  The [configuration of this build is in OpenCSW](
  https://buildfarm.opencsw.org/source/xref/opencsw/csw/mgar/pkg/r/trunk/Makefile).
* 64-bit GCC build at `/opt/csw/bin/amd64/R`, from the same OpenCSW
  `r_base` package.
* 32-bit ODS build, see the a `config.site` file [in our `solarischeck`
  repository](https://github.com/r-hub/solarischeck/blob/master/config.site.cc).

If you call [`rhub::platforms()`](
https://r-hub.github.io/rhub/reference/platforms.html) you'll see two
Solaris builders:
* `solaris-x86-patched` uses the 32-bit GCC build of R. This is also what
  [`rhub::check_on_solaris()`](
  https://r-hub.github.io/rhub/reference/check_shortcuts.html) uses.
* `solaris-x86-patched-ods` uses the 32-bit GCC build to install all
  dependencies of the package, but then runs `R CMD check` with the ODS
  build. This maximizes the chance of a meaningful `R CMD check` run.

## System requirements

It is unclear how system requirements are installed on CRAN's Solaris.
R-hub's Solaris machine installs all system requirements from OpenCSW.
The list of installed packages are in the [`solarischeck` GitHub repository](
https://github.com/r-hub/solarischeck/blob/master/sysreqs.txt).

If you need another library for your package, please let use know and
we can install it. See [the current list of OpenCSW packages](
https://www.opencsw.org/get-it/packages/).

If the library you need is missing from OpenCSW, or it is outdated,
let us know, there is a good chance that we can add it or update it.

## Creating your own Solaris 10 VM

R-hub's Solaris check is great if you want to make sure that your package
works on Solaris before a CRAN submission. If it does not work, and the
fix is not obvious, then you probably want to debug it on a Solaris
machine.

We have a template for the [packer](https://packer.io) automated VM builder
tool that helps you do this with as little work as possible. (Which is not
the same as _little_ work, mind you!) You'll need:

* A sufficiently powerful computer to run a virtual machine.
* packer, available for Windows, macOS, Linux, free and open source.
* Virtualization software: VMware or VirtualBox. VirtualBox is open
  source and available for many platforms. VMware has free evaluation
  versions.
* You need to create an Oracle ID, and download the Oracle Solaris 10
  install DVD image and also Oracle Developer Studio. They are free for
  non-commercial use, without support.
* You need to wait about 15-30 minutes for packer to build the virtual
  machine.

Please see [the detailed steps in the docs of the `solarischeck` repo](
https://github.com/r-hub/solarischeck/tree/master/packer#readme).

If you want to install Solaris yourself without the packer automation,
our internal R-hub docs have a [detailed walkthrough](
https://docs.r-hub.io/technical/solaris/).

## Need help? Contact us!

If creating a virtual machine is not the right solution for you, do not
despair! We have other ways to help you with your Solaris issues, please
contact us by [opening an issue in the R-hub issue tracker](
https://github.com/r-hub/rhub/issues).

## Resources, links

* [CRAN's Solaris config](
  https://www.stats.ox.ac.uk/pub/bdr/Rconfig/r-patched-solaris-x86) by
  Brian Ripley.
* Our [`solarischeck` GitHub repository](
  https://github.com/r-hub/solarischeck) for Solaris related tools.
* [packer template](
  https://github.com/r-hub/solarischeck/tree/master/packer#readme)
  to build your Solaris VM:
* An R ODS build and other Solaris related tools, e.g. newer R builds
  for Solaris [on our web site](https://files.r-hub.io/solaris).
* A walkthrouh of a manual Solaris installation, [on our internal docs
  site](https://docs.r-hub.io/technical/solaris).
* The R-hub OpenCSW catalog is at <https://files.r-hub.io/opencsw>.
* [Prior art and inspriration](https://github.com/jeroen/solarisvm).
* Our packer config is based on [Edwin Biemond's work](
  https://github.com/biemond/packer-vagrant-builder).
* To get help with R and Solaris please open an issue in our
  [issue tracker](https://github.com/r-hub/rhub/issues).
