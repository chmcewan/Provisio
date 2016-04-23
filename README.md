# Provisio
The lightweight provisioner

## Synopsis
Provisio adds just enough syntactic sugar to Bash scripts to make them more suitable for provisioning servers, virtual machines and containers. It does not rely on embedded domain-specific languages, YAML files, runtime environments, daemons or whatever. It's just Bash with some tasteful "smart comments" and declarative one-liners. 

In particular, what Provisio provides over and above Bash is the following:

* One-time execution of blocks of code
* Local cacheing of dependencies for off-line or read-only provisioning
* Fail-fast error handling and transparent log redirection
* Simple interpolation of environmental variables into templates 
* A distinction between public and private environmental variables

You could do all this in pure Bash, but it would be ugly and annoying.

## Quick start

Provisio is a single file that is easy to import via a host file system or the web, e.g.

    $ wget -P /usr/bin/ https://raw.githubusercontent.com/chmcewan/Provisio/master/provisio
    $ chmod 755 /usr/bin/provisio
    
The business-end of Provisio is invoking the following command in the same directory as a `Provisiofile`.

    $ provisio up [conf/development.env]
  
The `.env` file will simply be sourced in a way that makes variables available to your provisioner and child processes. 
    
A Provisiofile is essentially (and will be directly translated into) a Bash script. Embedded smart comments control execution and secondary provisio commands invoked within the Provisiofile provide additional functionality. 



## Discussion
 
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
    
Commented scripts are syntactically valid without Provisio. If they don't use `never` or `if` and don't use secondary provsio commands then they will be self-sufficient since there is usually no distinction between `once` and `always` in production. In such cases, Provisio can still aid development, where configurations have not stabalised and reprovisioning is common. 

### Secondary provisio commands

In addition to the primary user command (`up`), Provisio provides a small collection of commands that are useful to invoke from inside a Provisiofile. Specifically:

    $ provisio install <MANAGER> <PACKAGE>
    
performs a package manager (currently `yum`, `npm` and `pip`) install that checks and populates a local cache before performing the install proper. Similarly

    $ provisio download <URL>

downloads and caches a remote dependency, such as a tarball. Regardless of the ultimate source, the file is "downloaded" to the /tmp directory, in case the working directory is read-only.

All of this cacheing creates an explicit manifest of dependencies and supports off-line or read-only (e.g. CD-ROM) provisioning at a later date. Provisio tries to support diverse reporting, installation, security or provenance needs.

Provisio also provides some simple commands to aid management and local configuration, such as

    $ provisio cat <file>
    
which streams a file to standard out while substituting `{{foo}}` with the environmental variable `foo`. This is sufficient for most paramaterised configuration file needs. Regarding environmental variables, 

    $ provisio env
    
echoes provision-time environmental variables, such that they can be stored and sourced by e.g. init scripts. Crucially, Provisio makes a distinction between environmental variables that begin with an underscore ("_"). These are considered private and, although used during provisioning, will not be echoed and stored in the provisioned system, e.g. passwords.

Lastly, 

    $ provisio include <path>
    
supports decomposition of provisioning scripts so that common configurations (e.g. base Linux) can be shared amongst different systems (e.g. web and database servers). Cacheing is conserved across systems. One-time executions are not.

## Bugs and caveats

Provisio offers 80% of provisioner functionality for 20% of the effort. How to best avoid it becoming [this](https://xkcd.com/1654/) is still not clear. Expect to fork and use as a foundation for your own must-have provisioning routines. Pull requests welcome if sufficiently general and compelling.



