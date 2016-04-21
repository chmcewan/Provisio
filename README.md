# Provisio
The lightweight provisioner

## Synopsis
Provisio adds just enough syntactic sugar to Bash scripts to make them more suitable for provisioning servers, virtual machines and containers. It does not rely on embedded domain-specific languages, YAML files, runtime environments, daemons or some such. It's just Bash with some tasteful "smart comments". 

What Provisio provides over and above Bash is the following:

* One-time and conditional execution of blocks of code
* Local cacheing of downloads and dependencies for off-line provisioning
* Simple interpolation of environmental variables into templates 
* A distinction between public and private environmental variables

It's not big and it's not clever. But it is quite useful.

Pimped scripts are syntactically valid without Provisio and, depending on the features employed, may be self-sufficient as releasable provisioning scripts. Provisio tries to be helpful during development, when provisioning is less well determined. 

## Usage
The business-end of Provisio is invoking the following command in the same directory as a *Provisiofile*.

    $ provisio up [/path/to/environment-vars]
  
 The *Provisiofile* is essentially (and will be translated into) a Bash script. Embedded smart comments control execution and secondary provisio commands provide additional provisioning functionality. 
 
 An example of smart comments is

    #do foo once
      echo $(date) > installed-date.txt
    #end
    
    #do bar always
      echo $(date) > updated-date.txt
    #end
    
    #do baz never
      rm -fR /
    #end

Pretty simple. Mostly it saves your provisioning script from ugly sentry file boilerplate. It also creates a nice, clean report of what code blocks are being executed or skipped with proper redirection to logs for sub commands. 
