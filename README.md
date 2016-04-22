# Provisio
The lightweight provisioner

## Synopsis
Provisio adds just enough syntactic sugar to Bash scripts to make them more suitable for provisioning servers, virtual machines and containers. It does not rely on embedded domain-specific languages, YAML files, runtime environments, daemons or whatever. It's just Bash with some tasteful "smart comments" and declarative one-liners. 

In particular, what Provisio provides over and above Bash is the following:

* One-time and conditional execution of blocks of code
* Local cacheing of downloads and dependencies for off-line or read-only provisioning
* Fail-fast error handling and log redirection without syntactic noise
* Simple interpolation of environmental variables into templates 
* A distinction between public and private environmental variables

You could do all this in pure Bash, but it would be ugly and annoying.

## Usage
The business-end of Provisio is invoking the following command in the same directory as a *Provisiofile*.

    $ provisio up [/path/to/environment-vars]
  
The *environment-vars* file will simply be sourced in a way that makes variables available to sub-commands. For example you may make a distinction between

    $ provisio up config/development.env
    
and

    $ provisio up config/production.env
  
The *Provisiofile* is essentially (and will be translated into) a Bash script. Embedded smart comments control execution and secondary provisio commands invoked within the Provisiofile provide additional functionality. 
 
### "Smart" comments
 
Smart comments remove ugly boilerplate from the provisioning script. They provide clean reporting of how code blocks are being executed with proper error handling and redirection of command output to a persistent log. 

    #do foo once
      echo $(date) > installed-date.txt
    #end
    
    #do bar always
      echo $(date) > updated-date.txt
    #end
    
    #do baz never
      rm -fR /
    #end
    
    #do buzz if production
      echo "It worked!" | mail user@gmail.com
    #end
    
Commented scripts are syntactically valid without Provisio. If they don't use *never* or *if* and don't use secondary provsio commands then they may be self-sufficient when there is no distinction between *once* and *always* in production. For such simple scripts, Provisio can still be helpful during development when configurations may still be in flux (see Example below). 

### Secondary provisio commands

In addition to the primary user command (*up*), Provisio provides a small collection of commands that are useful to invoke from inside a Provisiofile. Specifically:

    $ provisio install <MANAGER> <PACKAGE>
    
performs a package manager (currently *yum*, *npm* and *pip*) install that checks and populates a local cache before performing the install proper. Similarly

    $ provisio download <URL>

downloads and caches a remote dependency, such as a tarball. Regardless of the ultimate source, the file is "downloaded" to the /tmp directory, in case the working directory is read-only.

All of this cacheing creates an explicit manifest of dependencies and supports off-line or read-only (e.g. CD-ROM) provisioning at a later date. Provisio does not assume you are a trendy web-shop with no security or provenance concerns.

Provisio also provides some simple commands to aid management and local configuration, such as

    $ provisio cat <file>
    
which reads a file to standard output while interpolating {{foo}} with the environmental variable *foo*. This is sufficient for most paramaterised configuration file needs. Regarding environmental variables, 

    $ provisio env
    
echoes provision-time environmental variables, such that they can be stored and sourced by e.g. init scripts. Crucially, Provisio makes a distinction between environmental variables that begin with an underscore ("_"). These are considered private and, although used during provisioning, will not be echoed and stored in the provisioned system, e.g. passwords.

Lastly, 

    $ provisio include <path>
    
supports decomposition of provisioning scripts. Note that cacheing occurs seperately for each Provisiofile such that common provisioning (e.g. base Linux provisioning) is shared amongst particular systems (e.g. web and database servers). 

## Example

TODO
