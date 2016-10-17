# Tomcat 8 (`tomcat8`)

Master: [![Build Status](https://semaphoreci.com/api/v1/bas-ansible-roles-collection/tomcat8/branches/master/badge.svg)](https://semaphoreci.com/bas-ansible-roles-collection/tomcat8)
Develop: [![Build Status](https://semaphoreci.com/api/v1/bas-ansible-roles-collection/tomcat8/branches/develop/badge.svg)](https://semaphoreci.com/bas-ansible-roles-collection/tomcat8)

Downloads, installs and configures Apache Tomcat Java servlet container 8

**Part of the BAS Ansible Role Collection (BARC)**

**This role uses version 0.2.0 of the BARC flavour of the BAS Base Project - Pristine**.

## Overview

* Downloads and installs Tomcat as a binary, non-package managed application
* Creates an Upstart or SystemD system service configured to run Tomcat at system startup
* Creates a non-privileged, non-interactive operating system user for running the Tomcat service
* Configures permissions for Tomcat's configuration, logging and data directories, owned by the Tomcat system user
* Creates symbolic links from Tomcat's installation directory to conventional locations, such as '/etc' for 
configuration files
* Optionally, configures the system firewall to allow access to Tomcat services (HTTP), this is enabled by default

## Quality Assurance

This role uses manual and automated testing to ensure its features work as advertised. 
See [here](tests/README.md) for more information.

## Dependencies

* [**bas-ansible-roles-collection.oracle-jdk8**](https://galaxy.ansible.com/bas-ansible-roles-collection/oracle-jdk8/)
  * Minimum version: *0.1.0*

More information on role dependencies is available in the 
[BARC General Documentation](https://antarctica.hackpad.com/BARC-Overview-and-Policies-SzcHzHvitkt#:h=Role-dependencies)

## Requirements

* None

More information on role requirements is available in the 
[BARC General Documentation](https://antarctica.hackpad.com/BARC-Overview-and-Policies-SzcHzHvitkt#:h=Role-requirements)

## Limitations

* Tomcat must be installed as a non-package managed application

There are no operating system packages available for Tomcat 8, from either non-system and system-only package sources.
This therefore requires the manual installation of binary applications, and other related setup tasks, normally 
performed by operating system packages.

This is not an ideal situation, as in many cases non-conventional locations for configuration and/or data will be used.

See [BARC-118](https://jira.ceh.ac.uk/browse/BARC-118) for further details.

*This limitation is considered to be significant. Solutions will be actively pursued.*
*Pull requests to address this will be gratefully considered*

* The version of Tomcat installed is hard-coded to a specific version

Due to the way in which Tomcat is currently installed it is necessary to download a specific version, including the 
patch version.

Currently it is not possible to override this version, without overriding the download location of the Tomcat installer.
In future adding variables to specify the major, minor and patch version of Tomcat would help to address this 
limitation.

See [BARC-119](https://jira.ceh.ac.uk/browse/BARC-118) for further details.

*This limitation is considered to be significant. Solutions will be actively pursued.*
*Pull requests to address this will be gratefully considered*

* This role does not support running Tomcat over HTTPS

Currently Tomcat can only be ran using HTTP, which is insecure, and not in line with the current BAS policy to ensure
all new services use HTTPS wherever possible.

However, it is expected Tomcat will be configured to operate behind a reverse proxy, either using a standalone Nginx 
instance for example, or using a centralised, or shared, load balancing / reverse proxy solution. In these cases 
Tomcat's HTTP interface would not be accessible to the public internet and is therefore a lesser concern.

*This limitation is **NOT** considered to be significant. Solutions will **NOT** be actively pursued.*
*Pull requests to address this will be considered*

See [BARC-120](https://jira.ceh.ac.uk/browse/BARC-120) for further details.

More information on role limitations is available in the 
[BARC General Documentation](https://antarctica.hackpad.com/BARC-Overview-and-Policies-SzcHzHvitkt#:h=Role-limitations)

## Usage

### BARC Manifest

By default, BARC roles will record that they have been applied to a system, this is termed the BAS Manifest.
The specific local facts set for this role are:

* `ansible_local.barc_tomcat8.general.role_applied` - (boolean) records whether a role has been applied
* `ansible_local.barc_tomcat8.general.role_version` - (string) records the version of the role that was applied

Note: You **SHOULD** use this feature to determine whether this role has been applied to a system.
If you do not want these facts to be set by this role, you **MUST** skip the **BARC_SET_MANIFEST** tag.

More information is available in the 
[BARC General Documentation](https://antarctica.hackpad.com/BARC-Overview-and-Policies-SzcHzHvitkt#:h=Role-Manifest)

### Firewall configuration

This role assumes the machine it is applied to is using a system firewall. By default this role will generate a firewall 
service relevant to Tomcat and automatically enable and persist this service to allow access to Tomcat's services.

Specifically:

* On CentOS machines:
    * A set of firewalld firewall services will be generated
    * Each firewall service will contain rules for a single firewall scenario
    * This service will be enabled persistently
    * The firewall service is reloaded
* On Ubuntu machines:
    * A UFW firewall application will be generated (overriding the default firewall application for Nginx)
    * This application contains rules for all firewall scenarios
    * This application will be enabled
    * The firewall service is reloaded

Custom generated firewall services are used to ensure if custom listening ports are used (i.e. not port 8080) 
rules will reflect the ports that are used.

A single firewall scenario is available within this role:

* *tomcat-http*
    * Enables HTTP connections only on the port set by the *tomcat8_firewall_port_http* variable

If you do not want this role to manage the system firewall, or no system firewall is used, you **MUST** skip the 
**BARC_CONFIGURE_FIREWALL** tag when using this role.

Note: If you are using BAS maintained [operating system images](https://github.com/antarctica/packer-vm-templates) this
configuration is fully supported and should be seamless. In other cases additional configuration may be needed depending
on how the system firewall has been configured.

### Tomcat installation method

Tomcat 8 is currently unavailable as an operating system package, from both system and non-system package sources.

Tomcat is therefore installed manually using the Tomcat binary installer (a compressed archive of binaries and related 
configuration scripts).

This has several limitations, including specifying the Tomcat version which is installed, discussed separately, and 
in not using conventional locations for configuration files etc. It also requires users and service management profiles 
(i.e. for SystemD) to be created manually.

This is considered a limitation. See the *Limitations* section for more information.

### Tomcat version

Due to the way Tomcat is installed, discussed in the *Tomcat installation method* section, a specific version of Tomcat
is installed by this role.

Currently this version is: **8.0.33**.

Currently, this version is hard-coded and cannot be changed. Future versions of this role may allow this, until then 
this is considered a limitation. See the *Limitations* section for more information.

**Note:** This version may change in future versions of this role. Such changes will be considered minor releases.

### Tomcat configuration

This role will not modify any of the Tomcat configuration files, such as `server.xml` or `web.xml`. Where default 
versions of these files are unsuitable, it is safe to generate alternate files using a template, or alter specific 
values in default files.

### Tomcat file and directory permissions

The following directories are configured with user and group read/write access, with other users having no access:

* `/opt/tomcat8/conf`
* `/opt/tomcat8/logs`
* `/opt/tomcat8/temp`
* `/opt/tomcat8/work`
* `/opt/tomcat8/webapps`

The user for these directories is set to `tomcat`, the group is set to `adm`. This group is a conventional default on
both CentOS and Ubuntu systems.

**Note:** As the Tomcat user cannot be used interactively, i.e. you cannot login as this user, you will need to add 
other user accounts to the `adm` group to access Tomcat's configuration and log files.

### Typical playbook

```yaml
---
 
- name: ...
  hosts: all
  become: yes
  vars: []
  roles:
    - bas-ansible-roles-collection.tomcat8
```

### Tags

BARC roles use standardised tags to control which aspects of an environment are changed by roles.
Tasks in this role will 'tag' themselves with these tags as appropriate, typically in `tasks/main.yml`.

More information is available in the
[BARC General Documentation](https://antarctica.hackpad.com/BARC-Overview-and-Policies-SzcHzHvitkt#:h=Appendix-B---BARC-Standardised)

### Variables

#### *tomcat8_barc_role_name*

* **MUST NOT** be specified
* Specifies the name of this role within the BAS Ansible Roles Collection (BARC) used for setting local facts
* See the *BARC roles manifest* section for more information
* Example: `tomcat8` 

#### *tomcat8_barc_role_version*

* **MUST NOT** be specified
* Specifies the name of this role within the BAS Ansible Roles Collection (BARC) used for setting local facts
* See the *BARC roles manifest* section for more information
* Example: `2.0.0`

#### *tomcat8_binary_download_url*

* **MAY** be specified
* Specifies the URL for the Tomcat binary installer
* This URL **MUST** resolve to a Tomcat binary installer, packages as a TAR archive
* Example: `http://mirror.vorboss.net/apache/tomcat/tomcat-8/v8.0.33/bin/apache-tomcat-8.0.33.tar.gz`

#### *tomcat8_binary_checksum_sha256*

* **MAY** be specified
* Specifies the SHA256 checksum of the file set by the *tomcat8_binary_download_url* variable
* Example: `c77873c1861ed81617abb8bedc392fb0ff5ebf871de33cd1fcd49d4c072e38b7`

#### *tomcat8_firewall_port_http*

* **MAY** be specified
* Specifies the port on which Tomcat will listen for insecure (HTTP) requests, used for generating firewall rules
* Values **MUST** be a valid system port, as determined by the operating system
* The default value, `8080`, is a conventional default, other values **SHOULD NOT** be used without good reason
* Default: `8080`

## Developing

### Issue tracking

Issues, bugs, improvements, questions, suggestions and other tasks related to this package are managed through the 
[BAS Ansible Roles Collection](https://jira.ceh.ac.uk/projects/BARC) (BARC) project on Jira.

This service is currently only available to BAS or NERC staff, although external collaborators can be added on request.
See our contributing policy for more information.

More information is also available in the
[BARC General Documentation](https://antarctica.hackpad.com/BARC-Overview-and-Policies-SzcHzHvitkt#:h=Issue-Tracking)

### Source code

All changes should be committed, via pull request, to the canonical repository:

`ssh://git@stash.ceh.ac.uk:7999/barc/ROLE-NAME.git` 

A read-only mirror of this repository is maintained on GitHub:

`git@github.com:bas-ansible-roles-collection/ROLE-NAME.git`

More information is available in the
[BARC General Documentation](https://antarctica.hackpad.com/BARC-Overview-and-Policies-SzcHzHvitkt#:h=Source-Code)

### Contributing policy

This project welcomes contributions, see `CONTRIBUTING.md` for our general policy.

### Release procedure

The general release procedure for BARC roles is available in the
[BARC General Documentation](https://antarctica.hackpad.com/BARC-Overview-and-Policies-SzcHzHvitkt#:h=Release-procedures)

## Licence

Copyright 2016 NERC BAS.

Unless stated otherwise, all documentation is licensed under the Open Government License - version 3.
All code is licensed under the MIT license.

Copies of these licenses are included within this project.
