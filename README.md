# Introduction to Openlane
OpenLane is an automated RTL to GDSII flow based on several components including OpenROAD, Yosys, Magic, Netgen, CVC, SPEF-Extractor, KLayout and a number of custom scripts for design exploration and optimization. It also provides a number of custom scripts for design exploration, optimization and ECO.  
The flow performs all ASIC implementation steps from RTL all the way down to GDSII. Currently, it supports both A and B variants of the sky130 PDK, but support for the newly released GF180MCU PDK is in the works, and instructions to add support for other (including proprietary) PDKs are documented.

<img src="https://user-images.githubusercontent.com/11850122/209485649-7a5b0a11-118f-4fed-a699-f44770463c05.png" width=80%>

## Openlane Lab Purpose
* Setup a Docker image of Openlane flow
* Execute Openlane flow in non-interactive/interactive mode
* Change component configurations before executing Openlane flow
* Create a customized Openlane Docker image

## Openlane Lab Prerequisites
* Ubuntu 20.04
* Docker packages
* Openlane sources with [tag 2022.10.20](https://github.com/The-OpenROAD-Project/OpenLane/tree/2022.10.20)

## 1. Setup a Docker image of Openlane flow

Install Docker:

    $ sudo apt-get update
    $ sudo apt install -y build-essential python3 python3-venv python3-pip make git
    $ sudo apt-get remove docker docker-engine docker.io containerd runc
    $ sudo apt-get update
    $ sudo apt-get install -y ca-certificates curl gnupg lsb-release
    $ sudo mkdir -p /etc/apt/keyrings
    $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    $ echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
         $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    $ sudo apt-get update
    $ sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
    $ sudo docker -v
    Docker version 20.10.22, build 3a2c30b
 
Set USER and add it to Docker group (docker command without sudo):
    
    $ USER=yourusername
    $ sudo usermod -aG docker $USER
    $ sudo reboot
    
Get Openlane lab sources form offical github or attached [OpenLane.zip](https://github.com/bol-edu/openlane-lab/blob/main/OpenLane.zip):

    $ git clone -b 2022.10.20 --depth 1 https://github.com/The-OpenROAD-Project/OpenLane    
    
Build Openlane Docker image:

    $ cd /home/$USER/OpenLane/
    $ make

Check built Openlane Docker image:

    $ docker images
    REPOSITORY          TAG                                        IMAGE ID       CREATED        SIZE
    efabless/openlane   daae2154590cf20e0c20b77e3fc02b6526ad09af   80bd52b6e039   2 months ago   974MB

## 2. Execute Openlane flow in non-interactive/interactive mode

We use the RTL design spm located in directory /OpenLane/designs/spm/ as Openlane flow import and output result log files of each Openlane flow step.

Test to start/finish an Openlane Docker container:

    $ docker run -it efabless/openlane:daae2154590cf20e0c20b77e3fc02b6526ad09af
    OpenLane Container (daae215):/openlane$ ls
    AUTHORS.md  configuration  dependencies  designs  env.py  flow.tcl  LICENSE  regression_results  run_designs.py  scripts
    OpenLane Container (daae215):/openlane$ exit
    exit

Openlane flow in non-interactive mode:

    $ cd /home/$USER/OpenLane && \
         docker run --rm -v /home/$USER/OpenLane:/openlane -v /home/$USER/OpenLane/designs:/openlane/install -v /home/$USER/OpenLane/pdks:/home/$USER/OpenLane/pdks \
         -e PDK_ROOT=/home/$USER/OpenLane/pdks -e PDK=sky130A  --user 1000:1000 -e DISPLAY=localhost:10.0 -v /tmp/.X11-unix:/tmp/.X11-unix -v \
         /home/$USER/.Xauthority:/.Xauthority --network host -ti efabless/openlane:daae2154590cf20e0c20b77e3fc02b6526ad09af    
    OpenLane Container (daae215):/openlane$ sh -c "./flow.tcl -design spm -tag openlane_test -overwrite"  && \
         [ -f ./designs/spm/runs/openlane_test/results/signoff/spm.gds ] && \
         echo "Basic test passed" || \
         echo "Basic test failed"
    OpenLane daae2154590cf20e0c20b77e3fc02b6526ad09af
    All rights reserved. (c) 2020-2022 Efabless Corporation and contributors.
    Available under the Apache License, version 2.0. See the LICENSE file for more details.

    [INFO]: Using configuration in 'designs/spm/config.json'...
    [INFO]: PDK Root: /home/hls/OpenLane/pdks
    [INFO]: Process Design Kit: sky130A
    [INFO]: Standard Cell Library: sky130_fd_sc_hd
    [INFO]: Optimization Standard Cell Library: sky130_fd_sc_hd
    [INFO]: Run Directory: /openlane/designs/spm/runs/openlane_test
    [INFO]: Preparing LEF files for the nom corner...
    [INFO]: Preparing LEF files for the min corner...
    [INFO]: Preparing LEF files for the max corner...
    [STEP 1]
    [INFO]: Running Synthesis (log: designs/spm/runs/openlane_test/logs/synthesis/1-synthesis.log)...
    [STEP 2]
    [INFO]: Running Single-Corner Static Timing Analysis (log: designs/spm/runs/openlane_test/logs/synthesis/2-sta.log)...
    [STEP 3]
    [INFO]: Running Initial Floorplanning (log: designs/spm/runs/openlane_test/logs/floorplan/3-initial_fp.log)...
    [WARNING]: Current core area is too small for the power grid settings chosen. The power grid will be scaled down.
    [INFO]: Floorplanned with width 90.62 and height 89.76.
    [STEP 4]
    [INFO]: Running IO Placement (log: designs/spm/runs/openlane_test/logs/floorplan/4-place_io.log)...
    [STEP 5]
    [INFO]: Running Tap/Decap Insertion (log: designs/spm/runs/openlane_test/logs/floorplan/5-tap.log)...
    [INFO]: Power planning with power {VPWR} and ground {VGND}...
    [STEP 6]
    [INFO]: Generating PDN (log: designs/spm/runs/openlane_test/logs/floorplan/6-pdn.log)...
    [STEP 7]
    [INFO]: Running Global Placement (log: designs/spm/runs/openlane_test/logs/placement/7-global.log)...
    [STEP 8]
    [INFO]: Running Placement Resizer Design Optimizations (log: designs/spm/runs/openlane_test/logs/placement/8-resizer.log)...
    [STEP 9]
    [INFO]: Running Detailed Placement (log: designs/spm/runs/openlane_test/logs/placement/9-detailed.log)...
    [STEP 10]
    [INFO]: Running Clock Tree Synthesis (log: designs/spm/runs/openlane_test/logs/cts/10-cts.log)...
    [STEP 11]
    [INFO]: Running Placement Resizer Timing Optimizations (log: designs/spm/runs/openlane_test/logs/cts/11-resizer.log)...
    [STEP 12]
    [INFO]: Running Global Routing Resizer Timing Optimizations (log: designs/spm/runs/openlane_test/logs/routing/12-resizer.log)...
    [STEP 13]
    [INFO]: Running Detailed Placement (log: designs/spm/runs/openlane_test/logs/routing/13-diode_legalization.log)...
    [STEP 14]
    [INFO]: Running Global Routing (log: designs/spm/runs/openlane_test/logs/routing/14-global.log)...
    [INFO]: Starting OpenROAD Antenna Repair Iterations...
    [STEP 15]
    [INFO]: Writing Verilog (log: designs/spm/runs/openlane_test/logs/routing/14-global_write_netlist.log)...
    [STEP 16]
    [INFO]: Running Fill Insertion (log: designs/spm/runs/openlane_test/logs/routing/16-fill.log)...
    [STEP 17]
    [INFO]: Running Detailed Routing (log: designs/spm/runs/openlane_test/logs/routing/17-detailed.log)...
    [INFO]: No DRC violations after detailed routing.
    [STEP 18]
    [INFO]: Running SPEF Extraction at the min process corner (log: designs/spm/runs/openlane_test/logs/signoff/18-parasitics_extraction.min.log)...
    [STEP 19]
    [INFO]: Running Multi-Corner Static Timing Analysis at the min process corner (log: designs/spm/runs/openlane_test/logs/signoff/19-rcx_mcsta.min.log)...
    [STEP 20]
    [INFO]: Running SPEF Extraction at the max process corner (log: designs/spm/runs/openlane_test/logs/signoff/20-parasitics_extraction.max.log)...
    [STEP 21]
    [INFO]: Running Multi-Corner Static Timing Analysis at the max process corner (log: designs/spm/runs/openlane_test/logs/signoff/21-rcx_mcsta.max.log)...
    [STEP 22]
    [INFO]: Running SPEF Extraction at the nom process corner (log: designs/spm/runs/openlane_test/logs/signoff/22-parasitics_extraction.nom.log)...
    [STEP 23]
    [INFO]: Running Multi-Corner Static Timing Analysis at the nom process corner (log: designs/spm/runs/openlane_test/logs/signoff/23-rcx_mcsta.nom.log)...
    [STEP 24]
    [INFO]: Running Single-Corner Static Timing Analysis at the nom process corner (log: designs/spm/runs/openlane_test/logs/signoff/24-rcx_sta.log)...
    [STEP 25]
    [INFO]: Creating IR Drop Report (log: designs/spm/runs/openlane_test/logs/signoff/25-irdrop.log)...
    [STEP 26]
    [INFO]: Running Magic to generate various views...
    [INFO]: Streaming out GDSII with Magic (log: designs/spm/runs/openlane_test/logs/signoff/26-gdsii.log)...
    [INFO]: Generating MAGLEF views...
    [STEP 27]
    [INFO]: Streaming out GDSII with KLayout (log: designs/spm/runs/openlane_test/logs/signoff/27-gdsii-klayout.log)...
    [STEP 28]
    [INFO]: Running XOR on the layouts using KLayout (log: designs/spm/runs/openlane_test/logs/signoff/28-xor.log)...
    [STEP 29]
    [INFO]: Running Magic Spice Export from LEF (log: designs/spm/runs/openlane_test/logs/signoff/29-spice.log)...
    [STEP 30]
    [INFO]: Writing Powered Verilog (logs: designs/spm/runs/openlane_test/logs/signoff/30-write_powered_def.log, designs/spm/runs/openlane_test/logs/signoff/30-write_powered_verilog.log)...
    [STEP 31]
    [INFO]: Writing Verilog (log: designs/spm/runs/openlane_test/logs/signoff/30-write_powered_verilog.log)...
    [STEP 32]
    [INFO]: Running LVS (log: designs/spm/runs/openlane_test/logs/signoff/32-lvs.lef.log)...
    [STEP 33]
    [INFO]: Running Magic DRC (log: designs/spm/runs/openlane_test/logs/signoff/33-drc.log)...
    [INFO]: Converting Magic DRC database to various tool-readable formats...
    [INFO]: No DRC violations after GDS streaming out.
    [STEP 34]
    [INFO]: Running OpenROAD Antenna Rule Checker (log: designs/spm/runs/openlane_test/logs/signoff/34-antenna.log)...
    [STEP 35]
    [INFO]: Running CVC (log: designs/spm/runs/openlane_test/logs/signoff/35-erc_screen.log)...
    [INFO]: Saving current set of views in 'designs/spm/runs/openlane_test/results/final'...
    [INFO]: Saving runtime environment...
    [INFO]: Generating final set of reports...
    [INFO]: Created manufacturability report at 'designs/spm/runs/openlane_test/reports/manufacturability.rpt'.
    [INFO]: Created metrics report at 'designs/spm/runs/openlane_test/reports/metrics.csv'.
    [INFO]: There are no max slew, max fanout or max capacitance violations in the design at the typical corner.
    [INFO]: There are no hold violations in the design at the typical corner.
    [INFO]: There are no setup violations in the design at the typical corner.
    [SUCCESS]: Flow complete.
    [INFO]: Note that the following warnings have been generated:
    [WARNING]: Current core area is too small for the power grid settings chosen. The power grid will be scaled down.    

    Basic test passed
    OpenLane Container (daae215):/openlane$ klayout -e -nn $PDK_ROOT/sky130A/libs.tech/klayout/tech/sky130A.lyt \
         -l $PDK_ROOT/sky130A/libs.tech/klayout/tech/sky130A.lyp \
          ./designs/spm/runs/openlane_test/results/final/gds/spm.gds
    # View layout in KLayout GUI
    # Close KLayout GUI
    OpenLane Container (daae215):/openlane$ exit
    exit
                
<img src="https://user-images.githubusercontent.com/11850122/209523370-b20e3c58-d4fa-407b-b335-d71e3065e80a.png" width=60%>

Openlane flow in interactive mode:

    $ cd /home/$USER/OpenLane && \
         docker run --rm -v /home/$USER/OpenLane:/openlane -v /home/$USER/OpenLane/designs:/openlane/install -v /home/$USER/OpenLane/pdks:/home/$USER/OpenLane/pdks \
         -e PDK_ROOT=/home/$USER/OpenLane/pdks -e PDK=sky130A  --user 1000:1000 -e DISPLAY=localhost:10.0 -v /tmp/.X11-unix:/tmp/.X11-unix -v \
         /home/$USER/.Xauthority:/.Xauthority --network host -ti efabless/openlane:daae2154590cf20e0c20b77e3fc02b6526ad09af    
    OpenLane Container (daae215):/openlane$ sh -c "./flow.tcl -interactive"
    OpenLane daae2154590cf20e0c20b77e3fc02b6526ad09af
    All rights reserved. (c) 2020-2022 Efabless Corporation and contributors.
    Available under the Apache License, version 2.0. See the LICENSE file for more details.

    % package require openlane
    0.9
    % prep -design spm -tag openlane_test -overwrite
    [INFO]: Using configuration in 'designs/spm/config.json'...
    [INFO]: PDK Root: /home/hls/OpenLane/pdks
    [INFO]: Process Design Kit: sky130A
    [INFO]: Standard Cell Library: sky130_fd_sc_hd
    [INFO]: Optimization Standard Cell Library: sky130_fd_sc_hd
    [INFO]: Run Directory: /openlane/designs/spm/runs/openlane_test
    [INFO]: Removing existing /openlane/designs/spm/runs/openlane_test...
    [INFO]: Preparing LEF files for the nom corner...
    [INFO]: Preparing LEF files for the min corner...
    [INFO]: Preparing LEF files for the max corner...
    % run_synthesis
    [STEP 1]
    [INFO]: Running Synthesis (log: designs/spm/runs/openlane_test/logs/synthesis/1-synthesis.log)...
    [STEP 2]
    [INFO]: Running Single-Corner Static Timing Analysis (log: designs/spm/runs/openlane_test/logs/synthesis/2-sta.log)...
    % run_floorplan
    [STEP 3]
    [INFO]: Running Initial Floorplanning (log: designs/spm/runs/openlane_test/logs/floorplan/3-initial_fp.log)...
    [WARNING]: Current core area is too small for the power grid settings chosen. The power grid will be scaled down.
    [INFO]: Floorplanned with width 90.62 and height 89.76.
    [STEP 4]
    [INFO]: Running IO Placement (log: designs/spm/runs/openlane_test/logs/floorplan/4-place_io.log)...
    [STEP 5]
    [INFO]: Running Tap/Decap Insertion (log: designs/spm/runs/openlane_test/logs/floorplan/5-tap.log)...
    [INFO]: Power planning with power {VPWR} and ground {VGND}...
    [STEP 6]
    [INFO]: Generating PDN (log: designs/spm/runs/openlane_test/logs/floorplan/6-pdn.log)...
    % run_placement
    [STEP 7]
    [INFO]: Running Global Placement (log: designs/spm/runs/openlane_test/logs/placement/7-global.log)...
    [STEP 8]
    [INFO]: Running Placement Resizer Design Optimizations (log: designs/spm/runs/openlane_test/logs/placement/8-resizer.log)...
    [STEP 9]
    [INFO]: Running Detailed Placement (log: designs/spm/runs/openlane_test/logs/placement/9-detailed.log)...
    % exit
    OpenLane Container (daae215):/openlane$ exit
    exit

More interactive Openlane [Tcl commands](https://openlane.readthedocs.io/en/latest/reference/openlane_commands.html).

## 3. Change component configurations before executing Openlane flow

Change and save configurations in an example synthesis.tcl:

    $ cd /home/$USER/OpenLane && \
         docker run --rm -v /home/$USER/OpenLane:/openlane -v /home/$USER/OpenLane/designs:/openlane/install -v /home/$USER/OpenLane/pdks:/home/$USER/OpenLane/pdks \
         -e PDK_ROOT=/home/$USER/OpenLane/pdks -e PDK=sky130A  --user 1000:1000 -e DISPLAY=localhost:10.0 -v /tmp/.X11-unix:/tmp/.X11-unix -v \
         /home/$USER/.Xauthority:/.Xauthority --network host -ti efabless/openlane:daae2154590cf20e0c20b77e3fc02b6526ad09af
    OpenLane Container (daae215):/openlane$ ls configuration/
    checkers.tcl  cts.tcl  extraction.tcl  floorplan.tcl  general.tcl  load_order.txt  placement.tcl  routing.tcl  synthesis.tcl  
    OpenLane Container (daae215):/openlane$ vi configuration/synthesis.tcl
    
    # Copyright 2020 Efabless Corporation
    #
    # Licensed under the Apache License, Version 2.0 (the "License");
    # you may not use this file except in compliance with the License.
    # You may obtain a copy of the License at
    #
    #      http://www.apache.org/licenses/LICENSE-2.0
    #
    # Unless required by applicable law or agreed to in writing, software
    # distributed under the License is distributed on an "AS IS" BASIS,
    # WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    # See the License for the specific language governing permissions and
    # limitations under the License.

    # Synth defaults
    set ::env(SYNTH_BIN) yosys
    set ::env(SYNTH_SCRIPT) $::env(SCRIPTS_DIR)/yosys/synth.tcl
    set ::env(SYNTH_NO_FLAT) 0
    set ::env(SYNTH_CLOCK_UNCERTAINTY) 0.25
    set ::env(SYNTH_CLOCK_TRANSITION) 0.15
    set ::env(SYNTH_TIMING_DERATE) 0.05
    set ::env(SYNTH_SHARE_RESOURCES) 1
    set ::env(SYNTH_BUFFERING) 1
    set ::env(SYNTH_SIZING) 0
    set ::env(SYNTH_MAX_FANOUT) 10
    set ::env(SYNTH_STRATEGY) "AREA 0"
    set ::env(SYNTH_ADDER_TYPE) "YOSYS"
    set ::env(CLOCK_BUFFER_FANOUT) 16
    set ::env(SYNTH_READ_BLACKBOX_LIB) 0
    set ::env(SYNTH_ELABORATE_ONLY) 0
    set ::env(SYNTH_FLAT_TOP) 0
    set ::env(IO_PCT) 0.2
    set ::env(SYNTH_EXTRA_MAPPING_FILE) ""

    set ::env(BASE_SDC_FILE) $::env(SCRIPTS_DIR)/base.sdc
    
    :wq!
    
    OpenLane Container (daae215):/openlane$    

The Openlane flow defined [configuration variables](https://openlane.readthedocs.io/en/latest/reference/configuration.html).

## 4. Create a customized Openlane Docker image

Run Openlane Docker container in background:

    $ docker run -d -it efabless/openlane:daae2154590cf20e0c20b77e3fc02b6526ad09af
    c0b53c50e2a3abaf5f3360f73478e264c83abd18834513d261e64ac2b70d50c1

Query running Docker container and get its CONTAINER ID 'c0b53c50e2a3':

    $ docker ps
    CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS     NAMES
    c0b53c50e2a3   80bd52b6e039   "/bin/sh -c /bin/bash"   7 seconds ago   Up 6 seconds             reverent_chebyshev

Add a test.txt into Docker container:

    $ echo "Hello OpenLane" > ./test.txt && docker cp ./test.txt c0b53c50e2a3:/openlane/test.txt

Save changed Docker container to new Openlane Docker image:

    $ docker commit c0b53c50e2a3 boledulab/openlane-lab:1.0
    
List Docker images:

    $ docker images
    REPOSITORY               TAG                                        IMAGE ID       CREATED          SIZE
    boledulab/openlane-lab   1.0                                        a4d7bca163b7   14 seconds ago   974MB
    efabless/openlane        daae2154590cf20e0c20b77e3fc02b6526ad09af   80bd52b6e039   2 months ago     974MB

Test the new Openlane Docker image 'boledulab/openlane-lab:1.0':

    $ docker run -it boledulab/openlane-lab:1.0 cat ./test.txt
    Hello OpenLane
    $
    
## Reference
* OpenLANE: The Open-Source Digital ASIC Implementation Flow: https://woset-workshop.github.io/PDFs/2020/a21.pdf
* The OpenROAD Project - OpenLane: https://github.com/The-OpenROAD-Project/OpenLane
* The OpenLane Documentation: https://openlane.readthedocs.io/en/latest/index.html
