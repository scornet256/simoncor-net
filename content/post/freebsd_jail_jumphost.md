---
title: 'FreeBSD - Jail - Secure Jumphost'
description: "FreeBSD"
date: "2020-10-20"
---

The goal is to create a limited jail using rbash and securing it so it can only accept secure SSH sessions. It should only be used as an SSH jumphost to connect further. It should therefor not be possible to create, use or install other code in this limited environment.

All commands are executed as root inside the jail, unless specified otherwise.

# FreeBSD jail
Create a jail and connect to the console.
```
[simon@host ~]$ sudo ezjail-admin create bastion 'bridge0|10.0.0.10'
[simon@host ~]$ sudo ezjail-admin console bastion
```
Install `bash`.
```
# pkg install bash
```

# OpenSSH-Portable
Install `openssh-portable`.
```
# pkg install openssh-portable
```
Configure `rc.conf`.
```
# sysrc sshd_enable=NO
# sysrc openssh_enable=YES
```

Check only what the current best practices are regarding the full OpenSSH daemon configuration.
For example check; https://infosec.mozilla.org/guidelines/openssh

Make sure the daemon only listens to the assigned IP for this jail. And make sure the firewall running on the host accepts incoming and outgoing SSH connections.

```
# cat /usr/local/etc/sshd
...
ListenAddress 10.0.0.10
...
```

Stop and start the services.
```
# service sshd stop
# service openssh start
```


# User
Create a default `user` and make sure the `user` has the `/usr/local/bin/rbash` shell configured.
```
# mkdir /usr/home/user/bin
```
Symlink the only required binaries into this directory.
```
# ln -s /usr/local/bin/ssh /usr/home/user/bin/ssh
```
Create bash profile.
```
# cat /usr/home/user/.bash_profile
PATH=$HOME/bin
export PATH
```

Make sure the permissions are so that the user cannot modify its own `.(bash_)profile` files.
```
# chown root:user .bash_profile .profile
```

Remove also all unused <shell>rc files like cshrc, shrc, etc.
```
# rm .cshrc .shrc ...
```

Create `.ssh` folder and fill `authorized_keys` file (optional).
```
# mkdir /usr/home/user/.ssh
# echo "your_public_key_here" >> /usr/home/user/.ssh/authorized_keys
# chown -R user:user /usr/home/user/.ssh
# chmod -R 700 /usr/home/user/.ssh
```

User directory can look like this.
```
[user@bastion ~]$ ls -al
total 3
drwxr-xr-x  4 user  user   5 Oct 20 11:24 .
drwxr-xr-x  4 root  wheel  4 Oct 19 11:59 ..
-rw-r--r--  1 root  user  43 Oct 19 14:09 .bash_profile
drwx------  2 user  user   5 Oct 19 12:40 .ssh
drwxr-xr-x  2 user  user   3 Oct 19 14:21 bin
```

# Result
 - FreeBSD Jail with latest packaged version of OpenSSH-Portable
 - Commands are unavailable and absolute paths are not allowed.
 - The `$PATH` variable is read-only.
 - The `.bash_profile` file is read-only for the user.
 - Only some bash functions + the `ssh` binary is available for the user.

```
[user@bastion ~]$ ls
-rbash: ls: command not found

[user@bastion ~]$ /bin/ls  
-rbash: /bin/ls: restricted: cannot specify `/' in command names

[user@bastion ~]$ export PATH=/usr/bin
-rbash: PATH: readonly variable

[user@bastion ~]$
!          break      continue   else       fg         in         pushd      shopt      true       while
./         builtin    coproc     enable     fi         jobs       pwd        source     type       {
:          caller     declare    esac       for        kill       read       ssh        typeset    }
[          case       dirs       eval       function   let        readarray  suspend    ulimit     
[[         cd         disown     exec       getopts    local      readonly   test       umask      
]]         command    do         exit       hash       logout     return     then       unalias    
alias      compgen    done       export     help       mapfile    select     time       unset      
bg         complete   echo       false      history    popd       set        times      until      
bind       compopt    elif       fc         if         printf     shift      trap       wait
```
