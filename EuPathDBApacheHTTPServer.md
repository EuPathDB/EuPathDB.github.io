---
layout: page
title: Apache HTTP Server Infrastructure
permalink: apahcehttpserver.html
---

### Introduction
EuPathDB uses Apache HTTP Server (AHS) for CGI scripts, request rewrites, access control, directory browsing and as a proxy to the WDK application deployed in Tomcat. HTTP Server is not strictly required; the WDK requires only Tomcat. The other functionality (CGI scripts, etc) can be handled by Tomcat as well, although EuPathDB staff may have trouble supporting a pure Tomcat solution.

If you decide to use HTTP Server on your project, the next step is to decide how to install and configure it and how to lay out directory and file components. The following describes EuPathDB's setup of Apache HTTP Server. You are not required to follow these guidelines. If you have an existing HTTP Server deployment that you are happy with you probably will be able to install the EuPathDB-DevTools into your existing infrastructure. However if you are starting de novo, we suggest following these guidelines to maximise EuPathDB's ability to provide support.

EuPathDB's AHS setup is based on RHEL's default installation so logs, configurations, website working directories are in the vicinity of their expected locations. A system administrator who has experience with AHS on RHEL-based systems should have little trouble supporting the EuPathDB additions. We add to this default organization a few configuration files and subdirectories. The additional organization and naming conventions allow us to support hundreds of development and production websites across a dozen projects.

We configure AHS to use [name-based virtual hosts](https://httpd.apache.org/docs/2.2/vhosts/name-based.html) so a single IP address can server multiple websites.

This chapter will explain the steps for setting up HTTP Server for the project `FooDB`. The project name, `FooDB`, is also used for the model name for WDK utilities, e.g. `wdkCache -model FooDB`, and for the project id in some website URLs. That is, the name used for the data loading and web applications is carried through to the system infrastructure.

## Quick Start Installation

These are the basic steps for setting up HTTP Server for the project `FooDB`. 

`yum install httpd`

`mkdir -p /var/www/FooDB/{project_home,gus_home}`

`mkdir /var/log/httpd/FooDB`

`mkdir /etc/httpd/conf/{disabled_sites,enabled_sites,lib}`

Edit `/etc/httpd/conf/httpd.conf`, add these lines to the end

        NameVirtualHost *:80
        NameVirtualHost *:443
        Include conf/enabled_sites/[^\*]*.conf


Restart the httpd service and confirm it starts correctly.

`service httpd restart`



## Detailed Installation

### Install Apache HTTP Server

Install Apache HTTP Server from the `http` package in your vendor's YUM repository (e.g. CentOS or RHEL base). 

`yum install http`


### Directory structure

The default directories used by AHS on RHEL-based systems include

  - DocumentRoot at `/var/www/html`
  - Access and Error logs in `/var/log/httpd`
  - Configurations under `/etc/httpd/conf` and `/etc/httpd/conf.d`

EuPathDB's directories build upon these defaults to support hundreds of development and production websites across a dozen projects. We use naming conventions to aid server management and allow automated tasks.

### Directory structure for `/var/www`

Directories or symbolic links in `/var/www` are named, in all lowercase, after virtual hostnames such as `www.foodb.org` and `dev1.foodb.org`. This allow you to easily identify which directory a given URL is reading from. For example, the website at `http://qa.foodb.org/` is served from the directory `/var/www/qa.foodb.org/`.

For sites that are versioned there is one parent directory for each site in `/var/www` (e.g. `FooDB`, `ToxoDB`, `EuPathDB`, etc). Within that parent directory are the versioned directories (eg. `FooDB/foodb.1`, `FooDB/foodb.2`, ...). These parent directories should begin with an uppercase letter to help distinguish them as containers of versioned sites. Any version-independent supporting directories (eg. `awstats`, `pubcrawler`) can also go in the parent directory.

Symbolic links named after the Virtual Host are used at the `/var/www` level to point to the versioned directory that is in active use. The symbolic link names should be all lowercase.

The AHS virtual hosts are configured with that symbolic link as the path to the `DocumentRoot` so to change site versions you need only re-point the symbolic link. The configuration files for virtual hosts are in `/etc/httpd/conf/enabled_sites` and are also named after the virtual hostname.

For example:

The virtual host for http://foodb.org uses `/var/www/foodb.org/html` as its  `DocumentRoot` (configured in `/etc/httpd/conf/enabled_sites/foodb.org.conf`). `/var/www/foodb.org` is a symbolic link to a directory version (`FooDB/foodb.prod`) that contains all the necessary AHS directories ( `html`, `cgi-bin`, etc)

`foodb.org -> FooDB/foodb.prod/`

Similarly the virtual host for `http://dev1.foodb.org` uses `/var/www/dev1.foodb.org/html` as its `DocumentRoot` (configured in `/etc/httpd/conf/enabled_sites/dev1.foodb.org.conf`). `/var/www/dev1.foodb.org` is a symbolic link to a directory containing all the site files.

`dev1.foodb.org -> foodb/foodb.dev1/`

If the web site has a Tomcat webapp component (e.g. WDK driven sites), the versioned web site directory and the webapp should use the same name. For example, the web site at `/var/www/FooDB/foodb.dev1` has a Tomcat webapp component named '`foodb.dev1`'.



Here's an example of a partially expanded directory structure of `/var/www/`


            ApiDB/
            apidb.org -> ApiDB/apidb_v2.0
            Common/
                 \__ siteFiles/
                           \__ downloads/
                           \__ webServices/
                           \__ tutorials/
                 \__ tmp/                 
            CryptoDB/
                 \__ Common/
                         \__ cgi-bin
                         \__ html
                 \__ cryptodb.b20/
                 \__ cryptodb.b21/
                 \__ cryptodb.b22/
                           \__ etc/
                           \__ cgi-bin/
                           \__ cgi-lib/
                           \__ dashboard
                           \__ gus_home/
                           \__ html/
                           \__ project_home/
                           \__ webapp/
                 \__ pathwaytools/
            cryptodb.org -> CryptoDB/cryptodb.b21/
            dev1.apidb.org -> ApiDB/apidb_v2.1/
            qa.cryptodb.org -> CryptoDB/cryptodb.b22/
            ToxoDB/
                 \__ toxo.b20/
                 \__ toxo.b4/
                 \__ toxo.b5/
            toxodb.org -> ToxoDB/toxodb.b20
            v4-1.toxodb.org -> ToxoDB/toxo.b4
            v5-0.toxodb.org -> ToxoDB/toxo.b5

Three projects are shown: `ApiDB`, `CryptoDB`, `ToxoDB`. Within `CryptoDB` there are serveral subdirectores for different release versions: `cryptodb.b20`, `cryptodb.b21`, `cryptodb.b22`. (The suffix is EuPathDB's convention to indicate our build number; `.b20` is Build 20; you may use a diffent naming convention that suits your project). The `cryptodb.b22` directory is expanded to show the first level of subdirectories, including `project_home` which has the source code used by the GUS build system. The `etc` directory is a EuPathDB convention where we keep files used by internal utilities such as `rebuilder`. The other directories in `cryptodb.b22` are the working files used by AHS and Tomcat; these are created and populated by the GUS build system. `CryptoDB/Common` contains files that belong to the `CryptoDB` project but are independent of releases. The `/var/www/Common` directory is for files that are independent of projects. For example, EuPathDB uses `Common` to hold tutorials, BLAST databases, and bulk download files.

Several symbolic links are shown. For example, `cryptodb.org` points to the `CryptoDB/cryptodb.b21` directory and `qa.cryptodb.org` points to `CryptoDB/cryptodb.b22`. By convention, the symbolic link name match a webserver host name. If you want to find the installation for `http://qa.cryptodb.org/` you can simply `cd /var/www/qa.cryptodb.org` on the server. Alternatively, say you wanted to find the installation for Build 22 but did not know that was `qa.cryptodb.org`, you could `cd /var/www/CryptoDB/cryptodb.b22` and end up at the same place.

----

> A benefit of using these naming and directory structure conventions is that it allows EuPathDB to use PerlSections<sup>1</sup> to make virtual hosts in AHS self-configuring. We are able to write Perl code that parses the name of the AHS configuration file (e.g. `/etc/http/conf/enabled_sites/dev1.foodb.org.conf`) to determine the name of the virtual host (`dev1.foodb.org`). It can then look for a symbolic link under `/var/www/` having that name and use it as the basis for setting `DocumentRoot` (`/var/www/dev1.foodb.org/html`). And by looking at what the symbolic link is pointing to, AHS will learn the name of the Tomcat webapp (`foodb.dev1`) and can properly configure the `jk_mod` tomcat connector. Logging for the virtual host will be automatically established in a subdirectory under `/var/log/httpd` (e.g. `/var/log/httpd/dev1.foodb.org`).

> Configuring AHS using PerlSections can be complex and difficult to debug. We do not recommend using it unless you have too many virtual hosts to manage with normal, static configuration files. Still, these naming conventions can aid you in writing static configurations files and help you locate resources on your system. For example, EuPathDB uses custom scripts such as `rebuilder` and `cattail` to simplify building websites from source code and tailing log files, respectively. The scripts are given the hostname of the website to act on and the scripts are able to use naming conventions to bootstrap a suitable execution environment. These custom scripts are not supported outside of the EuPathDB project but we will be happy to provide you copies so you can adapt them to your own project.

----

### Directory structure for `/var/log/httpd`

The default AHS log directory on RHEL is `/var/log/httpd`. EuPathDB's convention is to create new subdirectories for each AHS virtual host.

            /var/log/httpd/
                     \__ dev1.foodb.org/
                     \__ foodb.org/
                     \__ qa.foodb.org/

----

### Directory structure for `/etc/httpd/conf/`

The default configuration directories for AHS on RHEL are `/etc/httpd/conf.d` and `/etc/httpd/conf`. EuPathDB's convention is to create new subdirectories `disabled_sites`, `enabled_sites`, `lib`.

            /etc/httpd/conf/
                     \__ disabled_sites/
                                   \__ old.foodb.org.conf
                     \__ enabled_sites/
                                   \__ dev1.foodb.org.conf
                                   \__ foodb.org.conf
                                   \__ qa.foodb.org.conf
                     \__ lib/

Each AHS virtual host has its own configuration file. Active configuations are kept in `enabled_sites`. To take a site offline temporarily, move the file to `disabled_sites` and reload the httpd service. If you have shared configurations, you can put them `lib` directory and `Include`
them in the virtual host configuration file.

----

### Patch `/etc/httpd/conf/http.conf`

The global AHS configuration file is `/etc/httpd/conf/http.conf`. At the end of the file add the lines

        NameVirtualHost *:80
        NameVirtualHost *:443
        Include conf/enabled_sites/[^\*]*.conf

This enables [name-based virtual hosts](https://httpd.apache.org/docs/2.2/vhosts/name-based.html) on ports 80 (http) and 443 (https), and then includes all the individual virtual host conf files in the `enabled_sites` directory.

----

### Example virtual host configuration file

See [https://gist.github.com/mheiges/1057055](https://gist.github.com/mheiges/1057055) . (This sample may be out of date.)

This is just a sample based on EuPathDB's infrstructure. You will want to alter it to fit your project. Refer to AHS documentation for syntax help.

----
#### Footnotes

<sup>1</sup> https://perl.apache.org/docs/2.0/api/Apache2/PerlSections.html