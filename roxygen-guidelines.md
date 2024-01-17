# Roxygen documentation guidelines



The scope of this document is to provide *opinionated yet motivated
guidelines and best practices* for documenting R objects (especially functions)
with **roxygen2**.

This is based on:

- **roxygen2** vignettes: `browseVignettes("roxygen2")`, in particular
[Generating Rd
files](https://cran.r-project.org/web/packages/roxygen2/vignettes/rd.html).
- The [tidyverse style guide](https://style.tidyverse.org/documentation.html).
- Package development experience.

Examples are provided to show the concrete application of the described
principles.


## General guidelines and documentation workflow

Note that the ultimate goal of documentation is to have meaningful and
consolidated help pages. For this reason, being happy about the roxygen tags in
the source code is no enough, and the final help pages should always be checked
as rendered by `help(<TOPIC>)`.

To quickly iterate trough documentation development, updates and checks, the following is recommended.

- Use `devtools::check_man()`, which does some basic checks on top of
roxygenizing your package, hinting about e.g. undocumented arguments.
- Assess the development version of the documentation via
`pkgload::dev_help(<TOPIC>)`, which is quicker than re-building the whole
package and using `help()`, allowing faster iterations.
- The final documentation should be then checked using `R CMD check` (possibly
with packages `devtools` or `rcmdcheck`), and the help pages assessed with
`help(<TOPIC>)` after installing the package.
- For larger documentation efforts, it can be worth to generate the full PDF
manual, where it is easier to also cross-check consistency at the whole package
level.

An approach seen too often is to introduce roxygen documentation tags very early
in the development of e.g. a function, using simple, pretty-uninformative
placeholder tags. Early documentation is very useful if already thought-through
upfront, to help thinking about and better design e.g. interfaces, usage,
behavior, return values.

Early, poor documentation tags only serve the purpose of a skeleton, but may
give the wrong impression of an actual documentation effort. Instead, it can be
a good idea to add normal comments describing the function behavior (even a
single comment saying what the function does). This can be the basis of proper,
thought-through documentation tags at a more advanced development stage. Also
note that RStudio offers the creation of a roxygen skeleton (_Code > Insert
Roxygen skeleton_ or Shift+Ctrl+Alt+), so there is little need / benefit in
creating one manually, with poor content, ahead of time.


## Content organization

Content structure and order of the roxygen2 documentation should be organized for
readability, alignment with the rendered content, maintainability.

- Apart from the `@title` (see below), always use **sentence case** with a final
**period (.)**.
- Content **order** should reflect how sections appear in the resulting help
page: title, description, arguments, details, value, custom sections,
references, see also, examples.
    - Note that `@family` belongs to the 'See Also' section, so place it
    accordingly. 
    - Place `@inherit*` and `@template` tags in the relevant place for the
    content you are including.
- Use an **empty line** to separate sections
- Avoid stating explicit `@title` and (single-paragraph) `@description` tags,
they will be inferred by position.
- Put single-paragraph content right after the tag (`@details Bla bla.`).
- If you have multi-paragraph content for a single tag, separate the paragraphs
with an empty line and start the first paragraph on a new line rather than right
after the tag. Note that multi-paragraph description always requires an explicit
`@description`.
- Place **NAMESPACE**-related tags `@importFrom`, `@import`, `@export` at the
bottom and in this order.
    - Use one `@importFrom` per symbol (not per package) and try to avoid
    `@import`.
    - For **un-exported** objects documented for internal purposes, specify
    `@keywords internal` instead of `@export`.
- Use
[**markdown**](https://cran.r-project.org/web/packages/roxygen2/vignettes/rd-formatting.html):
If not done at package level, `@md` should be the last element (so it can easily
be removed when moving to package-level).
- Do not use `@rdname` if not documenting multiple objects (see below).
- **Break lines** at 80 characters of content (you can set _Tools > Global
Options > Code > Display > Margin column: 80_ and _> Show margin_ for visual
aid): No matter how wide your screen is, human brain is still not meant for
processing long text, which would be poorly readable (code is a different
story).
    - Use RStudio menu _Edit > Reflow comment_ (Shift+Ctrl+/) after selecting a
    single block (selecting several parts might screw things up), which will
    also indent the lines after the first when needed.


### Note about `@title`

Keep in mind that the content of `@title` (the top paragraph, w/o explicit tag
as recommended above) is used and displayed for two purposes in the R
documentation:

1. **Title** of the topics help page (the most know and natural usage).
2. Brief description / hint to topics in the package **help index** (e.g.
`help(package = "roxygen2")`), a less known and often forgotten usage.

With this in mind:

- You may want to use title case (more natural for 1.) or sentence case (more
sensible for 2.), but be consistent within a package (and never add a final
period, ugly for 1.).
- Avoid using the very same topic name as title (e.g. the function name when
documenting a function). This does not provide any extra information, especially
in the help index, where describing e.g. `myFunction` as "myFunction" is pretty
pointless.


## Advanced usage and consistent documentation

Several techniques and tools are available to enable and support a more
consistent and maintainable documentation.

This can be described in detail in the [Do repeat
yourself](https://cran.r-project.org/web/packages/roxygen2/vignettes/rd.html#do-repeat-yourself)
section of vignette _Generating Rd files_, and are summarized as follows

- Cross-link documentation files with `@seealso` and `@family`.
- Reuse parameter documentation with `@inherit`, `@inheritParams`, and `@inheritSections`.
- Document multiple functions in the same place with `@describeIn` or `@rdname`.
- Run arbitrary R code with `@eval`.
- Create reusable templates with `@template` and `@templateVar`.


### Roxygen templates and example scripts

Roxygen **templates** allow modularizing documentation content. Shared content
sits in R files under the **man-roxygen** directory, and is included using the
`@template` tag.

Similarly, examples written as normal R code can be included in the 'Examples'
section using `@example` (without `s`), which for long examples is way more
convenient and less tedious than writing them as roxygen comments in the
`@examples` section.

- The **man-roxygen** directory must be added to .Rbuildignore, e.g. via
`usethis::use_build_ignore("man-roxygen")`
- **Template files** should be called _`<TAG>-<NAME>.R`_ depending on which
`@<TAG>` they contain (typically one per template), and be included according to
the corresponding tag order. This improves the readability and maintenance of
template-based documentation
    - e.g., _`param-first_arg.R`_ contains `#' @param first_arg Bla bla bla.`
- Markdown must be specified via **`@md`** in each template if not defined at
package level.
- **Examples R scripts** should be called _`ex-<FUNCTION_NAME>.R`_ and placed in
_man-roxygen_ (for convenience). They are then included via `@example
man-roxygen/ex-<FUNCTION_NAME>.R`.


### Using `@rdname` and `@describeIn`

Tags `@rdname` and `@describeIn` are a convenient way to document multiple
functions in the same file. See the roxygen2 vignette [Generating Rd
files](https://cran.r-project.org/web/packages/roxygen2/vignettes/rd.html)
(`vignette("rd", package = "roxygen2")`) for more detail.

In both case, not that `@title` should be specified only for the _main_
documentation object, since it will be ignored for others (a help page allowing
only one title).

It may also be convenient to collect the main generic documentation content as
roxygen2 tags for a `NULL` object with an explicit `@name` (see examples below).

#### Usage of `@rdname` {-}

**`@rdname`** provides the greatest flexibility for combining documentation
(description, arguments, details, etc.) of several objects into a single
documentation entry.

Tag `@rdname` should the first tag, and should be used **exclusively** when
_appending_ documentation content for a new object to another existing
`@rdname`. Having an `@rdname` for single and full-documented objects should be
avoided since

- it is redundant and unnecessary;
- it gives the impression we want to document this object alongside others;
- it does not help if we later want to change the `@rdname` to a different
topic, since the documentation content must be probably adapted (e.g. `@title`
should be removed as it would be ignored).

####  Usage of `@describeIn` {-}

**`@describeIn <name> <description>`** is meant for a set of functions with
(almost) same arguments and that can be described in a general way in the
'Description' section, whereas individual `<description>`s are collected in a
final section 'Functions'. 

Tag `@describeIn` should be the last tag before the specific `@examples` and
namespace tags (and possibly after specific `@param`).


## Examples

The following examples make use of the demo package `roxygenExPkg`. The package can be installed as:


```r
remotes::install_github(
  "miraisolutions/techguides/roxygen-guidelines/roxygenExPkg"
)
```

The command `help(package = "roxygenExPkg")` will give you access to the package help.




### General content organization

See the corresponding `?roxygenExPkg::fun_doc`.



```r
#' Function Documentation Example
#'
#' Show an example of roxygen tags organization, providing documentation best
#' practices and guidelines. An explicit `@description` is only needed for
#' multi-paragraph descriptions.
#'
#' @param first_arg Description of the first argument. Also define data type /
#'   class / structure / vectorization expectations where not obvious, and
#'   possibly use (see 'Details').
#' @param second_arg Second argument of the function.
#'
#' @details More details or context about the function and its behaviour. List
#'   possible side-effects, corner-cases and limitations here. Also use this
#'   section for describing the meaning and usage of arguments when this is too
#'   complex or verbose for the `@param` tags.
#'
#' @section Custom section:
#' The content of a custom section.
#'
#' @return Returns `NULL`, invisibly. The function is called for illustration
#'   purposes only.
#'
#' @references
#' Hadley Wickham, Peter Danenberg and Manuel Eugster. roxygen2: In-Line
#' Documentation for R. [https://CRAN.R-project.org/package=roxygen2]().
#'
#' Hadley Wickham, The tidyverse style guide.
#' [https://style.tidyverse.org/documentation.html]().
#'
#' @seealso [fun_doc_tpl()], [roxygen2::roxygenize()]. Even if you just put
#'   comma-separated links to functions, don't forget the final period (.).
#' @family function documentation examples
#'
#' @examples
#' # illustrate through examples how functions can be used
#' fun_doc("example_string", 3)
#'
#' @importFrom roxygen2 roxygenize
#' @importFrom roxygen2 roxygenise
#' @export
#'
#' @md
fun_doc <- function(first_arg, second_arg) {
  invisible()
}
```


### Templates and example scripts

See the corresponding `?roxygenExPkg::fun_doc_tpl`.



```r
#' Template Documentation Example
#'
#' Show an example of roxygen tags organization, providing documentation best
#' practices and guidelines, using templates.
#'
#' @template param-first_arg
#' @param second_arg Description of the second argument.
#'
#' @template details-fun_doc
#'
#' @section Custom section:
#' The content of a custom section.
#'
#' @template return-NULL
#'
#' @template reference-roxygen2
#' @template reference-tidyverse_style_guide
#'
#' @seealso [fun_doc_tpl()], [roxygen2::roxygenize()]. Even if you just put
#'   comma-separated links to functions, don't forget the final period (.).
#' @family function documentation examples
#'
#' @example man-roxygen/ex-fun_doc_tpl.R
#'
#' @importFrom roxygen2 roxygenize
#' @importFrom roxygen2 roxygenise
#' @export
#'
#' @md
fun_doc_tpl <- function(first_arg, second_arg) {
  invisible()
}
```


### `@rdname`

See the corresponding `?roxygenExPkg::divide`.


```r
#' Scalar Division
#'
#' Divide an object by a scalar.
#'
#' @param x The object to be divided
#' @param d Scalar to divide `x` by.
#'
#' @return The result of the scalar division of `x`.
#'
#' @examples
#' x <- matrix(1:6, 3L)
#'
#' @name divide
#'
#' @md
NULL

#' @rdname divide
#'
#' @details `divide_by()` performes generic division by `d`.
#'
#' @examples
#' divide_by(x, 1.5)
#'
#' @export
#'
#' @md
divide_by <- function(x, d) {
  x / d
}

#' @rdname divide
#'
#' @details `divide_by2()` performes division by two.
#'
#' @examples
#' divide_by2(x)
#'
#' @export
#'
#' @md
divide_by2 <- function(x) {
  divide_by(x, 2)
}

#' @rdname divide
#'
#' @details `divide_by3()` performes division by three.
#'
#' @examples
#' divide_by3(x)
#'
#' @export
#'
#' @md
divide_by3 <- function(x) {
  divide_by(x, 3)
}

#' @rdname divide
#'
#' @details `divide_by4()` performes division by four.
#'
#' @examples
#' divide_by4(x)
#'
#' @export
#'
#' @md
divide_by4 <- function(x) {
  divide_by(x, 4)
}
```


### `@describeIn`

See the corresponding `?roxygenExPkg::times`.


```r
#' Scalar Multiplication
#'
#' Multiply an object by a scalar.
#'
#' @param x The object to be multiplied.
#' @param m Scalar to multiply `x` by.
#'
#' @return The result of the scalar multiplication of `x`.
#'
#' @describeIn times Generic multiplication by `m`.
#'
#' @examples
#' x <- matrix(1:6, 3L)
#'
#' times(x, 1.5)
#'
#' @export
#'
#' @md
times <- function(x, m) {
  x * m
}

#' @describeIn times Multiplication by 2.
#'
#' @examples
#' times2(x)
#'
#' @export
#'
#' @md
times2 <- function(x) {
  times(x, 2)
}

#' @describeIn times Multiplication by 3.
#'
#' @examples
#' times3(x)
#'
#' @export
#'
#' @md
times3 <- function(x) {
  times(x, 3)
}

#' @describeIn times Multiplication by 4.
#'
#' @examples
#' times4(x)
#'
#' @export
#'
#' @md
times4 <- function(x) {
  times(x, 4)
}
```
