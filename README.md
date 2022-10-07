# often-used-shell-cmds
Often used shell commands in Unix-like operating system.
## tcsh
1. **echo $0**: show which shell is currently used;
2. **lsb_release -a**: show the detailed info. of current os;
3. **chsh -s /usr/bin/bash**: set the login shell as Bash
4. **!\***: return the paras of last cmd.
5. **printenv**: Print all or part of envs.

## Vim
1. **\string\c**: search string regardless of case

## Manual Software Installation and Uninstallation:
### Install
1. tar -xvzf xxx.tar.gz;
2. check INSTALL file for install instructions;
### Uninstall
1. if installed with non-root account (installed into ~): just remove relevant dirs/files;
2. if installed with root account:
   1. back into the dir where you ran ```./configure``` and ```make``` before, and run ```make uninstall```;
   2. if i doesn't work (Makefile not correctly written), try ```checkinstall``` which allows you
       to build from source code, but have the packages tracked by apt.      
