# ASIC development environment

## tcsh
1. **echo $0**: show which shell is currently used;
2. **lsb_release -a**: show the detailed info. of current os;
3. **chsh -s /usr/bin/bash**: set the login shell as Bash
4. **!\***: return the paras of last cmd.
5. **printenv**: Print all or part of envs.

## .tcshrc
```
#==========================EDA Environment==========================
#--------------------------SYNOPSYS---------------------------------
setenv Synopsys_Dir /ihp/ihpusr/synopsys

#Verdi
setenv Verdi_HOME $Synopsys_Dir/verdi201906/verdi/P-2019.06-SP2
set path=($Verdi_HOME/bin $path) 

#SpyGlass
setenv SPYGLASS_HOME $Synopsys_Dir/spyglass2012/SPYGLASS_HOME
set path=($SPYGLASS_HOME/bin $path) 

#VCS MX
setenv VCSMX_HOME $Synopsys_Dir/vcsmx201606
set path=($VCSMX_HOME/bin $path)

#--------------------------CADENCE---------------------------------
setenv Cadence_Dir /ihp/ihpusr/cadence

#Xcelium
setenv Xcelium_HOME $Cadence_Dir/xcelium2009/tools/
set path=($Xcelium_HOME/bin $path)

#--------------------------SOFTWARE/LIB--------------------------------- 
#PERL5LIB: IO/Tee
setenv PERL5LIB /home/dedong.zhao/tools/lib/perl5/share/perl5
 
#SystemC
setenv SYSTEMC_HOME /home/dedong.zhao/tools/systemc-2.3.0a/lib-linux64
set path=($SYSTEMC_HOME $path)
 
#==========================Alias==========================
alias l 'ls -la'
alias g vim
alias s 'source ~/.tcshrc'
alias c clear
alias r 'rm -rf'
alias eda 'cd /ihp/ihpusr/synopsys'
alias csh 'vim ~/.tcshrc'
alias . 'cd ..'
alias .. 'cd ../..'
alias ... 'cd ../../..'
alias cd 'cd \!*; l'
alias find 'find . -name !*'
alias grep 'grep -r \!* .'
alias nna 'cd /home/dedong.zhao/nvdla'
```
                                               
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
