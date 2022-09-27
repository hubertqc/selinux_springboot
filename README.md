SELinux policy module for Springboot applications
==========================================================
https://github.com/hubertqc/selinux_springboot

## Introduction

This SELinux policy module aims to prevent a Springboot (daemon/background) application to
 perform unexpected actions on a Linux host.

It should be used when the Springboot application is possibly exposed to a world full of
 *bad guys*.

The module should prevent the application from going rogue, and should the application be
compromised by an attacker this SELinux policy module should help limit the damage on the
system on which the Springboot application is running.

When used correctly, this SELinux policy module will make the Springboot application(s)
run on the host in the dedicated `springboot_t` SELinux domain.

## How to use this SELinux module

Once the SELinux policy module is compiled and installed in the running Kernel SELinux
 policy, a few actions must be taken for the new policy to apply to the Springboot
 application(s).
 
The policy can be adjusted with a handfull of SELinux booleans.

### Filesystem labelling
This SELinux policy module SELinux file context definitions are based on the Filesystem 
Hierarchy Standards [https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard].

The root for the Springboot installation is expected to be /opt/springboot.
The root for log files of the Springboot application(s) is expected to be
 /var/log/springboot.

A typical directory layout for the Springboot application `my_app` would be:

```
/opt/springboot/my_app
                      \ conf
                      \ lib
                      \ keys
                      
/var/log/springboot/my_app

/var/run/springboot/my_app

/srv/springboot/my_app
                      \ cache
                      \ work
                      \ dynlib
                      
```

Files with `.so` and `.jar` extensions under the /opt/springboot and /srv/springboot trees
will be assigned the *Springboot library* SELinux type.

Files with `.jks`, `.jceks`, `.p12` or `.pkcs12`extensions placed in a `conf`or
`properties` directory under /opt/springboot will be assigned the *Springboot 
authentication/credentials* SELinux type. All files located in a `keys`directory under
 /opt/springboot will be assigned the same SELinux type.


Should you prefer to used a different directory structure, you should consider using
SELinux fcontext equivalence rules to map your specifics to the filesystem layout expected
in the policy module, using the `semanage fcontext -a -e ORIGINAL CUSTOMISATION` command.

### Networking

#### Listen port
The TCP port the Springboot application binds and listens to MUST be assigned the
 `springboot_port_t` SELinux type.
 
#### Monitoring port
If the Springboot application needs to bind and listen to a specific port to offer
monitoring metrics and information, then this TCP port should be assigned the
`springboot_monitoring_port_t` SELinux type.


Then, (host)local processes need to be granted the permission to connect to the monitoring
port using the policy module interface `springboot_monitor()` with the SELinux domain
prefix as the only argument.

Example: `springboot_monitoring_port_t(zabbix)` to Allow Zabbix to connect to the
monitoring port of the Springboot application.

#### Services consumed by the Springboot application
If the application needs to connect to services such as databases, directory servers, ...
the SELinux type of these services network port must be allowed for the Springboot
application to connect to.
One of the `springboot_allow_connectto` or `springboot_allow_consumed_service` interfaces
should be used with the prefix name for the service as the only argument.

Examples:
- `springboot_allow_connectto(postgresql)` to allow the Springboot application to connect to a PostgreSQL database
- `springboot_allow_connectto(ldap)` to allow connection to LDAP directory services.

### SELinux booleans

#### allow_springboot_http_connect      (default: `true`)
When switch to `true`this boolean allows the Springboot application to connect to remote
HTTP/HTTPS ports (locally assigned the `http_port_t` SELinux type).

#### allow_springboot_other_connect      (default: `true`)
When switch to `true`this boolean allows the Springboot application to connect to remote
other Springboot application ports (locally assigned the `springboot_port_t` SELinux type).

#### allow_springboot_dynamic_libs		(default: `false`)
When switched to `true`, this boolean allows the Springboot application to create and use
(execute) SO libraries and JAR files under the /srv/springboot/.../dynlib directory.
Use with care, i.e. only when strictly required, as this would allow a compromised
Springboot application to offload arbitrary code and use it.

#### allow_springboot_purge_logs		(default: `false`)
When switched to `true`n, this boolean allows the Springboot application to delete its log
files. It can be useful for log file rotation, but it can also be useful for attackers who
would like to clean after themselves and remove traces of their actions...

#### allow_webadm_read_springboot_files		(default: `false`)
Users running with the `webadm_r`SELinux role and`webadm_t`domain are granted the
permissions to browse the directories of the Springboot application and the permission to
stop and start the Springboot application **systemd** services, as well as querying their
status.

When switched to `true`, this boolean allows such users additional permissions to read the 
contents of Springboot application files: log, configuration, temp and transient/cache
files.

#### allow_sysadm_write_springboot_files	(default: `false`)
When switched to `true`, this boolean allows users running with the `sysadm_r`SELinux role
and`sysadm_t`domain to:
- fully manage temporary files,
- delete and rename log files,
- delete and rename transient/cache files,
- modify the contents of configuration files,

Otherwise, such users are only granted read permissions on all Springboot application
files, except authentication/credentials files.
 

### Starting the Springboot application

The Springboot application should always and ony be started as a **systemd** service using
the`systemctl` command.

The service or target unit files MUST be located in /etc/systemd/system or in
/lib/systemd/system, the file name MUST start with `springboot`.
Directories to tune or override unit behaviour are supported.
Template/instantiated units are supported provided the master file is named
`springboot@.service`.

The script(s) used to start or stop the Springboot application MUST be located in the 
/opt/springboot/service/ directory. The /opt/springboot/bin/springboot_service file name
is also supported.

### Running multiple Springboot appplications on the same host

TO DO


## Disclaimer

The code of the this SELinux policy module is provided AS-IS. People and organisation
willing to use it must be fully aware that they are doing so at their own risks and
expenses.

The Author(s) of this SELinux policy module SHALL NOT be held liable nor accountable, in
 any way, of any malfunction or limitation of said module, nor of the resulting damage, of
 any kind, resulting, directly or indirectly, of the usage of this SELinux policy module.

It is strongly advised to always use the last version of the code, to check for the 
compatibility of the platform where it is about to be deployed, to compile the module on
the target specific Linux distribution and version where it is intended to be used.

Finally, users should check regularly for updates.
