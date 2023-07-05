# ASIC design environment
## EDA
### Xcelium
1. **start and load project**: `xrun -f $PROJ_DIR/proj.vc -access +rwc -gui&`
2. **improve font size**: `cp ${XCEILIUM_ROOT}/share/cdssetup/simvision/app-defaults/share/cdssetup/simvision/app-defaults/SimVision ~/.simvision/Xdefaults` and `vim  ~/.simvision/Xdefaults`
3. **save the signal list in Waveform window**: `File -> Save Command Script` and `Source Command Script`
4. **reload design in Waveform window**: `Simulation -> Reinvoke Simulator` 
### Genus
1. **generate synthesis script**: `write_template -simple -outfile tempname`
### Verdi
1. **start and load project**: `verdi -f $PROJ_DIR/proj.vc&`
### DC-Based Synthesis Flow
1. **design_syn_flow.tcl**:
```
#--------------------------Specify Libraries--------------------------
set mem_link_library "$MEM_LINK_PATH/T22SRF2HD256X16M2K2H_ssgwct0p81vn40c.db"
set target_library "$TAR_PATH/tcbn22ulpbwp7t30p140ssg0p81v125c_ccs.db"
set link_library "* $target_library $mem_link_library"
#set search_path "$TAR_PATH $MEM_LINK_PATH"

#--------------------------Prepare Filelist---------------------------
set FILE_LIST ""
set f [open "./spread.fl" r]
while {![eof $f]} {
    gets $f line
    append FILE_LIST "$line "
}
echo $FILE_LIST
close $f

#--------------------------Read Designs------------------------------
set TOP design_top
analyze -format sverilog $FILE_LIST
elaborate $TOP

#------------------------Set Current Design--------------------------
#current_design $TOP(auto)

#--------------------------Link Designs------------------------------
#link(auto)

#-------------------------------SDC----------------------------------
source design_sdc.tcl

#--------------------Map and Optimize the Design---------------------
compile_ultra -no_autoungroup -incremental

#---------------Check the Synthesized Design for Consistency---------
check_design -summary > ./check_design.rpt

#---------------------Report Timing and Area-------------------------
report_timing > ./timing.rpt
report_area -hierarchy > ./area.rpt

#----------------------Save Design Database--------------------------
change_names -rules sverilog -hierarchy
write_file -format verilog -hierarchy -output design_netlist.v
write_sdc design_sdc_post_syn
```
### Synopsys Design Constraints
1. **design_sdc.tcl**:
```
#==================================Env Vars===================================
set TIME_UNIT 1
set CYCLE200M [expr 5 * $TIME_UNIT]
set CYCLE8M [expr 125 * $TIME_UNIT]

#==================================Design Env=================================
#------------------------------Operating Conditions---------------------------
#GUIDANCE: the worst case: P(1), V(LOW), T(HIGH) for stricter constraint.
#set_operating_conditions -max ssg0p81v125c(default)

#-------------------------System Interface Characteristics--------------------
#An AND2 cell with minimum drving strength for stricter constraints.
set_driving_cell -lib_cell AN2D0BWP7T30P140 [all_inputs]
#If real clock, set infinite drive strength to avoid automatic buffer insertion.
set_drive 0 [get_ports[list clk0 clk1]]
set_load -pin_load 0.02 [all_outputs]
#set_fanout_load 4 [all_outputs]

#---------------------------------Wire Load Model-----------------------------
#GUIDANCE: WLM selection does not matter, it is not accurate.
#set auto_wire_load_selection true(default)

#==================================Design Rule Constr=========================
#GUIDANCE: use the default
#set_max_transition  0.25 [current_design]
#set_max_fanout      32   [current_design]
#set_max_capacitance 0.5  [current_design]

#==============================Design Optimiz Constr=========================
#--------------------------------Clock Definition------------------------------
create_clock -name clk0 -period $CYCLE200M [get_ports clk0]
create_clock -name clk1 -period $CYCLE8M [get_ports clk1]

set_clock_uncertainty -hold 0.053 [all_clocks]
set_clock_transition 0.15 [all_clocks]
set_input_transition 0.2 [remove_from_collection [all_inputs] [all_clocks]]

#--------------------------------I/O Constraint-----------------------------
#rst_ports
set rst_inputs [get_ports [list \
    rst0 \
    rst1 \
]]
set_ideal_network $rst_inputs

#ports in clk0 domain
set clk0_ports [get_ports [list \
    clk0_port0 \
    clk0_port1 \
]]

set clk0_inputs [get_ports $clk0_ports -filter "port_direction == in"]
set clk0_outputs [get_ports $clk0_ports -filter "port_direction == out"]

set_input_delay -max [expr $CYCLE200M * 0.6] -clock [get_clocks clk0] $clk0_inputs -add_delay
set_output_delay -max [expr $CYCLE200M * 0.3] -clock [get_clocks clk0] $clk0_outputs -add_delay

#---------------------------------Timing Exceptions-----------------------------
set false_ports [get_ports [list \
    false_port0 \
    false_port1 \
]]
set false_inputs [get_ports $false_ports -filter "port_direction == in"]
set false_outputs [get_ports $false_ports -filter "port_direction == out"]

set_false_path -from [get_ports $false_inputs]
set_false_path -to [get_ports $false_outputs]
```
## Shell
1. **\ps**：use original func of cmd `ps`.
2. **ps | grep simv | awk '{print $1}' | xargs kill -9**: clean processes in bulk
3. **ps -ef |grep defunct | awk '{print $2 “ ” $3}' |xargs kill -9**: clean zombie processes in bulk
4. **cd -**: change into former dir
5. **uname -a**: check sys info
6. **lsload**(IBM Spectrum LSF): display the current load levels of the cluster
7. **ssh -X xx**: switch to server xx
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
31. **echo $0**: show which shell is currently used
32. **lsb_release -a**: show the detailed info. of current os
33. **chsh -s /usr/bin/bash**: set the login shell as Bash
34. **!\***: return the paras of last cmd
35. **printenv**: Print all or part of envs

## Vim
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

## gVim Verilog Plugin
1. automatic verilog: [https://github.com/HonkW93/automatic-verilog](https://github.com/HonkW93/automatic-verilog)

## Manual Software Installation and Uninstallation
### Install
1. tar -xvzf xxx.tar.gz
2. cd xxx
3. ./configure --prefix= tools/xxx
4. make; make install
### Uninstall
1. if installed with non-root account (installed into ~): just remove relevant dirs/files;
2. if installed with root account:
   1. back into the dir where you ran ```./configure``` and ```make``` before, and run ```make uninstall```;
   2. if i doesn't work (Makefile not correctly written), try ```checkinstall``` which allows you
       to build from source code, but have the packages tracked by apt.
