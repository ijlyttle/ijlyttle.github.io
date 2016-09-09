# utilr{{Private}}

This is a lightweight package, built on top of Hadley Wickham's (henceforth Hadley) [devtools](https://github.com/hadley/devtools) and Jim Hester's [withr](http://github.com/jimhester/withr), to make it easier for you to install R pagkages hosted on {{Private}}'s installation of GitHub, and also to make it easier to work outside of {{Private}}.

As with everything in R, there is some up-front cost to setting up your "ecosystem". However, once you have everything set up, "it just works".

## Access to {{Private}} GitHub

If you are reading this, you likely already have access. To make sure (or to help others), point your favorite browser to: https://{{private_host}}, and make sure that you have an operational account.

### Installing R Packages

To install R packages from {{Private}} GitHub, you will not need to install git. You will need only to be able to access the {{Private}} GitHub API.

#### Personal Access Token

This is adapted from [Hadley's API-key best practices](https://github.com/hadley/httr/blob/master/vignettes/api-packages.Rmd#appendix-api-key-best-practices)

The installation function uses a Personal-Access Token (PAT) to authorize you to the GitHub API. Don't worry, the function takes care of the details - your responsibility is to get a PAT, and to put it in a place that the installation function will expect it.

Getting a PAT is easy:

1. Go to [{{Private}} GitHub > Settings > Tokens](https://{{private_host}}/settings/tokens). 

2. Click the "Generate new token" button, add a description, click the "Add token" button. The defaults have always worked well for me.

3. Copy the token to your clipboard, as this is your one-and-only chance to "see" the token on the website. 

Next, you have to put the token where the installation function will expect it. The best practice is to put this in a file called `.Renviron` in your home directory.

1. To determine your home directory, type `normalizePath("~/")` in the R console.

2. Create a new text file. If in RStudio, do File > New File > Text file.

3. Create a line like this:

    `GITHUB_{{Private}}_PAT=blahblahblahblahblahblah`
  
    where the name `GITHUB_{{Private}}_PAT` reminds you which API this is for an `blahblahblahblahblahblah` is your personal access token, pasted from the clipboard.

4. Make sure the last line in the file is empty (if it isn't, R will silently fail to load the file). If you're using an editor that shows line numbers, there should be two lines, where the second one is empty.

5. Save in your home directory with the filename `.Renviron`. If questioned, YES you do want to use a filename that begins with a dot `.`.

    Note that by default dotfiles are usually hidden. But within RStudio, the file browser will make `.Renviron` visible and therefore easy to edit in the future. Also note that on Windows, you may need to use the Windows Explorer to make the file visible in order to save it.

6. Restart R. `.Renviron` is processed only at the start of an R session.

To test that R can access this environment variable, type `Sys.getenv("GITHUB_{{Private}}_PAT")` into the console.

#### Installing underlying packages

The utilr{{Private}} package relies on two other packages, devtools and httr. Installing these packages are available on CRAN, so you can enter into the R console:

```R
install.packages(c("devtools", "httr"))
```

If your connection times out, the problem may be the {{Private}} firewall. To address this, you can add these lines to your `.Renviron` file (which you will have to update periodically)

```
no_proxy="{{private_host}}"

https_proxy="https://<proxy_url_and_port>"
https_proxy_user="<proxy_user>:blahblahblah" 

http_proxy=${https_proxy}
http_proxy_user=${https_proxy_user}
```

Of course, you will have to adapt `https_proxy_user` to your particular needs. Remember to restart your R session so that the environment variables are read. 

The converse is true: you may want to download packages outside when you are outside of the {{Private}} firewall, and it might fail. In that case, you can tell R to temporarily ignore `https_proxy`, we provide a function `without_proxy`. For example:

```R
without_proxy(install.packages("ggplot2"))
without_proxy(install_github("hadley/ggplot2"))
```

#### Installing utilr{{Private}}

The last step is to install utilr{{Private}}. Unfortunately, the first-time installation of this package is non-intuitive. The good news is that it will be much easier to install subsequent packages from  {{Private}} GitHub.

Just copy and paste this code into your R console:

```R
devtools::install_github(
  repo = "<account>/utilr{{Private}}", 
  auth_token = Sys.getenv("GITHUB_{{Private}}_PAT"), 
  host = "{{private_host}}/api/v3"
)
```

This should return witn an indication of success, ending with:

```
* DONE (utilr{{Private}})
Reloading installed utilr{{Private}}
```

Now, you can just use the `install_github_{{Private}}()` function to install any R package stored at {{Private}} GitHub:

```R
library("utilr{{Private}}")
install_github_{{Private}}("<account>/utilr{{Private}}")
```

Again, a bit of up-front pain to make things easier going forward.

### Contributing to {{Private}} GitHub

Contributing R packages to the {{Private}} community is one of the goals of this GitHub project. Although it is outside the purview of this package, if you are interested in contributing R code, either on {{Private}} GitHub, or on *the* GitHub, I would strongly urge you to refer to [Hadley's "R Packages"](http://r-pkgs.had.co.nz) book. The [chapter on Git and GitHub](http://r-pkgs.had.co.nz/git.html) may be particularly interesting to you.

Finally, another [piece of advice from Hadley](https://github.com/hadley/devtools#other-tips) that I find particularly useful, and have adapted to our use.

In the same way that you created an `.Renviron` file, it can be useful to create a `.Rprofile` file in your home directory as well.

Here's some useful code to put into your `.Rprofile` file:

```
.First <- function() {
  options(
    repos = c(CRAN = "https://cran.rstudio.com/"),
    browserNLdisabled = TRUE,
    deparse.max.lines = 2)
}

if (interactive()) {
  suppressMessages(require("devtools"))
  suppressMessages(require("utilr{{Private}}"))
}
```

This code loads the devtools and utilr{{Private}} packages into your workspace, whenever it is interactive.