# openlane_workshop
This project is done in the course "Advanced Physical Design using OpenLANE/Sky130" by VLSI System Design Corporation. In this project a complete RTL to GDSII flow for PicoRV32a SoC is executed with Openlane using Skywater130nm PDK. Custom desgined standard cells with Sky130 PDK are also used in the flow. Timing Optimisations are carried out. Slack violations are removed. DRC is verified.

# TABLE OF CONTENTS

[* Introduction](# Introduction)
Overall Design Flow
OpenLane Flow
1. Synthesis
1.1 Synthesis Strategies
1.2 Deign Exploration Utility
1.3 Design For Test - DFT Insertion
2. Floor Planning and Power Planning
3. Placement
4. Clock Tree Synthesis
5. Fake Antenna and diode swapping
5. Routing
6. RC Extraction
7. STA
8. Sign-off Steps
9. GDSII Extraction
OpenLane Installation and Environment Setup
OpenLane Directory Structure
Working with OpenLane
Start Openlane
Design Preparation
Configuration Priority
Synthesis
Key concepts
Utilisation Factor
Aspect Ratio
Floorplanning
Pre-Placed cells
Decoupling Capacitors to the pre placed cells
Power Planning
Pin Placement
Floorplanning - Openlane
Placement
Cell Design Flow
SPICE Deck Creation
Simulation in ngspce
VTC
VTC with 2.5 x W (2.5 times channel width of pmos
Transient Simulation
Custom Design of SKY130 Standard cell
SPICE Characterisation
LEF Extraction








Synthesis, Floorplanning with custom standard cell
Static Timing Analysis
Floorplanning and Placement
CTS
Pre-CTS Timing Analysis in OpenRoad
PDN
Routing
GDSII
Acknowledgements
References

























## Introduction
With the advent of open-source technologies for Chip development, there were several RTL designs, EDA Tools which were open-sourced. The missing piece in a complete Open source chip development was filled by the SKY130 PDK from Skywater Technologies and Google. There were several EDA Tools, which played specfic roles in the design cycle. There was not a clean design flow and Skywater pdk was compatible with only the industrty tools. OpenLane addressed these issues in providing a completely automated and clean RTL to GDSII flow. OpenLane is not a tool, but a flow which consists of several EDA tools, automation scripts and Skywater-pdks tuned to work specifically with the open-source EDA tools.

## Overall Design Flow
For a design Specification an RTL Design is written in HDLs like Verilog /VHDL or RTL Design is generated using Hardware Construction Languages like Chisel or High Level Synthesis using SystemC, MATLAB HDL Coder, Bluespec etc or a modern abstraction level called TL-Verilog (its not a HDL/HLS) , specified by TL-x.org. After this begins the workflow of taking the RTL Netlist into a fabricated IC, which is called as Physical Design Flow.

Physical Design begins with Floor planning - placing the preplaced cells, power planning etc., secondly Placement of Logical Synthesis. Now we do CTS (Clock Tree Synthesis) such there the skew of the clock is the minimum or within the required threshold. After CTS, Routing is done to route all the components placed. Between each and every step that happens in the physical design flow starting from Logic Synthesis to routing, a procedure called "Static Timing Analysis" is done to analyse the design at every step to ensure the actual correctness of the design. To view every stage, Magic is an open source tool to view the layouts. A small netlist can be extracted and a SPICE Simulation can be performed and compared with the Post Layout Simulation using ngspice![1](https://github.com/LanwinDsouza/openlane_workshop/assets/104729279/baa90e96-b888-42b0-81c0-defdde567039)

## OpenLane Flow

#### 1. Synthesis
   The RTL Level Design is then synthesized using a Logic Synthesizer. We use Yosys which is an Open Source Logic Synthesizer. The RTL Netlist is then converted into a synthesised netlist where there are details about the standard cells and its implementations. Yosys takes the RTL design and timing .libs and verilog models of standard cells and converts into a RTL Netlist. abc does the tehnology mapping to the required skywater-pdk variants
        
##### 1.1 Synthesis Strategies
Different strategies can be used to synthesize for the either the least area or the best timing. To analyse this, synthesis exploration utility generates a report showing the effect on delays/timing/area et.,

##### 1.2 Deign Exploration Utility
This is used to suit the design configuration and generate reports with different metrics to select the best. This is also used for regression testing

##### 1.3 Design For Test - DFT Insertion
This is an optional step carried out by Fault. It is used to test the design

#### 2. Floor Planning and Power Planning
This is done by OpenROAD flow. The macros and IPs are placed in the core before proceding further. This is called as pre-placement. Floor planning is done separately for the macros and it is called macro floor planning. They are placed in such a way that they are closer to the inputs/outputs/other macros where more connections are present. Then to prevent the loading effects de-coupling capacitors are placed so that the logic states are well within the noise margin.

When several blocks tap power from a single source, there is a problem of Voltage Droop at the Vdd and Ground Bounce at the Vss which can again push the logic out of the required noise margin into the undefined state. To mitigate this Vdd and Vss are placed as horizontal and vertical strips in the chip so that the blocks can tap power from the nearest source.

#### 3. Placement
There are two types of placement. The other required logic is placed optimally. Placement is of two steps

Global Placement- finds the optimal position for each cells. These positions are not necessarly correct, cells may overlap
Detialed Placement - After Global placement is done minimal alterations are done to correct the issues
#### 4. Clock Tree Synthesis
To ensure minimum skew the Clock is routed optimally through the circuit using different algorithms. This is done in the OpenROAD flow. This is done by TritonCTS.

#### 5. Fake Antenna and diode swapping
Long wires acts as antennas and cause accumulation of charges during the fabrication process damaging the transistor. To avoid this bridging is used to pass the wire through different layers or an antenna diode cell is added to leak away the charges

OpenLane approach - Insert Fake Diode to every cell input during placement. This matches the footprint of the library of the antenna diode. The Antenna Checker is run to check for violations, if there are violations then the fake diode is swapped with a real one.
OpenROAD approach - In the global route step, the antenna violation is addressed automatically by inserting an antenan diode OpenLane allows the user to chose either of the above approaches
#### 5. Routing
This step is used to implement the interconnect using the different metal layers specified in the PDK. There are two steps

Global Routing - This is done inside the OpenROAD flow (FastRoute)
Detailed Routing - This is performed using TritonRoute outside the OpenROAD flow after the global routing. Before performing this step the Logic Equivalence Check is performed by Yosys, since OpenROAD does some optimisations the circuit.
#### 6. RC Extraction
From the .def file, the parasitic extraction is done to generate the .spef file (Standard Prasitic Exchange Format) which produces an accurate analog model of the circuit by including the parasitic effects due to wires, parasitic capacitances, etc.,

#### 7. STA
At this stage again OpenSTA is used to perform the Static Timing Analysis.

#### 8. Sign-off Steps
Design Rule Check (DRC) is performed by Magic
Layout Versus Schematic (LVS) is performed by Netgen
#### 9. GDSII Extraction
The routed .def file is used my Magic to generate the GDSII file

## Working with OpenLane
### Start Openlane
Go the the openlane directory and type docker to start the docker containter.\

The terminal changes into the docker instance.\

Open the OpenLane in interactive mode.\

``` ./flow.tcl -interactive\ ```

Set the package required by OpenLane.\

``` package require openlane 0.9 ```

### Design Preparation
##### Prepare the design

``` prep -design picorv32a ```

* To resume from a previous run use ``` -tag run_name ```
* To overwrite the previous run use ``` tag run_name -overwrite ```
* Note: Any configuration done in the ``` config.tcl  ``` of the source folder after design preparation will not be refleceted. To run wih a modified configuration, the design configuration can     be overriten by passing the configuration to openlane interactively
* A runs folder is created as discussed
* On loading a previous run, to know the last run state one has to check the ``` Current def ``` file which is set. This can be done using
           > echo $::env(CURRENT_DEF) 
* To set to resume from a stage before the ``` current DEF ```, one has to set the ``` CURRENT_DEF ``` environment variable to the required path. This can be done using
           > set ::env(CURRENT_DEF) /path/to/the/required/def/file 
* The ``` def ``` files of every stage can be found in the ``` runs>results>stage_name>design_stage.def path.```
* These ``` def ``` files can be opened with ``` magic  ```by using the ``` sky130A.tech'```  as the technology file and the ```lef``` file from the ``` tmp``` directory if required.
   
   ///pic///
   

### Configuration Priority
Configuration priority (from high to low) is as follows

* ```pdk_specific_config.tcl ```- Design Folder
* ```config.tcl``` - Design Folder
* ```tool_specific_config``` - Configuration Folder in OPENLANE_ROOT

## Synthesis
Run the synthesis

```run_synthesis```

OpenLane invokes the following

```Yosys``` - RTL Synthesis and maps to yosys generic cells
```abc``` - Technology mapping with the ```Skywater130 PDK```. Here sky130_fd_sc_hd Skywater Foundry produced High density standard cells are used.
```OpenSTA ```- This does the Static Timing Analysis on the netlist generated after synthesis and generated the timing reports
View the synthesis statistics

///pic///


The STA Reports can be viewed from the Reports folder.

The openSTA tool generated the timing reports. It can be seen from below that

* total negative slack = -759.46
* worst negative slack = -24.89
 
### Key concepts
#### Utilisation Factor
* The flop ratio is defined as the ratio of the number of flops to the total number of cells
* Here flop ratio is 1613/14876 = 0.1084 (i.e: 10.8%) [From the synthesis statistics]
### Utilisation Factor
* The ratio of area occupied by the cells in the netlist to the total area of the core
* Best practice is to set the utilisation factor less than 50% so that there will be space for optimisations, routing, inserting buffers etc.,
### Aspect Ratio
* Aspect ratio is the ratio of height to the width of the die.
* Aspect Ratio of 1 indicates that the die is a square die.


## Floorplanning
Floorplanning involves the following stages

### Pre-Placed cells
* Whenever there is a complex logic which is repeated multiple times or a design given by a third-party it can be perceived as abstract black box with input and output ports, clocks etc .,

* These modules can be either macros or IP

    * Macro - It is a module such as CPU Core which are developed by the entity fabicating the chip
    * IP - It is an "Intellectual Propertly" which the entity fabricating the chip gets as a package from a third party or even packaged Hard IPs developed by the same entity. Common examples of       IPs are SRAM, PLL, Protocol Converters etc.,
* These Macros and IPs are placed in the core at first before placing the standard cells and power planning

* These are optimally such that the cells which are more connected to each other are placed nearby and oriented for input and ouputs

### Decoupling Capacitors to the pre placed cells
* The power lines can have some RLC component causing the voltage to drop at the node where it enters the Blocks or the ground of the cell can be at a higher potential than ideally 0V
* When this happens, there is a chance such that the logic transitions are not to the upper or lower noise margins but to the forbidden state causing the circuit to misbehave
* This is prevented by adding a capacitor in parallel with the power and ground node of the block such that the capacitor decouples the block from the power source whenever there is a logic          transition
### Power Planning
* When there are several cells or blocks drawing power from the same power rail and sinking power to the same ground pin the following effects are observed
   * Whenever there is alogic transition from 1 to 0 in a large number of cells then there is a Voltage Droop in the power lines as Voltage Drops from Vdd
   * Whener there is a logic transition from 0 to 1 in a large number of cells simultaneously causes the ground potential to raise above 0V calles as Ground Bump
   * These effects pose a risk of driving the logic state out of the specified noise margin.
   * To avoid this the Vdd and Gnd are placed as a grid of horizontal and vertical tracks and the cell nearer to an intersection can tap power or sink power to the Vdd or Gnd intersection     
     respectively
### Pin Placement
* The input, output and Clock pins are placed optimally such that there is less complication in routing or optimised delay
* There are different styles of pin placement in openlane like ```random pin placement ```, ```uniformly spaced ```etc.,

### Floorplanning - Openlane
Command: ```run_floorplan```

Let us change the ```VMETAL``` and ```HMETAL Layers```

///pic///
Run the floorplan
///pic///
Configuration reflected in the runs folder
///pic///

This command generated the ```picorv32a.floorplan.def``` file in the ```./results/floorplan directory```

Open the file in ```magic```

```magic -T /path tosky130A.tech ```file in ```libs.tech magic/```

In the ```tkcon``` window read the ```lef ```and ```def``` file as follows. The ```lef``` file is present in the ```tmp ``` directory as ```merged.lef```

///pic//



Zoom in view that the pins are equally spaced
![image](https://github.com/LanwinDsouza/openlane_workshop/assets/104729279/e1314012-b8be-464a-ad48-e59204a57cd5)

* Tap calls are used to avoid Latchup connections
* They connect the nwell to the Vdd and Substrate to Gnd
* In the lower left corner some standard cell buffer are placed even though placement is not done


## Cell Design Flow

* Inputs : PDK, DRC & LVS rules, SPICE models, library & User defined specs
  * The introduction of lambda based design rules allowed a design to be loosely tied with the fabrication process
  * The layout geometry (DRC) are expressed in terms of multiples of lambda which is half the feature size
  * Users define the cell height to be the separation between the power and the ground rail
  * Cell width is dependent on the timing information and required drive strength
  * Cell Width increases, Area Increases, Timing decreases, Drive Strength increases as the Resistance and Capacitance decreases(RC)
  * Supply voltage is also specified by the top level design
  * The designed cell must fit in the above specifications
* Output : CDL(Circuit Description Language), GDSII(Graphic Design Standard 2), LEF(Layout Exchange Format), .lib containing Timing, Noise and Power characteristics
* Process
   1. Circuit Design

     * The function is implemented interms of MOSFETs and a network graph is drawn for PDN and PUN
     * The Euler Path is identified for PUN and PDN
     * The w/l ratio of the mosfets are decided
     * The output we get is interms of a Circuit Description Language
     
  2.Layout Design

     * Based on the Euler Path a stick diagram is drawn and the layout is drawn in magic
     * DRC is verified in magic
     * ```extract``` all command is used to extract the ```.ext ```file
     * ```ext2spice cthresh0 rthresh0 ```and RC model spice extraction is done
     
  3.Charactersisation

     * Modify the   ```.spice``` file with the necessary power sources
     * Add the library files and pmos, nmos models
     * Add Stimulus commands
     * Obtain
      * slew_low_rise_thr (20% of max)
      * slew_high_rise_thr (80%)
      * slew_low_fall_thr (20%)
      * slew_high_fall_thr(80%)
      * in_rise_thr
      * in_fall_thr
      * out_fall_thr
      * out_rise_thr
    * Calculate
     * Slew_x = difference between slew_high_x_thr and slew_low_x_thr
     * Delay_x = difference between out_x_thr and in_x_thr


### Custom Design of SKY130 Standard cell
Refer "Nickson-Jose" Git Repo for the files

The objective is to insert the custom designed inverter into the openlane flow

Open the ``` sky130_inv.mag ```file in magic
![image](https://github.com/LanwinDsouza/openlane_workshop/assets/104729279/e1bb2b8a-e5ce-4af3-9647-431c80ab1f8a)

* DRC is checked. To place in the openLANE flow we need the LEF File only.

* LEF is the Library Exchange Format.

* It has only the information of the metal layers.

* It has no information of the function.

* Because only the metal contacts are sufficient enough to do the placement.

* This allows for protection of the IP of the vendor so the buyer cant reverse engineer the design as a single LEF can enumerate to multiple Layouts as the number of possible interconnection keep increasing with intersections and the layers

#### SPICE Characterisation

Extracted Spice File

Ngspice simulation


From the graph manually timing characterisation is done

### LEF Extraction
To make the standard cells to be used in the PnR the following rules are followed

 *The input and output ports must lie on the vertical and horizontal tracks
 *The width of the standard cell muse be odd multiple of the track pitch
 *The height of the standard cell must be odd multiple of the vertical pitch
 *tracks.info file contains this information


Adjust grid accordingly so that the geometry can be interpreted with the track information
![image](https://github.com/LanwinDsouza/openlane_workshop/assets/104729279/bf2bad04-3c9c-4f3f-92bc-26cd33d1eff8)
![image](https://github.com/LanwinDsouza/openlane_workshop/assets/104729279/15e36b25-81fd-48e7-9a5f-46440c4e44af)

In the Layout file the ports are defined,as the LEF file requires only ihe information of the ports by using magic edit>text

Extract the lef file

![image](https://github.com/LanwinDsouza/openlane_workshop/assets/104729279/1b61cb4d-f24a-4f49-9f30-c3faf3d07625)

Next, Copy the libraries and lef file the the design source folder.

## Synthesis, Floorplanning with custom standard cell
Edit the config.tcl in the design folder as shown below

In openlane enter the following to include the lef file

Run Synthesis

It can be seen that the added cell is included


