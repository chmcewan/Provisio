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
    
Commented scripts that don't use *never* are syntactically valid without Provisio and may be self-sufficient if there is no distinction between *once* and *always* in production. Even here, Provisio can still be helpful during development when configurations may be less well determined. 

### Secondary provisio commands

In addition to the primary user command (*up*), Provisio provides a small collection of commands that are useful to invoke from inside a Provisiofile. Specifically:

    $ provisio download <URL>

Downloads and caches a remote dependency, such as a tarball. Similarly

    $ provisio install <PACKAGE>
    $ provisio pip <PACKAGE>
    $ provisio npm <PACKAGE>
    
performs a package manager install that checks and populates a local cache before performing the install proper. All of this cacheing creates an explicit manifest of dependencies and supports off-line or read-only (e.g. CD-ROM) provisioning.

Provisio also provides some simple commands to aid management and local configuration, such as

    $ provisio cat <file>
    
which reads a file to standard output while interpolating {{foo}} with the environmental variable *foo*. This is sufficient for most paramaterised configuration needs. Speaking of environmental variables, 

    $ provisio env
    
echoes configuration-time environmental variables, such that they can be sourced by e.g. init scripts. Crucially, Provisio makes a distinction between environmental variables that begin with an underscore ("_"). These are considered private and, although used during provisioning, will not be stored in the provisioned system, e.g. passwords.

Lastly, 

    $ provisio include <path>
    
allows for some basic decomposition of provisioning scripts. Note that cacheing occurs seperately for each Provisiofile such that common provisioning (e.g. base Linux provisioning) is shared amongst particular systems (e.g. web and database servers). 


