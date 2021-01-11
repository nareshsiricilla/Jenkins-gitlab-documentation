## Hardening (done)

Hardening of AMI

### Password policy or configure LDAP.

Update System Packages

```
sudo apt update -y && sudo apt upgrade -y
```

Install pam_cracklib Package

```
sudo apt install -y libpam-pwquality
```

Edit the /etc/pam.d/common-password file

```
 sudo vim /etc/pam.d/common-password
```

replaced the line no.25 with following

```
password        requisite                       pam_pwquality.so retry=3 minlen=14 dcredit = -1 ucredit = -1 ocredit = -1 lcredit = -1
```

### Implement Audit.d

```
apt-get install auditd
 service auditd stop
```

Open /etc/audit/rules.d/audit.rules and replaced with following.

```
# This file contains the auditctl rules that are loaded
# whenever the audit daemon is started via the initscripts.
# The rules are simply the parameters that would be passed
# to auditctl.

# First rule - delete all
-D

# Increase the buffers to survive stress events.
# Make this bigger for busy systems
-b 8192

# Feel free to add below this line. See auditctl man page

-a exclude,always -F msgtype=CWD

-a always,exit -S all -F dir=/etc -F perm=w -k system_object
-a always,exit -S all -F dir=/boot -F perm=w -k system_object
-a always,exit -S all -F dir=/usr/lib -F perm=w -k system_object
-a always,exit -S all -F dir=/bin -F perm=w -k system_object
-a always,exit -S all -F dir=/lib -F perm=w -k system_object
-a always,exit -S all -F dir=/lib64 -F perm=w -k system_object
-a always,exit -S all -F dir=/sbin -F perm=w -k system_object
-a always,exit -S all -F dir=/usr/bin -F perm=w -k system_object
-a always,exit -S all -F dir=/usr/sbin -F perm=w -k system_object
-a always,exit -S all -F dir=/home -F perm=w -k system_object

-a always,exit -F euid=0 -F perm=x -k root_action

-w /var/log/audit/ -p w -k audit_access
-w /var/log/auth.log -p w -k audit_access
-w /var/log/syslog -p w -k audit_access

-a always,exit -F arch=b64 -S setuid -Fa0=0  -k elevation
-a always,exit -F arch=b32 -S setuid -Fa0=0  -k elevation
-a always,exit -F arch=b64 -S execve -C uid!=euid -F euid=0  -k elevation
-a always,exit -F arch=b32 -S execve -C uid!=euid -F euid=0  -k elevation

-w /etc/ntp.conf -p w

#-w /home/ace/ror/config/local_env.yml -p w -k ace_access
-w /etc/apache2/sites-enabled/ror.conf -p w -k ace_access
service auditd start
```

### Harden SSH

Replaced /etc/ssh/sshd_config contents with following.

```
#       $OpenBSD: sshd_config,v 1.101 2017/03/14 07:19:07 djm Exp $

# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/usr/bin:/bin:/usr/sbin:/sbin

# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override the
# default value.

#Port 22
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

#HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_ecdsa_key
#HostKey /etc/ssh/ssh_host_ed25519_key

# Ciphers and keying
#RekeyLimit default none

# Logging
#SyslogFacility AUTH
LogLevel INFO

# Authentication:

LoginGraceTime 20
PermitRootLogin no
StrictModes yes
MaxAuthTries 2
MaxSessions 2

#PubkeyAuthentication yes

# Expect .ssh/authorized_keys2 to be disregarded by default in future.
#AuthorizedKeysFile     .ssh/authorized_keys .ssh/authorized_keys2

#AuthorizedPrincipalsFile none

#AuthorizedKeysCommand none
#AuthorizedKeysCommandUser nobody

# For this to work you will also need host keys in /etc/ssh/ssh_known_hosts
#HostbasedAuthentication no
# Change to yes if you don't trust ~/.ssh/known_hosts for
HostbasedAuthentication no
IgnoreUserKnownHosts no
# Don't read the user's ~/.rhosts and ~/.shosts files
#IgnoreRhosts yes

# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication no
PermitEmptyPasswords no

# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
ChallengeResponseAuthentication no

# Kerberos options
#KerberosAuthentication no
#KerberosOrLocalPasswd yes
#KerberosTicketCleanup yes
#KerberosGetAFSToken no

# GSSAPI options
#GSSAPIAuthentication no
#GSSAPICleanupCredentials yes
#GSSAPIStrictAcceptorCheck yes
#GSSAPIKeyExchange no

# Set this to 'yes' to enable PAM authentication, account processing,
# and session processing. If this is enabled, PAM authentication will
# be allowed through the ChallengeResponseAuthentication and
# PasswordAuthentication.  Depending on your PAM configuration,
# PAM authentication via ChallengeResponseAuthentication may bypass
# the setting of "PermitRootLogin without-password".
# If you just want the PAM account and session checks to run without
# PAM authentication, then enable this but set PasswordAuthentication
# and ChallengeResponseAuthentication to 'no'.
UsePAM yes

#AllowAgentForwarding yes
#AllowTcpForwarding yes
#GatewayPorts no
X11Forwarding no
#X11DisplayOffset 10
#X11UseLocalhost yes
#PermitTTY yes
PrintMotd no
PrintLastLog yes
#TCPKeepAlive yes
#UseLogin no
PermitUserEnvironment no
#Compression delayed
ClientAliveInterval 300
ClientAliveCountMax 0
UseDNS no
#PidFile /var/run/sshd.pid
MaxStartups 10:30:60
#PermitTunnel no
#ChrootDirectory none
#VersionAddendum none
Macs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512,hmac-sha2-256
# no default banner path
#Banner /etc/issue.net

# Allow client to pass locale environment variables
AcceptEnv LANG LC_*

# override default of no subsystems
Subsystem       sftp    /usr/lib/openssh/sftp-server

# Example of overriding settings on a per-user basis
#Match User anoncvs
#       X11Forwarding no
#       AllowTcpForwarding no
#       PermitTTY no
#       ForceCommand cvs server
```

Replaced /etc/ssh/ssh_config contains with following.

```
# This is the ssh client system-wide configuration file.  See
# ssh_config(5) for more information.  This file provides defaults for
# users, and the values can be changed in per-user configuration files
# or on the command line.

# Configuration data is parsed as follows:
#  1. command line options
#  2. user-specific file
#  3. system-wide file
# Any configuration value is only changed the first time it is set.
# Thus, host-specific definitions should be at the beginning of the
# configuration file, and defaults at the end.

# Site-wide defaults for some commonly used options.  For a comprehensive
# list of available options, their meanings and defaults, please see the
# ssh_config(5) man page.

Host *
#   ForwardAgent no
#   ForwardX11 no
#   ForwardX11Trusted yes
#   PasswordAuthentication yes
#   HostbasedAuthentication no
#   GSSAPIAuthentication no
#   GSSAPIDelegateCredentials no
#   GSSAPIKeyExchange no
#   GSSAPITrustDNS no
#   BatchMode no
#   CheckHostIP yes
#   AddressFamily any
#   ConnectTimeout 0
#   StrictHostKeyChecking ask
#   IdentityFile ~/.ssh/id_rsa
#   IdentityFile ~/.ssh/id_dsa
#   IdentityFile ~/.ssh/id_ecdsa
#   IdentityFile ~/.ssh/id_ed25519
#   Port 22
#   Protocol 2
    Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes256-ctr
    MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512,hmac-sha2-256
#   EscapeChar ~
#   Tunnel no
#   TunnelDevice any:any
#   PermitLocalCommand no
#   VisualHostKey no
#   ProxyCommand ssh -q -W %h:%p gateway.example.com
#   RekeyLimit 1G 1h
    SendEnv LANG LC_*
    HashKnownHosts yes
    GSSAPIAuthentication yes
```