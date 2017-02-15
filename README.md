# Provisio
The lightweight provisioner

## Synopsis
Provisio adds just enough syntactic sugar to shell scripts to make them more suitable for provisioning servers, virtual machines and containers. It does not rely on embedded domain-specific languages, YAML files, runtime environments, daemons or whatever. It's just Bash with some tasteful annotations and declarative one-liners. 

Provisio provides the following benefits over a pure shell script:

* Idempotency (i.e. one-time execution of blocks of code)
* Local cacheing of dependencies for off-line or read-only provisioning
* Fail-fast error handling with clean reporting and transparent log redirection
* A distinction between public and private environmental variables
* High-level utility functions for common provisioning tasks 

Of course, you could do all of this in your provisioning script, but it would be ugly and annoying.

## Quick start

Provisio is a single file that is easy to inject into servers, virtual machines and containers, e.g.

    $ curl -s https://raw.githubusercontent.com/chmcewan/Provisio/master/provisio | sudo sh
    
Like Make, Docker and Vagrant, Provisio is invoked in the same directory as a `Provisiofile`.

    $ provisio up
  
Unlike Make, Docker and Vagrant, a `Provisiofile` is just an annotated Bash script. Annotations organise and control script execution. Provisio commands abstract common configuration and dependency management tasks. 

Like Chef, Puppet and other tools, the Provisiofile emphasises the declarative semantics of your provisioning process. Unlike those tools, the Provisiofile preserves the full expressive power of the shell.

## The Provisiofile

It should all seem pretty obvious after an example. 
 
```
#!/bin/bash

#task harden_ssh once
    echo "PasswordAuthentication no" >> /etc/ssh/sshd_config
    echo "PermitRootLogin no" >> /etc/ssh/sshd_config
#end

#task install_dependencies once
    provisio install nginx
    provisio install httpd-tools    
    provisio install pip uwsgi
    provisio install pip flask    
#end

#task install_python_devtools once if development
    provisio install pip pytest
    provisio install pip lettuce
    provisio install pip webtest
#end

#task update_website always
    cp www/* /var/www/
    cp lib/* /usr/lib/python2.7/site-packages/
#end

#task configure_nginx always
    rm -f /etc/nginx/.htpasswd    
    htpasswd -b -c /etc/nginx/.htpasswd $admin_user $admin_password
    provisio cat etc/nginx.conf > /etc/nginx/nginx.conf
#end

#task restart_services always
    systemctl enable nginx
    systemctl restart nginx
#end

```
 
Clearly, the syntax of annotations is

    #task <name> [ once | always | never ] [ if|unless <variable> ]
        ...
    #end

Conditional tasks only check that `$<variable>` is (not) empty. More sophisticated conditional logic should probably be part of the task.

Often there is no distinction between `once` and `always` in production. Provisio can still be useful during development, when configurations are unstable and reprovisioning is common. In the most restricted cases, where no other provisio dependencies exist, running `sudo sh Provisiofile` should be equivalent to `provisio up`.

### Dependency management

Provisio uses cacheing to create an explicit manifest of dependencies and support off-line, read-only (e.g. CD-ROM) and archival provisioning. The cache is kept in a `.provisio` directory beside the Provisiofile.

    $ provisio install [ yum | yum-group | apt | rpm | pip | npm | npm-global | docker ] <package>
    $ provisio install rpm-key <package> <url>
    
performs a managed software install using the local cache. If no package manager is supplied, Provisio will try to infer one at runtime which can offer a degree of platform independence for common packages. 

For all other cases

    $ provisio download <url>

downloads and caches a remote dependency. The (possibly cached) file will appear "downloaded" to the /tmp directory because the working directory may be read-only at provision-time.

### Configuration utilities

Provisio provides some simple commands to aid local configuration and reduce line noise.

    $ provisio set <key> <value> <file>
    $ provisio get <key> <file>

allows config files that have the common `key=value` syntax to be treated as a persistent hash table.

    $ provisio cat <file>
    
streams `<file>` to stdout while substituting all occurences of `{{foo}}` with the environmental variable `foo`. This is sufficient for many paramaterised configuration file needs. 

Provision-time environmental variables can be persisted on the provisioned system, e.g. to be sourced by init scripts

    $ provisio env [filter] > /some/file
    
where `[filter]` can be used to grep a specific subset of variables based on e.g. prefix. Additionally, environmental variables that begin with an underscore ("_") are considered "private" and, although available during provisioning, will never be echoed and persisted on the provisioned system. This is obviously useful for passwords and so on.

Lastly, 

    $ provisio include <path>
    
supports decomposing Provisiofiles so that common configurations (e.g. base Linux) can be shared amongst different systems (e.g. web and database servers). Cacheing is shared across systems, whereas execution state is not.





