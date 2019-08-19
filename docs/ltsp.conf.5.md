## NAME
**ltsp.conf** - client configuration file for LTSP

## SYNOPSIS
The LTSP client configuration file is placed at `/etc/ltsp/ltsp.conf`
and it losely follows the .ini format. It is able to control various
settings of the LTSP server and clients. After each ltsp.conf modification,
the `ltsp initrd` command needs to be run so that it's included in the
additional ltsp.img initrd that is sent when the clients boot.

## CREATION
To create an initial ltsp.conf, run the following command:

```shell
install -m 0600 /usr/share/ltsp/common/ltsp/ltsp.conf /etc/ltsp/ltsp.conf
```

Optionally, you may also change its owner from "root" to yourself, so that
you can edit that file with a visual editor (e.g. gedit) without running sudo.
Of course, replace _administrator_ below with your username:

```shell
chown administrator:administrator /etc/ltsp/ltsp.conf
```

## SYNTAX
Open and view the /etc/ltsp/ltsp.conf file that you just created, so that it's
easier to understand its syntax.

The configuration file is separated into sections:

 * The special [server] section is evaluated only by the ltsp server.
 * The special [common] section is evaluated by both the server and ltsp clients.
 * In the special [clients] section, parameters for all clients can be defined.
   Most ltsp.conf parameters should be placed here.
 * MAC address, IP address, or hostname sections can be used to apply settings
   to specific clients. Those support globs, for example [192.168.67.*].
 * It's also possible to group parameters into named sections like
   [crt_monitor] in the example, and reference them from other sections with
   the INCLUDE= parameter.
 * Advanced users may also use [applet/host] sections, for example
   [initrd-bottom/library*] would be evaluated by the `ltsp initrd-bottom`
   applet only for clients that have a hostname that starts with "library".

The ltsp.conf configuration file is internally transformed into a shell
script, so all the shell syntax rules apply, except for the sections headers
which are transformed into functions.

This means that you must not use spaces around the "=" sign,
and that you may write comments using the "#" character.

The `ltsp initrd` command does a quick syntax check by running
`sh -n /etc/ltsp/ltsp.conf` and aborts if it detects syntax errors.

## PARAMETERS
The following parameters are currently defined; an example is given in
each case.

**CRONTAB_x=**_"30 15 * * *  poweroff"_
: Add a line in crontab. The example powers off the clients at 15:30.

**DEBUG_LOG=**_0|1_
: Write warnings and error messages to /run/ltsp/debug.log. Defaults to 0.

**DEBUG_SHELL=**_0|1_
: Launch a debug shell when errors are detected. Defaults to 0.

**DNS_SERVER=**_"8.8.8.8 208.67.222.222"_
: Specify the DNS servers for the clients.

**FSTAB_x=**_"server:/home /home nfs 0 0"_
: All parameters that start with FSTAB_ are sorted and then their values
are written to /etc/fstab at the client init phase.

**HOSTNAME=**_"pc01"_
: Specify the client hostname.

**HOSTNAME_PREFIX=**_"client"_
: Specify the prefix for autogenerated client hostnames; defaults to "ltsp".

**HOSTNAME_SUFFIX=**_"ip|mac"_
: Specify the suffix for autogenerated client hostnames; defaults to "ip".

**HOSTS_x=**_"192.168.67.10 nfs-server"_
: All parameters that start with HOSTS_ are sorted and then their values
are written to /etc/hosts at the client init phase.

**INCLUDE=**_"other-section"
: Include another section in this section.

**KEEP_SESSION_SERVICES=**_"at-spi-dbus-bus"_
: Whitelist some session (user) services so that they're not deleted, even if
they're listed in MASK_SESSION_SERVICES. Space separated list.

**KEEP_SYSTEM_SERVICES=**_"apparmor ssh"_
: Whitelist some system services so that they're not deleted, even if
they're listed in MASK_SYSTEM_SERVICES. Space separated list.

**LOCAL_SWAP=**_0|1_
: Activate local swap partitions. Defaults to 1.

**MASK_SESSION_SERVICES=**_"ubuntu-mate-welcome"_
: Mask some session services that shouldn't be started on LTSP clients.
Space separated list. See /usr/share/ltsp/client/init/56-rm-services.sh
for the default. Setting MASK_SESSION_SERVICES in ltsp.conf adds to that list.

**MASK_SYSTEM_SERVICES=**_"teamviewerd"_
: Mask some system services that shouldn't be started on LTSP clients.
Space separated list. See /usr/share/ltsp/client/init/56-rm-services.sh
for the default. Setting MASK_SYSTEM_SERVICES in ltsp.conf adds to that list.

**POST_APPLET_x=**_"ln -s /etc/ltsp/xorg.conf /etc/X11/xorg.conf"_
: All parameters that start with POST_ and then have an ltsp client applet
name are sorted and their values are executed after the main function of
that applet. See the ltsp(8) man page for the available applets.
The usual place to run client initialization commands that don't need to
daemonize is POST_INIT_x.

**PRE_APPLET_x=**_"debug_shell"_
: All parameters that start with PRE_ and then have an ltsp client applet
name are sorted and their values are executed before the main function of
that applet.

**PWMERGE_SUR=**, **PWMERGE_SGR=**, **PWMERGE_DGR=**, **PWMERGE_DUR=**
: Normally, all the server users are listed on the client login screens
and are permitted to log in. To exclude some of them, define one or
more of those regular expressions. For more information, read
/usr/share/ltsp/client/login/pwmerge. For example, if you name your clients
pc01, pc02 etc, and your users a01, a02, b01, b02 etc, then the following
line only shows/allows a01 and b01 to login to pc01:
`PWMERGE_SUR=".*${HOSTNAME#pc}"`

**SEARCH_DOMAIN=**_"ioa.sch.gr"_
: A search domain to add to resolv.conf and to /etc/hosts. Usually provided
by DHCP.

**SERVER=**_"192.168.67.1"_
: The LTSP server is usually autodetected; it can be manually specified
if there's need for it.

**X_DRIVER=**"_vesa_"
**X_HORIZSYNC=**"_28.0-87.0_"
**X_MODELINE=**'_"1024x768_85.00"   94.50  1024 1096 1200 1376  768 771 775 809 -hsync +vsync_'
**X_MODES=**'"_1024x768" "800x600" "640x480"_'
**X_PREFERREDMODE=**"_1024x768_"
**X_VERTREFRESH=**"_43.0-87.0_"
**X_VIRTUAL**="_800 600_"
: If any of these parameters are set, the /usr/share/ltsp/client/init/xorg.conf
template is installed to /etc/X11/xorg.conf, while applying the parameters.
Read that template and consult xorg.conf(5) for more information.
The most widely supported method to set a default resolution is X_MODES.
If more parameters are required, create a custom xorg.conf as described in
the EXAMPLES section.

## EXAMPLES
If some clients need an custom xorg.conf file, create it in e.g.
`/etc/ltsp/xorg-nvidia.conf`, and put the following in ltsp.conf
to dynamically symlink it for those clients on boot:

```shell
[pc01]
INCLUDE=nvidia

[nvidia]
POST_INIT_LN_XORG="ln -sf ../ltsp/xorg-nvidia.conf /etc/X11/xorg.conf"
```

Since ltsp.conf is transformed into a shell script and sections into
functions, it's possible to do all kinds of fancy things, even to directly
include code. But it's usually best to keep it simple and put code in
separate scripts.

```shell
[clients]
# Set the root password to "qwer1234" for all clients.
# The password hash contains ' and $, making it hard to escape it,
# so use a "HEREDOC" instead.
{ POST_INIT_SET_ROOT_HASH=$(cat); } <<"EOF"
sed 's|^root:[^:]*:|root:$6$p2LdWE6j$PDd1TUzGvvIkj9SE8wbw1gA/MD66tHHlStqi1.qyv860oK47UnKcafSKqGp7cbgZUPlgyPv6giCVyCSCdJt1b0:|' -i /etc/shadow
EOF

[initrd-bottom/]
# Putting commands under [applet/] sections means that they will only be run
# by that specific ltsp applet.
# The following commands work around LP: #345374 bug for SiS clients.
test -d /sys/module/sis190 || return 0
ip link set dev "$DEVICE" mtu 1492
```