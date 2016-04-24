# Provisio
The lightweight provisioner

## Synopsis
Provisio adds just enough syntactic sugar to Bash scripts to make them more suitable for provisioning servers, virtual machines and containers. It does not rely on embedded domain-specific languages, YAML files, runtime environments, daemons or whatever. It's just Bash with some tasteful annotations and declarative one-liners. 

Provisio provides the following over and above Bash:

* One-time execution of blocks of code
* Local cacheing of dependencies for off-line or read-only provisioning
* Fail-fast error handling and transparent log redirection
* A distinction between public and private environmental variables
* High-level utility functions for common provisioning tasks 

You could do all of this in regular Bash yourself, but it would be ugly and annoying.

## Quick start

Provisio is a single file that is easy to import from the host file system or the web, e.g.

    $ wget -P /usr/bin/ https://raw.githubusercontent.com/chmcewan/Provisio/master/provisio
    $ chmod 755 /usr/bin/provisio
    
Like Make, Docker and Vagrant, Provisio is invoked in the same directory as a `Provisiofile`.

    $ provisio up [conf/development.env]
  
The arbitrary `.env` file will be sourced such that variables will be available to your provisioning script and child processes. See the `env` and `cat` commands below for examples of how Provisio uses environmental variables.

## Provisiofile

Unlike Make, Docker and Vagrant, a Provisiofile is essentially a Bash script. Annotations organise and control execution and secondary commands provide additional functionality. It should all seem pretty obvious after an example: 
 
```
TODO
```
 
## Annotations
 
Annotations remove ugly boilerplate from the provisioning script. They provide clean reporting of how code blocks are being executed with proper error handling and redirection of command output to a persistent log. Their syntax is:

    #task <name> [ once | always | never ] [ if|unless <variable> ]

Annotated scripts that don't use `never` or `if` may have no dependency on Provisio, since there is usually no distinction between `once` and `always` in production. In such cases, Provisio can still aid development when configurations are unstable and reprovisioning is common. 

Conditional tasks only check that `$<variable>` is (not) empty. More sophisticated conditional logic should probably be part of the task and expressed in Bash.

## Secondary commands

In addition to the primary user-facing command (`up`), Provisio provides a small collection of commands to invoke from the Provisiofile that simplify dependency management and configuration.

#### Dependency management

Provisio uses extensive cacheing to create an explicit manifest of dependencies and support off-line, read-only (e.g. CD-ROM) and archival provisioning. 

    $ provisio install [ yum | apt | rpm | pip | npm | npm-global ] <package>
    
performs a managed software install using the local cache. If no package manager is supplied, Provisio will try to infer one at runtime which can offer a degree of platform independence for common packages. 

Due to a combination of its ubiquity and terrible website, a special case is provided for installing Oracle's JVM

    $ provisio install jdk [7u55-b13/jdk-7u55-linux-x64]

For websites that are more accomodating

    $ provisio download <url>

downloads and caches a remote dependency, such as a tarball. The (possibly cached) file will be "downloaded" to the /tmp directory because the working directory may be read-only at provision-time.

#### Configuration utilities

Provisio also provides some simple commands to aid local configuration and reduce line noise.

    $ provisio set <key> <value> <file>
    $ provisio get <key> <file>

allows you to treat config files that have a `key=value` syntax as a persistent hash table.

    $ provisio cat <file>
    
streams `<file>` to stdout while substituting all occurences of `{{foo}}` with the environmental variable `foo`. This is sufficient for most paramaterised configuration file needs. 

Provision-time environmental variables can be propogated into the provisioned system (e.g. to be sourced by init scripts) using

    $ provisio env [filter] > /some/file
    
where `[filter]` can be used to grep a specific subset of variables. Additionally, environmental variables that begin with an underscore ("_") will be considered "private" and, although available during provisioning, will not be echoed and persisted in the provisioned system. This is useful for passwords and similar.

Lastly, 

    $ provisio include <path>
    
supports decomposing Provisiofiles so that common configurations (e.g. base Linux) can be shared amongst different systems (e.g. web and database servers). Cacheing is shared across systems whereas execution state is not.

## Contributions

Provisio offers 80% of provisioner functionality for 20% of the effort. The line between useful utility and [universal install script](https://xkcd.com/1654/) is a fine one. Pull requests are welcome, but simply forking and adding your specific must-have provisioning routines may be more productive than trying to solve all the problems for everyone. Have fun!



