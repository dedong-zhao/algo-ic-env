# Environment for algorithm and IC development
- [EDA](#eda)
  * [Verdi](#verdi)
  * [Xcelium](#xcelium)
  * [Genus](#genus)
  * [Innovus](#innovus)
  * [DC-Based Synthesis Flow](#dc-based-synthesis-flow)
  * [Innovus-Based Implementation Flow](#innovus-based-implementation-flow)
  * [Voltus-Based Dynamic Power Analysis](#voltus-based-dynamic-power-analysis)
- [Linux](#linux)
  * [Shell](#shell)
  * [Vim](#vim)
  * [Vim Plugin](#vim-plugin)
- [C Programming](#c-programming)
  * [Makefile](#makefile)
  * [GDB](#gdb)
  * [Valgrind](#valgrind)
- [Git and SVN](#git-and-svn)
  * [Git](#git)
  * [SVN](#svn)
- [Msc](#msc)
  * [Writing C Extensions Using Python's C API](#writing-c-extensions-using-python-s-c-api)
  * [File Header Comment](#file-header-comment)
    + [VS Studio](#vs-studio)
    + [PyCharm](#pycharm)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

## EDA
### Verdi
1. **signal trace**: `put signal into search box(Ctrl + f) -> upper layer -> search`
2. **search specific module**: `source -> find scope`
3. **clean trace results**: `right click -> delete all`
4. **find instantiation location**: `double click the module`
5. **view import log in full screen**: `File -> View Import Log`
6. **common setting**:
   - **Font**: `General -> Font and Size`
   - **Editor**: `Editor -> Other`
   - **Code Folding**: `Source Code -> Code Folding`
   - **Code Trace**: `Trace -> General -> Trace View -> Show Top Level Ports`
7. **nWave hot keys**:
   - **b(begin)**: `move cursor to waveform start`
   - **e(end)**: `move cursor to waveform end`
   - **z(zoom out)**: `zoom out the waveform`
   - **Z(zoom in)**: `zoom in the waveform`
   - **Ctrl + Right Arrow**: `move right for half screen`
   - **Ctrl + Left Arrow**: `move left for half screen`
   - **c(color)**: `change the color, width and type of the waveform`
   - **f(full)**: `full waveform`
   - **x**: `show signal value at cursor`
   - **g(get)**: `get signal to show its waveform`
   - **l(last)**: `last view of the waveform`
   - **m(marker)**: `add marker`
   - **y**: `keep cursor at center, again to cancel`
   - **s**: `make cursor alignned to signal edge`
   - **count pulse number**: `select range -> View -> Signal Event Report`
   - **h(hierarchy)**:`show signal hierarchy`
   - **double click**: `locate signal in RTL`

### Xcelium
1. **start and load project**: `xrun -f $PROJ_DIR/proj.vc -access +rwc -gui&`
2. **improve font size**: `cp ${XCEILIUM_ROOT}/share/cdssetup/simvision/app-defaults/share/cdssetup/simvision/app-defaults/SimVision ~/.simvision/Xdefaults` and `vim  ~/.simvision/Xdefaults`
3. **save the signal list in Waveform window**: `File -> Save Command Script` and `Source Command Script`
4. **reload design in Waveform window**: `Simulation -> Reinvoke Simulator`
5. **zoom in/out waveform**: `Ctrl + Mouse Scroll Up/Down`
6. **shift waveform**: `left/right arrow`
7. **center cursor**: `Alt + c`

### Genus
1. **generate template script for synthesis flow**: `write_template -simple -outfile tempname`
2. **run genus**:
```
rm -rf genus* outputs* reports* fv
genus -files stdp_syn_flow.tcl -abort_on_error&
```
3. **fail and stop when issuing Errors**: `set_db / .fail_on_error_mesg true`

### Innovus
1. **generate template script for implementation flow**: `write_flow_template`
2. **fit, see the outline of the layout**: `f`
3. **zoom in selected area**: `mouse right click –> hold –> drag –> release`
4. **undo last operation**: `undo`

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
2. **design_sdc.tcl**:
```
#==================================Env Vars===================================
# time unit is ps for GF22nm technology
set TIME_UNIT 1 
set CYCLE1000G  [expr 1 * $TIME_UNIT]
set CYCLE500G   [expr 2 * $TIME_UNIT]
set CYCLE200G   [expr 5 * $TIME_UNIT]
set CYCLE8G     [expr 125 * $TIME_UNIT]
set CYCLE4G     [expr 250 * $TIME_UNIT]
set CYCLE2G     [expr 500 * $TIME_UNIT]
set CYCLE1G     [expr 1000 * $TIME_UNIT]
set CYCLE500M   [expr 2000 * $TIME_UNIT]

#==================================Design Env=================================

#------------------------------Operating Conditions---------------------------
# GUIDANCE: the worst case: P(1), V(LOW), T(HIGH) for stricter constraint.
# set_operating_conditions -max ssg0p81v125c(default)

#-------------------------System Interface Characteristics--------------------
# for clocks, set infinite drive strength to avoid automatic buffer insertion
set_drive 0 [get_ports [list clk]]

# 1/resistance as drive, proportional to cell size
# should be small enough
# AND2 SC8T_AN2X0P5_CSC20R in GF22FDX
set_driving_cell -lib_cell SC8T_AN2X0P5_CSC20R [remove_from_collection [all_inputs] [all_clocks]]
# set_input_transition 0.2 [remove_from_collection [all_inputs] [all_clocks]]

# capacitance as load, proportional to cell size
# should be large enough
# D pin cap of SDFF SC8T_SDFFX1_CSC20R in GF22FDX
set_load -pin_load 0.48356 [all_outputs]
# set_fanout_load 4 [all_outputs]

#---------------------------------Wire Load Model-----------------------------
# GUIDANCE: WLM selection does not matter, it is not accurate.
# set auto_wire_load_selection true(default)

#==================================Design Rule Constr=========================
# GUIDANCE: use the default
# set_max_transition  0.25 [current_design]
# set_max_fanout      32   [current_design]
# set_max_capacitance 0.5  [current_design]

#==============================Design Optimiz Constr=========================
#--------------------------------Clock Definition------------------------------
create_clock -name clk0 -period $CYCLE200M [get_ports clk0]
create_clock -name clk1 -period $CYCLE8M [get_ports clk1]

# jitter + skew
# shoud only include jitter after CTS
set_clock_uncertainty -setup 0.1*$CYCLE500M [all_clocks]
set_clock_uncertainty -hold 0.05*$CYCLE500M [all_clocks]

# should be removed after CTS
set_clock_transition 0.01*$CYCLE500M [all_clocks]

#--------------------------------I/O Constraint-----------------------------
# rst_ports
set rst_inputs [get_ports [list \
    rst0 \
    rst1 \
]]
set_ideal_network $rst_inputs

# ports in clk0 domain
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
### Innovus-Based Implementation Flow
```
#==========================================================================================
# 1. Design Initialization
#========================================================================================== 
# 1.1 File -> Import Design -> Load -> OK

source ./inputs/stdp.globals
set init_design_uniquify 1 
setDesignMode -process 22
init_design

setPreference CmdLogMode 2
setMultiCpuUsage -localCpu 16 -cpuPerRemoteHost 8 -remoteHost 0 -keepLicense true

# 1.2 File -> Save/Restore Design
saveDesign stdp.enc
saveDesign stdp.enc -timingGraph
restoreDesign stdp.enc.dat stdp

#==========================================================================================
# 2. Floorplan
#==========================================================================================
# 2.1 Floorplan > Specify Floorplan > Advanced
# left bottom right top
floorPlan -site SC8T_104CPP_CMOS22FDX -r 0.812772133527 0.696316 10 10 10 10
# saveFPlan inputs/stdp.fp
# loadFPlan inputs/stdp.fp

#==========================================================================================
# 3. Pin Assignment
#==========================================================================================
# 3.1 Edit -> Pin Editor
    # - De-select "Group Bus"
    # - Location -> Spread -> From Center -> Spacing

# saveIOfile inputs/stdp.io
# loadIoFile inputs/stdp.io

#==========================================================================================
# 4. Power Plan
#==========================================================================================
# 4.1 Power -> Connect Global Nets
    # - Apply All

globalNetConnect VDD -type pgpin -autoTie -pin VDD -all -override -verbose
globalNetConnect VSS -type pgpin -autoTie -pin VSS -all -override -verbose

# 4.2 Power -> Power Planning -> Add Ring
    # - Spacing is dependent on Width. 
    # - Plan power ring in highest metal layers for low resistance.
    # - Center in channel

addRing -nets {VDD VSS} -layer {top LB bottom LB left QB right QB} -width 2 -spacing 2 -center 1

# Optional: Additional connections from power rings to power/ground rails in the core.
# 4.3 Power -> Power Planning -> Add Stripes
    # - "Width" and "Spacing" similar to power ring
    # - Use "Start" and "Stop" for easier stripe location

addStripe -nets {VDD VSS} -layer QB -direction vertical -width 2 -spacing 2 -number_of_sets 1 -start_from left -start_offset 5 -stop_offset 0

# VDD/VSS wires between rings and core power rails
# 4.4 Route -> Special Route
setSrouteMode -viaConnectToShape {ring}
sroute -nets {VDD VSS} -connect {corePin}

#remove all power preroutes(rings, stripes, rails) from the floor plan 
deleteAllPowerPreroutes

#==========================================================================================
# 5. Place
#==========================================================================================
# 5.1 Place -> Place Standard Cell
place_design -noprePlaceOpt

#==========================================================================================
# 6. Pre-CTS Timing Analysis and Optimization(Timing Violation Acceptable)
#==========================================================================================
setAnalysisMode -cppr both   
timeDesign -preCTS       -prefix preCTSSetup        -outDir ./timingReports/preCTSSetup
# optDesign  -preCTS -drv
optDesign  -preCTS       -prefix preCTSSetupOpt     -outDir ./timingReports/preCTSSetupOpt
optDesign  -preCTS -incr -prefix preCTSSetupIncrOpt -outDir ./timingReports/preCTSSetupIncrOpt

# unplaceAllInsts

#==========================================================================================
# 7. CTS
#==========================================================================================
create_ccopt_clock_tree_spec
# get_ccopt_clock_trees * # myCLK
set_ccopt_property target_max_trans 60 # 60ps
set_ccopt_property target_skew 20 # 20ps
ccopt_design

# report_ccopt_clock_trees -file ./outputs/stdp_clock_trees.rpt
# ctd_win

#==========================================================================================
# 8. Post-CTS Timing Analysis and Optimization
#==========================================================================================
set_interactive_constraint_modes [all_constraint_modes -active]
set_propagated_clock [all_clocks]
set_interactive_constraint_modes {}

# report_clocks
# get_property [all_clocks] is_propagated_clock

setAnalysisMode -cppr both

timeDesign -postCTS       -prefix postCTSSetup -outDir ./timingReports/postCTSSetup
timeDesign -postCTS -hold -prefix postCTSHold  -outDir ./timingReports/postCTSHold

optDesign -postCTS       -prefix postCTSSetupOpt     -outDir ./timingReports/postCTSSetupOpt
optDesign -postCTS -incr -prefix postCTSSetupIncrOpt -outDir ./timingReports/postCTSSetupIncrOpt
optDesign -postCTS -hold -prefix postCTSHoldOpt      -outDir ./timingReports/postCTSHoldOpt

# editDelete -type Regular

#==========================================================================================
# 9. Route
#==========================================================================================
# 9.1 Route -> NanoRoute -> Route
    # - viaOpt & - wireOpt 

routeDesign -globalDetail -viaOpt -wireOpt

#==========================================================================================
# 10. Post-Route Timing Analysis and Optimization
#==========================================================================================
# report_clock_timing -type summary

setAnalysisMode -cppr both -analysisType onChipVariation

timeDesign -postRoute       -prefix postRouteSetup -outDir ./timingReports/postRouteSetup
timeDesign -postRoute -hold -prefix postRouteHold  -outDir ./timingReports/postRouteHold

optDesign -postRoute       -prefix postRouteSetupOpt      -outDir ./timingReports/postRouteSetupOpt 
optDesign -postRoute -incr -prefix postRouteSetupIncrOpt  -outDir ./timingReports/postRouteSetupIncrOpt
optDesign -postRoute -hold -prefix postRouteHoldOpt       -outDir ./timingReports/postRouteHoldOpt

# editDelete -type Regular

#==========================================================================================
# 11. Add Filler
#==========================================================================================
# 11.1 Place > Physical Cell > Add Filler
setFillerMode -core {SC8T_FILLX1_CSC20R SC8T_FILLX2_CSC20R SC8T_FILLX3_CSC20R SC8T_FILLX4_CSC20R SC8T_FILLX5_CSC20R SC8T_FILLX8_CSC20R SC8T_FILLX16_CSC20R SC8T_FILLX32_CSC20R SC8T_FILLX64_CSC20R SC8T_FILLX128_CSC20R} -preserveUserOrder true

addFiller

#==========================================================================================
# 12. Verification
#==========================================================================================
# 12.1 Verify > Verify DRC
    # - Optional if not taped out
verify_drc 

# 12.2 Verify > Verify Conectivity 
    # - Regular Only
verifyConnectivity -type regular

# ecoRoute -fix_drc

#==========================================================================================
# 13. PPA
#==========================================================================================
report_timing > timing.rpt
report_area > area.rpt
source spa.tcl
```
2. **mmmc.view**:
```
#---------------------------------------------------------------------
# constraint modes
#---------------------------------------------------------------------
create_constraint_mode -name pre_cts_constr -sdc_files {inputs/stdp_m.sdc}
# create_constraint_mode -name post_cts_constr -sdc_files {inputs/stdp_m_post_cts.sdc}

#---------------------------------------------------------------------
# delay corner = timing library plus rc corner
#---------------------------------------------------------------------
# worst-case corner = max delay, affects setup
# best-case corner  = min delay, affects hold

# typical timing library
create_library_set -name tc_lib -timing {/ihp/projects/_COMMON/GF22FDX/22FDX-EXT_IP/GF22FDX_SC8T_104CPP_BASE_CSC20R_FDK_RELV05R50/model/timing/lib/GF22FDX_SC8T_104CPP_BASE_CSC20R_TT_0P50V_0P00V_0P00V_0P00V_25C.lib.gz}

# create RC corner from QRC tech file
create_rc_corner -name rc_corner -qx_tech_file {/ihp/projects/_COMMON/GF22FDX/22FDX-EXT/V1.0_4.1/PEX/QRC/10M_2Mx_5Cx_1Jx_2Qx_LBthick/nominal/qrcTechFile} -T {25}

create_delay_corner -name tc_dc -library_set {tc_lib} -rc_corner {rc_corner}

#---------------------------------------------------------------------
# analysis view = constraint mode x delay_corner
#---------------------------------------------------------------------
create_analysis_view -name typ -constraint_mode {pre_cts_constr} -delay_corner {tc_dc} 

#---------------------------------------------------------------------
# set analysis view to above for both setup and hold
#---------------------------------------------------------------------
set_analysis_view -setup {typ} -hold {typ}
```
3. **design.globals**:
```
# Netlist
set design_netlisttype verilog
set init_verilog {inputs/stdp_m.v}
set init_design_set_top 1
set init_top_cell stdp

# Technology/Physical Libraries
set init_lef_file { /ihp/projects/_COMMON/GF22FDX/22FDX-EXT/V1.0_4.1/PlaceRoute/Innovus/Techfiles/10M_2Mx_5Cx_1Jx_2Qx_LB/22FDSOI_10M_2Mx_5Cx_1Jx_2Qx_LB_104cpp_tech.lef \
                    /ihp/projects/_COMMON/GF22FDX/22FDX-EXT_IP/GF22FDX_SC8T_104CPP_BASE_CSC20R_FDK_RELV05R50/lef/GF22FDX_SC8T_104CPP_BASE_CSC20R.lef}

# Floorplan

# Power
set init_pwr_net {VDD}
set init_gnd_net {VSS}

# Analysis Configuration
set init_mmmc_file {inputs/mmmc.view}
```
### Voltus-Based Dynamic Power Analysis
1. **Normal Power Analysis**
```
set_power_analysis_mode -reset

read_activity_file -reset

set_power_output_dir .

report_power -outfile power.rpt
```
2. **Static Power Analysis**
```
set_power_analysis_mode -reset
set_power_analysis_mode -analysis_view typ -method static -power_grid_library {
    /ihp/projects/_COMMON/GF22FDX/22FDX-EXT_IP/GF22FDX_SC8T_104CPP_BASE_CSC20R_FDK_RELV05R50/model/power/voltus/TT_0P50V_0P00V_0P00V_0P00V_25C/techonly.cl \
    /ihp/projects/_COMMON/GF22FDX/22FDX-EXT_IP/GF22FDX_SC8T_104CPP_BASE_CSC20R_FDK_RELV05R50/model/power/voltus/TT_0P50V_0P00V_0P00V_0P00V_25C/stdcells.cl
}

read_activity_file -reset
read_activity_file -format VCD -scope stdp_tb/stdp_u0 -start 0ps -end 1420340ps -block {} ../xm/stdp.vcd

set_power_output_dir .

report_power -outfile spa.rpt
# Generate additional report formats
# report_power -instances {u0} -outfile u0.rpt
# report_power -instances {*} -outfile all.rpt
# report_power -instances {ring} -outfile ring.rpt
```
3. **Dynamic Power Analysis**
```
set_power_analysis_mode -reset
set_power_analysis_mode -analysis_view typ -method dynamic_vectorbased -disable_static false -write_static_currents true -power_grid_library {
    /ihp/projects/_COMMON/GF22FDX/22FDX-EXT_IP/GF22FDX_SC8T_104CPP_BASE_CSC20R_FDK_RELV05R50/model/power/voltus/TT_0P50V_0P00V_0P00V_0P00V_25C/techonly.cl \
    /ihp/projects/_COMMON/GF22FDX/22FDX-EXT_IP/GF22FDX_SC8T_104CPP_BASE_CSC20R_FDK_RELV05R50/model/power/voltus/TT_0P50V_0P00V_0P00V_0P00V_25C/stdcells.cl
}

read_activity_file -reset
read_activity_file -format VCD -scope stdp_tb/stdp_u0 -start 0ps -end 1420340ps -block {} ../xm/stdp.vcd

set_power_output_dir ./dynamicPowerResults

set_dynamic_power_simulation -reset

#sim steps around 1000
set_dynamic_power_simulation -period 10ns -resolution 10ps
report_power -outfile spa.rpt
```
## Linux
### Shell
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
36. **cat /proc/cpuinfo**: physical id(no. of physical CPUs), cpu cores(no. of cores per physical CPU), processor(no. of logical CPUs)
37. **Manual Software Installation and Uninstallation**:
    - **Install**
        1. tar -xvzf xxx.tar.gz
        2. cd xxx
        3. ./configure --prefix= tools/xxx
        4. make; make install
    - **Uninstall**
        1. if installed with non-root account (installed into ~): just remove relevant dirs/files;
        2. if installed with root account:
           1. back into the dir where you ran ```./configure``` and ```make``` before, and run ```make uninstall```;
           2. if a doesn't work (Makefile not correctly written), try ```checkinstall``` which allows you to build from source code, but have the packages tracked by apt.
### Vim
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
25. **set autoread**: reload file automatically
26. **:/xx** or **\*xx**: search xx
27. **\\<int\\>**: full word match
28. **:%s/foo/bar/g(c)**: global search and replace
29. **50p**: plaste 50 times
30. **yy+p**: copy and plaste; **dd+p**: cut and plaste
31. **:u** or **u**: undo; **CTRL+r**: redo
32. **:vsp xx.v**: col split; **CTRL+w+w**: switch window
33. **ngg** or **:n**: go to line n; **gg**: go to head; **G**: go to end
34. **:2,3>** or **shift+>+>** or **v+>+>**: indentation; **:2,3<** or **shift+<+<** or **v+<+<**: anti-indentation;
35. **rx**: change single letter to x
36. **vimdiff**: compare two files file1 and file2 

### Vim Plugin
- **Source Code**: https://github.com/HonkW93/automatic-verilog
- **Handbook**: https://automatic-verilog.honk.wang/#/handbook
- **Configuration in .vimrc**:
   ```
   let g:atv_snippet_project = ''
   let g:atv_snippet_company = 'HP GmbH & University of Potsdam'
   let g:atv_snippet_device = ''
   let g:atv_snippet_author = 'Dedong Zhao'
   let g:atv_snippet_email = 'dedong.zhao@ihp-microelectronics.com'
   let g:atv_snippet_website = ''

   map \ai :call g:AutoInst(0)<ESC> 
   map \ap :call g:AutoPara(0)<ESC>
   map \ad :call g:AutoDef()<ESC>  
   map \aa :call g:AutoArg()<ESC>

   ab ai /*autoinst*/
   ab ap /*autoinstparam*/
   ab ad /*autodef*/
   ab aa /*autoarg*/
   ab als always @(posedge clk or negedge rst_n) begin<CR>     if(!rst_n) begin<CR>        end<CR>     else begin<CR>      end<CR> end
   ab alc always @(*) begin<CR>        end
   ab dir //Local Variables:<CR>verilog-library-directories:("." "./aaa/bbb/")<CR>verilog-library-directories-recursive:1<CR>End: 
   ```
- **Hot Keys**:
   - **header comment**: `\hd`
   - **comment**: `\//`(normal or visual mode)
   - **always**: `\ac`, `\as`
   - **auto instantiation**: `\ai`
   - **auto parameter**: `\ap`
   - **auto define**: `\ad`
   - **auto argument**: `\aa`

## C Programming
### Makefile
1. make, make run, make clean
2. general template
```
# Define compiler and compile options
# -g -O0 necessary for debugging
CC = gcc
CFLAGS = -Wall -Wextra  -std=c99 -g -O0

# Define include directory and source directory
INCLUDES = -I./include
SRC_DIR = ./src
OBJ_DIR = ./obj

# Define source files and object files
SRCS = $(wildcard $(SRC_DIR)/*.c)
OBJS = $(patsubst $(SRC_DIR)/%.c, $(OBJ_DIR)/%.o, $(SRCS))

# Define the name of the executable file
TARGET = snn_train 

# Default target
all: $(TARGET)

# Compile the executable file
$(TARGET): $(OBJS)
	$(CC) $(CFLAGS) $(INCLUDES) -o $@ $^

# Compile object files
$(OBJ_DIR)/%.o: $(SRC_DIR)/%.c | $(OBJ_DIR)
	$(CC) $(CFLAGS) $(INCLUDES) -c $< -o $@

# Create the object files directory
$(OBJ_DIR):
	mkdir -p $(OBJ_DIR)

# Clean up the compiled files
clean:
	rm -rf $(OBJ_DIR) $(TARGET)

# Run the executable file
run: $(TARGET)
	./$(TARGET)

.PHONY: all clean run

```
### GDB
1. **gdb ./snn_train**
2. **(gdb) start**: stop at the first line of main()
3. **(gdb) info threads**
4. **(gdb) thread thead_id**
5. **(gdb) continue**: continue from the breakpoint
6. **(gdb) run**: run through the program
7. **(gdb) break utils.c:148**
8. **(gdb) print a[i]**
9. **(gdb) step**: inside function
10. **(gdb) next**: not inside function
11. **(gdb) info breakpoints**
12. **(gdb) delete**: delete all breakpoints
### Valgrind
1. valgrind --tool=memcheck --leak-check=full --track-origins=yes ./snn_train

## Git and SVN
### Git
1. **git clone**
   - HTTPS:
      ```
      # clone a repo
      git clone https://github.com/dedong-zhao/SNNs
      # clone a repo at cur dir (without creating a dir)
      git clone https://github.com/dedong-zhao/SNNs . 
      ```
   - SSH:
      ```
      # Generate an SSH key (if you don't have one already)
      ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
      # Add your SSH key to the SSH agent
      eval "$(ssh-agent -s)"
      ssh-add ~/.ssh/id_rsa
      # Copy the SSH key to your clipboard and add it to your Git account settings
      cat ~/.ssh/id_rsa.pub
      # Clone the repository using the SSH URL
      git clone git@git.ihp-microelectronics.com:d-sya/public/snn.git
      ```

4. **check in**:
   - git pull
   - git status
   - git add file0
   - git add .: stage all files
   - git reset file1 file2: cancel staging of file1 and file2
   - git commit -m "comment"
   - git push origin main
5. **check out**: git checkout filename
6. **git log**: show revision history of a file
7. **git diff**: show the file changes not committed yet
8. **git tag**: show all tags
9. **move tag**:
   - git tag -d  \<tag\>
   - git push origin --delete \<tag\>
   - git tag \<tag\>
   - git push origin \<tag\>
10. **gitk**: show GUI of Git

### SVN
1. **svn clone**: clone a repo
2.  **check in**:
    - svn status
    - svn add/update filename
    - svn commit -m "comment"
3. **svn diff**: show the file changes not committed yet

## Msc
### Writing C Extensions Using Python's C API
1. Write the C function, e.g., stdp_trace.c

2. Create a setup.py file
```
# setup.py
from setuptools import setup, Extension

module = Extension('stdp',
                   sources=['stdp_trace.c'])

setup(name='stdp',
      version='1.0',
      description='This is a STDP module',
      ext_modules=[module])
```

3. Build the module
```
python setup.py build_ext --inplace
```
The model built: stdp.cpython-310-x86_64-linux-gnu.so

4. Call the C function in Python code
```
import stdp
...
stdp.stdp_trace(...)
```
### File Header Comment
#### VS Studio
1. Manage --> User Snippets --> Verilog.jason
2. Add the following code:
```
{
    "HEADER":{
		"prefix": "header",
		"body": [
			"//Institution :    IHP GmbH & University of Potsdam",
			"//Author      :    Dedong Zhao",
			"//Contact     :    dedong.zhao@ihp-microelectronics.com",
			"//Design      :    $TM_FILENAME",
			"//Time        :    $CURRENT_HOUR:$CURRENT_MINUTE:$CURRENT_SECOND $CURRENT_DATE/$CURRENT_MONTH/$CURRENT_YEAR",
			"//Description :    ",
			"//Comments    :    ",
		]
	}
}
```
3. Type "header" and Enter
#### PyCharm
1. File --> Settings --> Editor --> File and Code Templates --> Python Script
2. Add the following Code:
```
# Institution : IHP GmbH & University of Potsdam
# Author      : Dedong Zhao
# Contact     : dedong.zhao@ihp-microelectronics.com
# Time        : ${TIME} ${DATE} 
# File        : ${NAME}.py
# Project     : ${PROJECT_NAME}
# Description :
# Comments    :
```
3. New file with automatically generated header comment

