# ASIC design environment
## Common shell commands
### tcsh
1. **echo $0**: show which shell is currently used
2. **lsb_release -a**: show the detailed info. of current os
3. **chsh -s /usr/bin/bash**: set the login shell as Bash
4. **!\***: return the paras of last cmd
5. **printenv**: Print all or part of envs

### Bash
1. **\ps**：use `ps`'s original func.
2. **ps | grep simv | awk '{print $1}' | xargs kill -9**: clean processes in bulk
3. **ps -ef |grep defunct | awk '{print $2 “ ” $3}' |xargs kill -9**: clean zombie processes in bulk
4. **cd -**: change into former dir
5. **uname -a**: check sys info
6. **lsload**(IBM Spectrum LSF): display the current load levels of the cluster
7. ssh -X xx: switch to server xx
8. **ln -s file filesoft**: create soft links
9. **find /demo -name "*.v" | xargs cat | grep -v ^$|wc -l**: Count the number of lines in all files ending with ".v" in the demo directory (excluding blank lines)
10. **module av mathwork**: check module "mathwork" is available or not
11. **module add xx**: load module xx
12. **module li**: show loaded modules
13. **acroread、yozo、soffice**: open pdf file
14. **ls | grep -v xx | xargs rm -rf**：delete all files except xx
15. **Ctrl + z**：suspend process, can be recovered using `bg` or `fg` in the background or foreground
16. **Ctrl + c**：end process
17. **jobs**：show suspended process, can be killed further using `kill %n`
18. **du -sh \***: show files' size in cur dir recursively.
19. **grep -r xxx .**: search string xxx recursively
20. **firefox xxx.html**: open html doc
21. **which**: show the full path of (shell) commands
22. **top**: display processes（exit with `Ctrl + c`）
23. **“ ”**：illegal characters contained in file names should be enclosed in double quotes
24. **!prefix_cmd**: run the most recent cmd matched in the .history file
25. **bsub -Is "task"**(IBM Spectrum LSF): allocate servers to tasks automatically
26. **bjobs -w**(IBM Spectrum LSF): check which server the task is allocated to
27. **module load/unload**: switch EDA tool version
28. **ls -R**: list files recursively
29. **kill -9**: kill process forcibly
30. **find . -type f -ctime -1| xargs ls –l**: find files modified within one day

### .tcshrc
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

## Common Vim commands
1. **:/string\c**: case insensitive matching
2. **:/string\C**: case sensitive matching
3. **ggVG**: select all
4. **=**: left alignment
5. **f+char**: move the cursor to the next char
6. **F+char**: move the cursor to the last char
7. **dw**: delete a word from current position
8. **DEL**: delete a char
9. **Use a certain row as the prefix of multiple rows**: copy the row multiple times and place them with col operation. 
10. **y+6400**:select 6400 rows
11. **''**: back to last position
12. **:r !seq 1 20** or **:r !seq 20 -1 1**: generate sequence
13. **:20,$d**: delete from 20th row to end
14. **c+$**: delete from current position to line end
15. **yw**: copy a word
16. **.**: repeat last operation(usually combined with 17)
17. **cw**: change word
18. **".**: back to last modification
19. **V**: select multiple lines
20. **CTRL+n**: auto completion
21. **noh**: cancel highlight
22. **gf**: go to file; **CTRL+^**: go back to last file
23. **CTRL+v**: col operation; **I+ESC**: insert; **d+ESC**: delete; **c+ESC**: change
24. **:e**: reload file
25. **:/xx** or **\*xx**: search xx
26. **\<int\>**: full word match
27. **:%s/foo/bar/g(c)**: global search and replace
28. **50p**: plaste 50 times
29. **yy+p**: copy and plaste; **dd+p**: cut and plaste
30. **:u** or **u**: undo; **CTRL+r**: redo
31. **:vsp xx.v**: col split; **CTRL+w+w**: switch window
32. **ngg** or **:n**: go to line n; **gg**: go to head; **G**: go to end
33. **:2,3>** or **shift+>+>** or **v+>+>**: indentation; **:2,3<** or **shift+<+<** or **v+<+<**: anti-indentation; 

## Manual Software Installation and Uninstallation
### Install
1. tar -xvzf xxx.tar.gz;
2. check INSTALL file for install instructions;
### Uninstall
1. if installed with non-root account (installed into ~): just remove relevant dirs/files;
2. if installed with root account:
   1. back into the dir where you ran ```./configure``` and ```make``` before, and run ```make uninstall```;
   2. if i doesn't work (Makefile not correctly written), try ```checkinstall``` which allows you
       to build from source code, but have the packages tracked by apt.
