# Deploying Software with Environment Variables

Scientific software -- typically scripts written in bash, python or R -- are 
often written on the developer's desktop machine and then need to be deployed
to another machine which may have different file paths for executables or data.

## File paths

### Rules

* Use well thought out directory structures
* **Do not** _hardcode_ absolute paths.
* Avoid using relative paths.
* Use one ore more _configureable_ `basePath` or `baseUrl` variables.
 
#### Directory structures

A well structured directory tree is a beautiful thing. Just like the Dewey
decimal system, it allows you to find what you want based on certain characteristics
that map onto the directory structure. Useful structures for storing data might 
look like any of the following:

* `model > year > month > day`
* `project > cohort > parameter > datestamp`
* `device > version > month`

Basically, any structure that captures how people want to access the data will
work. As a side benefit, when data is placed on a web server, these directory
trees map nicely onto RESTful URLs.

### File paths

In order to deploy software to machines with different setups we need to
accommodate different file paths. So absolute paths should **never** be allowed
in scientific software. Relative paths are also suspect because they depend on
the environt `PATH` to be set appropriately, a common source of error and
confusion.

Instead, a configureable `basePath` (`baseUrl` for web accessible data) should
be used. When combined with a _well known directory structure_, it will be
possible to create absolute paths on any machine by simply modifying the
`basePath`.

## Using environment variables

The simplest way to create different absolute paths on different unix machines
is to use _environment variables_. All scripting languages have a way to access
variables from the environment in which they are running and this document 
fill focus on **R**.

### Setting environment variables

You can see what environment variables are already set in any Unix environment
with `printenv`. _(Google "windows environment variables" to learn how to do 
this on a Windows machine.)_

Here are a few environment variables from my OSX system:

```
$ printenv
TERM_PROGRAM=Apple_Terminal
PYENV_ROOT=/Users/jonathan/.pyenv
SHELL=/bin/bash
TERM=xterm-256color
TMPDIR=/var/folders/vd/zpgw5sv92ngdx11k5dzqz5800000gn/T/
TERM_PROGRAM_VERSION=433
OLDPWD=/Users/jonathan/Projects/MazamaScience
TERM_SESSION_ID=6115100F-D213-4770-84BF-4C805F8C5330
GIT_EDITOR=/usr/bin/vim
SVN_EDITOR=/usr/bin/vim
USER=jonathan
...
```

Different shells use different syntax for setting environment variables but here
is a small chunk from the `.bash_profile` file I keep in my home directory and 
run by typing: `/bin/bash ~/.bash_profile`

```
...
export SVN_EDITOR="/usr/bin/vim"
export GIT_EDITOR="/usr/bin/vim"
...
```

You can of course create other bash scripts on individual systems specyfing
information you need in your scripts. For example, the `.bash_experiment1` script
on a **system_A** might have:

```
export CUSTOM_SOFTWARE_A="/opt/local/bin/custom_software_a"
export USER_DATA="/data/experiment123/participant_data/2020/"
export SHARED_R_FUNCTIONS="/usr/abc/R/experiment123/utils/"
```

If you log on to **system_A** and `source .bash_experiment1`, all of these
variables will be defined in the current environment.

### Using environment variables in R scripts

Accessing information from the environment in **R** is simple and these variables
can then be used in the script to create absolute paths appropriate for **system_A**:


```
# Using all cap variable names reminds us that these are environment variables

# ----- Get environment variables ----------------------------------------------

USER_DATA <- Sys.getenv("USER_DATA")
SHARED_R_FUNCTIONS <- Sys.getenv("SHARED_R_FUNCTIONS")

# ----- Set variables specific to this script ----------------------------------

# These might relate to the "well structured directory"

PROJECT <- "project_A"
COHORT <- "cohort_2"

# Lower case reminds us this is not a single, configured varabile
parameters <- c("param1", "param2", "param3")

...

# ----- Source shared R scripts ------------------------------------------------

R_files <- list.files(SHARED_R_FUNCTIONS, pattern = "^.+\\.R")

for ( file in R_files ) {
  source( file.path(SHARED_R_FUNCTIONS, file) )
}

# ----- Create absolute data paths ---------------------------------------------

dataPathList <- list()

for ( parameter %in% parameters ) {

  name <- paste(PROJECT, COHORT, PARAMETER, sep="_")
  dataPathList[[name]] <-
    file.path(USER_DATA, PROJECT, COHORT, parameter, "all_patients.csv")

}

# Validation
dataPathExists <- lapply(dataPathList, file.exists)
 
# Write your own test/error message as needed at this stage.

# Then go ahead and process the data
```

This simple use of environment variables should be enough to handle many issues 
related to deployment of software to different systems.

