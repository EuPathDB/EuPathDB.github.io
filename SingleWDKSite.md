---
layout: page
title: Preparing a Single WDK Website
permalink: singlewdksite.html
---

### Introduction

This section describes the highlights of setting up directories for a single WDK-based website served by Apache HTTP Server (AHS) and following EuPathDB's naming conventions. If you are not using AHS or are installing into a different system infrastructure design then most of these instructions will not apply and you will need to develop your own setup procedure.

**Requirements**: _Apache HTTP Server Infrastructure_

EuPathDB uses name-based virtual hosting. Each website has its own host name. In this section we will use `jane.foodb.org` as an example (this will be a website where Jane will do her code development). The `jane.foodb.org` host name needs to be resolveable to an IP address. Ideally this is done by adding an entry to DNS; alternately, for internal development/testing, you can add entries to `/etc/hosts`.

In the _Apache HTTP Server Infrastructure_ document we discussed the directory structure `/var/www` used by EuPathDB for organizing the source code and installations of individual websites.

      /var/www/
             \__ FooDB/
                     \__ foodb.b1/
                               \__ cgi-bin/
                               \__ cgi-lib/
                               \__ gus_home/
                               \__ html/
                               \__ project_home/
                               \__ webapp/


`mkdir -p /var/www/FooDB/foodb.b1/{project_home,gus_home}`
`mkdir /var/log/httpd/jane.foodb.org`

Check out source code into project_home and build (see WDK documentation). After building and installing the WDK you should have several directories such as `html`, `cgi` under `/var/www/FooDB/foodb.b1`

Edit the sample AHS configuration file [https://gist.github.com/mheiges/1057055](https://gist.github.com/mheiges/1057055) and place it in `/etc/httpd/conf/enabled_sites`. We recommend you name the file after the virtualhost name, for example `jane.foodb.org.conf` (the `.conf` extension is required). Making the substitutions documented at the top of the file should be enough to get a virtual host online but you will want to make other changes to fit your project.
