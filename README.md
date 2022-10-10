# ASIC development environment

## tcsh
1. **echo $0**: show which shell is currently used
2. **lsb_release -a**: show the detailed info. of current os
3. **chsh -s /usr/bin/bash**: set the login shell as Bash
4. **!\***: return the paras of last cmd
5. **printenv**: Print all or part of envs

## Bash
1. **\ps**：use `ps`'s original func.
2. **ps | grep simv | awk '{print $1}' | xargs kill -9**: clean processes in bulk
3. **ps -ef |grep defunct | awk '{print $2 “ ” $3}' |xargs kill -9**: clean zombie processes in bulk
4. **cd -**: change into former dir
5. **uname -a**: check sys info
6. lsload: 查询当前各服务器负载
7. ssh -X xx: 切换服务器到xx
8. **ln -s file filesoft**: create soft links
9. **find /demo -name "*.v" |xargs cat|grep -v ^$|wc -l**：统计demo目录下以".v"结尾的所有文件的行数（不包括空行）
10. module av mathwork: 查看matlab模块是否可用
11. module add xx: 加载xx模块
12. module li: 查看已加载的模块
13. acroread、yozo、soffice: 在U网打开pdf
14. ls | grep -v xx | xargs rm -rf：删除除了xx之外的所有文件
15. CTRL + z：挂起进程，可以使用bg 或 fg 在后台或者前台恢复进程
16. CTRL + c：结束进程
17. jobs：查看被挂起的进程，可以进一步使用kill  %n 杀掉被挂起的进程
18. du -sh *: 递归查看当前文件夹中所有文件的大小
19. grep -r xxx . : 递归搜索xxx字符串
20. firefox xxx.html: 打开html文档
21. which: 查找文件
22. top: 任务管理（Ctrl + C 退出）
23. “”：当文件名中含有非法字符时，可以用双引号括起来
24. !prefix_cmd: 执行history列表中匹配到的最近一条cmd指令；
25. bsub -Is "task": 在服务器集群中自动分配服务器执行任务task；
26. bjobs -w: 查看任务分配在哪台服务器； 
27. module load/unload: 切换eda工具版本。
28. ls -R: 递归列出文件
29. kill -9: 强制立即杀死进程

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
