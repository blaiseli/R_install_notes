Notes on installing R and RStudio under Debian-based Linux distributions
========================================================================


About possibly interfering packaging systems
--------------------------------------------


### Linux distributions and packaging systems

Programs and libraries (i.e. predefined functionalities that programs can
re-use) can be installed in various ways. The hard-core way is to install
things directly from the source code provided by the programmers, but this can
be sometimes a little complicated. Linux is usually obtained in the form of a
"distribution", which includes the core operating system, a variable amount of
pre-installed tools, and a database of installable packages (i.e. sets of
programs and libraries) together with tools to manage their installation. On
Debian-based distributions, this is the
[`apt`](https://www.debian.org/doc/manuals/debian-reference/ch02.en.html)
system.

It is generally recommended to use your distributions' official packaging
system to install programs, whenever possible. This should help avoiding
incompatibilities between packages.


### Language-specific packaging mechanisms

However, programming languages such as R tend to come with their own way to
install packages. R has an `install.packages` function that does precisely
that.

Some R packages can then be installed either using your distributions'
packaging system, or `install.packages`, and things may get messy if you use a
mix of both approaches. One reason is that your distributions' packaging system
has to be used with administrative privileges (typically, prefixing commands
with `sudo`), whereas you should probably avoid using R with such privileges if
you want to avoid destroying your system by accident. If you have already dealt
with `install.packages`, you may have noticed error messages about packages
that cannot be updated because some place in the disk is not writeable. That's
the typical symptom of an interference between R internal mechanisms used as a
normal user, and your `apt`-installed R packages: R is likely trying to update
them, but they are not stored in your user directory.

So why not stick to `apt`, then? Unfortunately, it will eventually happen that
some R packages are either not available that way, or not enough up-to-date for
your taste or necessities. So it may be wise to avoid as much as possible to
install R packages using `apt`. The `r-base-core` and `r-base-dev` packages may
actually be sufficient. Another viable alternative might be to install R from
source, and would have the benefit that updating your system would not mess
with your R installation, but this approach might not be suitable for every
user.


### Bioconductor

To make things more complicated, there is an effort of the bioinformatics
community to provide a set of specialized R packages through it's own system:
[Bioconductor](https://www.bioconductor.org/install/#install-bioconductor-packages)
(which actually uses `install.packages` under the hood).

Bioconductor strives to provide a set of compatible packages, and can also
install R packages that are not specific to bioinformatics. I would therefore
suggest to install packages using Bioconductor whenever possible. You may still
need to install packages directly with `install.packages`, and some packages
might not build unless you install programs and libraries external to R. In the
latter case, try as much as possible to use the `apt` system.


Installing R
------------

As of February 2020, Bioconductor requires R version 3.6, which is not the one
provided by default by the current stable Debian ("buster" / 10) and Long-Term
Support Ubuntu ("bionic" / 18.04) distributions.

To make version 3.6 available in `apt`'s database, you need to provide it's
localization through `apt`'s configuration mechanism, which consists in files
residing in the `/etc/apt/sources.list.d/` directory.

Refer to <https://cloud.r-project.org/bin/linux/> to find the exact source for
R version 3.6 (you need to know on which Debian or Ubuntu version your
distribution is based). This will consist in some text starting with "deb
source=" that you will have to add in a file in `/etc/apt/sources.list.d/`. The
name of the file does not matter but might have to end in ".list". I suggest
`cran.list` (for ["Comprehensive R Archive
Network"](https://cran.r-project.org)). Either proceed by using your favourite
text editor, or on the command-line:

```bash
# If you are using Ubuntu 18.04
# deb_source="deb https://cloud.r-project.org/bin/linux/ubuntu bionic-cran35/"
# If you are using MX Linux 19 or Debian 10
deb_source="deb http://cloud.r-project.org/bin/linux/debian buster-cran35/"
# Add the source for R to the source list
echo ${deb_source} | sudo tee -a /etc/apt/sources.list.d/cran.list
```

In order to avoid some warning messages, you also need to register the authentication key so that `apt` recognizes the package source as trusted:
```bash
# If you are using Ubuntu 18.04
# sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9
# If you are using MX Linux 19 or Debian 10
sudo apt-key adv --keyserver keys.gnupg.net --recv-key 'E19F5F87128899B192B1A2C2AD5F960A256A04AF'
```

Update the package database:
```bash
sudo apt update
```

You may now install R and the necessary tools to install R packages:
```bash
sudo apt install r-base-core r-base-dev
```

If everything worked as expected, you should now have a minimal R installation.
Check that you have the desired version with the command `R --version`. If you
get something older than 3.5, good luck: you may have another version of R
interfering with the one you just installed. The present document does not
cover the subject of already installed R versions and how to be sure to use the
correct one.


Installing RStudio
------------------

RStudio does not seem to be provided as a package by the `apt` system, neither
is it an R package. You need to install it "manually" using a package or
"installer" provided here:
<https://www.rstudio.com/products/rstudio/download/#download>.

Either download the most relevant "installer" for your distribution, by
clicking on the link and continuing the mouse way, or copy the link and
proceed to the download and installation the command line way (the actual
version number may vary):

```bash
# If you are using Debian 10, MX Linux 19 or Ubuntu 18.04:
wget https://download1.rstudio.org/desktop/bionic/amd64/rstudio-1.3.1093-amd64.deb
# For a distribution based on Debian 9, such as MX Linux 18:
# wget https://download1.rstudio.org/desktop/debian9/x86_64/rstudio-1.3.1093-amd64.deb
# Install it (https://unix.stackexchange.com/a/159114/55127):
sudo apt install ./rstudio-1.3.1093-amd64.deb
```


Note the dot (".") in the last command: it tells `apt install` that you want to
use this exact downloaded file and not a package from the database whose name
is "rstudio-1.3.959-amd64.deb".

If installation fails, it may be due to missing dependencies. Look at error
messages for hints, and use help from colleagues or the internet and your
package manager to install the missing packages before trying again.


Using Bioconductor to install R packages
----------------------------------------

As mentioned earlier, one way to install R packages is to do it via the
Bioconductor project, which maintains a set of hopefully mutually-compatible
packages.

Within an R session, start by installing the `BiocManager` R package:

```r
install.packages("BiocManager")
```

Then, whenever you want to install packages, let BiocManager perform some updates:

```r
BiocManager::install()
```

And finally, use it to install the new packages:

```r
BiocManager::install(c("evaluate", "highr", "markdown", "stringr", "yaml", "htmltools", "caTools", "bitops", "knitr", "jsonlite", "base64enc", "rprojroot", "rmarkdown"))
```

As already mentioned, installing packages may require other tools, that are not
part of R. In particular, you will likely need compilers because R packages sometimes
use other programming languages internally, such as C, C++ (that you can get by
installing the `build-essential` package using `apt`) and Fortran. You might also
need development libraries, that you should try to install using `apt` after
consulting the internet. It may be useful to know that development libraries
packages often have names that end in "-dev" and provide files with names
ending in ".h" (for "header") or ".so" (possibly followed by extra dots and
numbers) (for "shared object"). If you see error messages suggesting that such
a file is missing, you will be happy to know that the `apt` system has tools
that help finding the name of the package providing the missing file (see
<https://askubuntu.com/a/1912/129295> and <https://wiki.debian.org/apt-file>).

