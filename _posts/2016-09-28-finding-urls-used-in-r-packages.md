---
layout:   post
comments: true
share:    true
date:     2016-07-28
modified: 
excerpt: "How to figure out URLs internally used in some R packages"
tags: [R, web services, API, JSON, XML]
categories:
---

<!--
http://stackoverflow.com/questions/4962495/where-to-find-free-public-web-services
-->

[CRAN]:         https://cran.r-project.org   "The Comprehensive R Archive Network"
[Bioconductor]: http://bioconductor.org      "Bioconductor provides tools for the analysis and comprehension of high-throughput genomic data."

[RCurl]:   https://cran.r-project.org/web/packages/RCurl/index.html         "RCurl: General Network (HTTP/FTP/...) Client Interface for R"
[curl]:    https://cran.r-project.org/web/packages/curl/index.html          "curl: A Modern and Flexible Web Client for R"
[Rlabkey]: https://cran.r-project.org/web/packages/Rlabkey/index.html       "Rlabkey: Data Exchange Between R and LabKey Server"

[RStripe]: https://cran.rstudio.com/web/packages/RStripe/index.html         "RStripe: A Convenience Interface for the Stripe Payment API"
[tumblR]:  https://cran.r-project.org/web/packages/tumblR/index.html        "tumblR: Access to Tumblr v2 API"
[biomaRt]: http://bioconductor.org/packages/release/bioc/html/biomaRt.html  "Interface to BioMart databases (e.g. Ensembl, COSMIC, Wormbase and Gramene)"

[jsonlite]: https://cran.rstudio.com/web/packages/jsonlite/index.html       "jsonlite: A Robust, High Performance JSON Parser and Generator for R"
[rjson]:    https://cran.rstudio.com/web/packages/rjson/index.html          "rjson: JSON for R"

[twitteR]: https://cran.rstudio.com/web/packages/twitteR/index.html         "twitteR: R Based Twitter Client"


Many R libraries query web services for information.
Sometimes the purpose is to download the complete dataset to be analyzed.
In other occasions we just need to add some extra bits to the data we are exploring.
Packages like [RStripe] and [tumblR] in [CRAN]
or [biomaRt] in [Bioconductor]
perform such queries.

Usually,
this kind of packages use the R library [RCurl] 
(or some alternative such as [curl]) 
to connect to the specific web service and get the required data.
Generally,
some function in the package
composes a URL following the web API specification, 
and then, `RCurl::getURL` is called over this URL to retrieve the data into the R session.

Getting to know such URLs is generally valuable as it can be used for
testing, debugging, understanding how the API works, or simply to use it with some other software.
Nevertheless, most of such libraries build the URLs internally 
and do not make them available to the end user.

Occasionally I like to __rebuild__ one of such packages
for the URLs to be printed when the web service is called. 
Few days ago I did so with the [Rlabkey] library 
and I would like to share here how I did it as it may be useful for some of you:


1. I cloned the library from its [Github repository](https://github.com/LabKey/labkey-api-r)
but you could also download the source code from [CRAN] or [Bioconductor].

1. I explored the [DESCRIPTION](https://github.com/LabKey/labkey-api-r/blob/develop/Rlabkey/DESCRIPTION) file
and found out that [Rlabkey] uses [RCurl].
In this case it is indicated in the _Depends_ row of the DESCRIPTION file, 
but for some other libraries it will usually be in the _Imports_ 
or even in the _Suggests_ one.

1. I went to the [R](https://github.com/LabKey/labkey-api-r/tree/develop/Rlabkey/R) folder
where the R code leaves,
and searched for scripts with the word `getURL`
(actually [Rlabkey] uses `getURI` which is a synonym of `getURL`).
I used the Linux Shell command
[grep](http://linuxcommand.org/man_pages/grep.html)
but any other method will do...
even [searching in Gighub](https://github.com/LabKey/labkey-api-r/search?utf8=✓&q=getURI&type=Code).

1. I found out that the call to `getURI` is done by the
[labkey.get](https://github.com/LabKey/labkey-api-r/blob/6f0983f5d460ffbe2014bfe87e5c07cc058d29e9/Rlabkey/R/labkey.defaults.R#L51)
function in the  [Rlabkey] library, 
and that the function takes the desired URL as an input parameter.
Then I just added a `print` at the beginning of the function.

1. I reinstalled the library from my local copy using:  
`install.packages ("labkey-api-r/Rlabkey", repos = NULL)`


[Here](https://github.com/dmontaner/labkey-api-r/commit/fd57e1d73782e1c579338da306ab0258ea4f1ffb)
you can find the git commit adding the `print` line.



Hidden curl calls
--------------------------------------------------------------------------------

Many web services yield __JSON__ formatted results.
Usually those are read into R _lists_ or _data.frames_
using functions like `fromJSON` implemented in the [jsonlite] library.
Such functions can read from a text file but also from a web connection;
for example `fromJSON` internally calls `RCurl::getURL` to read from a URL.

Thus, when looking for the URL as I did above,
you may sometimes need to search for the `fromJSON` or similar functions.

The same can happen with serviced returning __XML__ format.
Functions like
may be then used to find where the URL is called so that it can be printed for you.
