# ISPD25 Contest: Performance-Driven Large Scale Global Routing

<img width="1000" alt="profile" src="etc/ispd_logo_2025.png">

### Contest Introduction

Global routing is a critical component of the VLSI design process, exerting a substantial influence on circuit timing, power consumption, and overall routability. The efficiency of global routing is of paramount importance, as a swift and scalable approach can guide optimizations in early design stages like floor-planning and placement.

To encourage academic research in addressing the scalability challenges of global routing algorithms using GPU and machine learning (ML) technologies, we hosted the [ISPD2024 contest on GPU/ML-enhanced large-scale global routing](https://github.com/liangrj2014/ISPD24_contest/blob/main/index.md). This contest introduced a set of large-scale benchmarks, encompassing up to 50 million cells. These benchmarks reflected industrial-level congestion, presenting challenging routing scenarios. The contest featured simplified input/output formats and evaluation metrics, framing the global routing challenges as mathematical optimization problems.

While the simplified input/output formats and evaluation metrics enhance the accessibility of the competition to participants from diverse backgrounds, they can introduce inaccuracies in performance modeling. The input files used in the ISPD2024 contest lack timing and power information. Metrics based solely on wirelength and routing overflow fail to accurately model timing performance and power consumption. For instance, minimizing total wirelength does not necessarily reduce delays on timing-critical paths. Wires on different metal layers exhibit varying resistance, resulting in different delays and power consumption. Additionally, the impact of vias on delays is difficult to model with simple metrics. Inter-wire coupling capacitance can also cause significant discrepancies between actual and nominal timing responses and power consumption.

In light of the above, we will host another GPU/ML-enhanced large-scale global routing to consider more accurate performance modeling, bringing the contest problem one step closer to the real-world routing challenges. Additionally, we encourage participants to release their code under a permissive open-source license. The main additions in this year's contest are as follows:

(1) For each testcase, this contest provides two sets of input files: a) industry-standard LEF, DEF, LIB, and SDC files, and b) simplified rerouting resource and net information files. The industry-standard files serve as the raw input, enabling contestants to perform the most accurate modeling of routing resources and performance. The simplified input files offer abstracted routing resource and net connection information, along with pre-routing timing estimates, allowing contestants to quickly engage with the contest.
    
(2) This contest establishes a global routing performance evaluation flow utilizing [OpenROAD](https://github.com/The-OpenROAD-Project/OpenROAD), a leading open-source chip design toolset. The evaluation flow involves loading a global routing solution into OpenROAD, which then estimates parasitics, timing, power, and routing congestion for the global routing solutions.

(3) This contest encourages participants to release their code under a permissive open-source license, promoting contributions to the open-source community.

### Problem Formulation

In global routing, a 3D routing space is defined using global routing cells (GCells), created by a regular grid of horizontal and vertical lines. This configuration results in the formation of a grid graph where each GCell is treated as a vertex and edges connect adjacent GCells within the same layer (GCell edges) or between GCells in neighboring layers (via edges). The global router needs to establish a concrete path for each net within the grid graph and optimize the routability, timing and power.

For each testcase, the global router starts with a placed design,
and generates a global routing solution. The global routing solution is evaluated by OpenROAD, which reports timing, power, and routing congestion. Additionally, the runtime and memory efficiency of the global router are critical factors.

### Input/Output Formats and Evaluation

For each testcase, two sets of input files are provided: industry-standard files and simplified files.

The industry-standard files include DEF, LEF, LIB, and SDC files. The DEF file contains definitions for CORE, ROW, TRACKS, and GCELLGRID, along with placed COMPONENTS and unrouted NETS. Similar to the [ICCAD2019 global routing contest](https://www.iccad-contest.org/2019/problems.html), GCells are specified using the definition from the DEF GCELLGRID section. The LEF file includes MACRO definitions and technology information. The LIB files offer timing and power data for library cells, while the SDC files provide timing constraints. These files serve as the raw input, allowing contestants to perform the most accurate routing resource and performance modeling. 

For each circuit, we also provide a set of simplified input files, which include a routing resource file (with a .cap extension) and a net information file (with a .net extension). The routing resource file follows the same format as used in the ISPD2024 contest, while the net information file is an extended version of the one used in ISPD2024. The routing resource file offers a detailed representation of the GCell grid graph and its available routing resources. The net information file provides the access points for all the pins within each net, along with the pin names and pre-routing stage slack estimates. These slack estimates provide a rough timing view of the circuit and enable contestants to perform net-based timing optimization. The simplified input files enable contestants to quickly engage with the contest and facilitate framing global routing challenges as mathematical optimization problems.

The output file conforms to the route segment file format compatible with the OpenROAD physical design flow. This format is similar to the output file format used in the ISPD2024 contest. The primary difference is that the ISPD2024 contest format describes route segments in the GCell coordinate system, whereas the route segment file format describes the route segments in the original layout coordinate system.

Here is an illustrative example of a global routing solution for a net:

    # Net name

    Net0

    (

    {$x_l$ $y_l$ $z_1$ $x_h$ $y_h$ $z_2$} 

    270 13230 M4 270 13230 M5

    270 13230 M5 270 13230 M6

    270 13230 M4 810 13230 M4

    810 13230 M4 810 13230 M3

    810 13230 M2 1350 13230 M2

    1350 13230 M2 1350 13230 M1

    )
where each row ($x_l$ $y_l$ $z_1$ $x_h$ $y_h$ $z_2$) describes a segment spanning from $(x_l, y_l, z_1)$ to $(x_h, y_h, z_2)$. For example, "270 13230 M4 810 13230 M4" represents a horizontal line in the routing layer M4. And "270 13230 M4 270 13230 M5" represents a via going from M4 to M5. Notice that the via segments can go from lower layers to upper layers (like in "270 13230 M4 270 13230 M5") or from upper layers to lower layers (like in "810 13230 M4 810 13230 M3").

To be considered valid, a global routing solution for a net must ensure that its wires cover all pins of the net and that the wires collectively form a connected graph. In this graph representation, each wire corresponds to a vertex. An edge exists between two vertices (wires) if they satisfy one of the following conditions: (i) They touch each other on the same metal layer, or (ii) Vias connect them. The resulting graph must be a connected structure. For an overall global routing solution to be deemed valid, it must satisfy the validity criteria for all nets in the circuit. 

Please check [Introduction of the contest](https://github.com/liangrj2014/ISPD25_contest/blob/main/etc/ISPD2025_contest_Performance_Driven_Large_Scale_Global_Routing.pdf) for more details. 

### Submission Guidance

Teams are required to build a Docker image on top of the provided [Dockerfile](https://github.com/liangrj2014/ISPD24_contest/blob/main/Dockerfile). Within the Docker environment, please create a directory named "router" under the **"/app"** folder and place the global router binary/scripts in this directory (**/app/router**). We expect that the global route can accept the following command line:


> ./route -library ${library folder} -def ${design}.def -v ${design}.v.gz -sdc ${design}.sdc -cap ${design}.cap -net ${design}.net -output ${design}.route

Notes: 
1) The router is not required to utilize all the provided input files. However, it should be capable of accepting the file names as inputs via the command.
2) Metal1 should not be used for net routing.
3) For stacked vias, such as a via from metal1 to metal3 represented by 409500 1614900 metal1 409500 1614900 metal3, the output format should be:
   
   "409500 1614900 metal1 409500 1614900 metal2
   
    409500 1614900 metal2 409500 1614900 metal3"
   
   Please see the reason here: https://github.com/liangrj2014/ISPD25_contest/issues/12
5) Since the OpenROAD router neither recognizes the GCELLGRID keyword in the DEF file nor supports manually specifying Gcell shapes (it only supports square Gcells), please ignore the GCELLGRID information in DEF files. Instead, use the Gcell definitions provided in the .cap files, which create Gcells with a fixed size of 4200 × 4200.
6) The alpha submission primarily serves to resolve formatting issues. The weights in the scoring function will be determined empirically based on the solutions from the alpha submissions. Alpha submission scores will be provided to each team for debugging purposes but will not be released publicly.
7) **Kindly name your Docker image as {TeamID}:beta, save it as {TeamID}_beta.tar.gz using the docker save command, and upload it to Google Drive. Please ensure the file is accessible to anyone with the link and share the link with us.**


During the evaluation process, the Docker images will be pulled and executed on a NVIDIA platform equipped with NVIDIA GPUs. Specifically, we will mount a "benchmarks" folder (containing the input files) to /app/benchmarks, a "NanGate45" folder (containing a "lib" folder, a "dbs" foler and a "lef" folder) to /app/NanGate45, and an "evaluation" folder (containing the evaluation scripts) to /app/evaluation. The evaluation script will be executed to run the submitted global router and evaluate the generated solutions. 
**Kindly send the link to your Docker image to ispd2025contest@gmail.com using the following format. Please set the email subject as "{TeamID} beta submission" and submit it by February 7, 2025.**


**Team ID      Link to the docker image**

######





Routing resource limit:
- RAM: 200 GB
- CPU Cores: 8 cores
- GPUs: 1 NVIDIA PG506-230 with 100GB
  
  

### Leaderboard

TBA

### Anouncement
- Registration opens on Sep 16, 2024!
- Released the contest introduction on Sep 13, 2024

### Registration

- Please fill in this [online registration form](https://form.jotform.com/242564578418164)
- Registration window: Sep 16, 2024 - Nov 30, 2024
- We confirm that we've received the registrations forms from the following teams. Please feel free to send us an email if we overlooked your registration forms or any related information.
  
  |  ID   | Team Name  | Affiliation |
  |  ----  | ----  | ----  |
  | 1  | Unknown | Unknown |
  | 2  | bllnghamton | Binghamton University |
  | 3  | Sai Harika | Arizona State University |
  | 4  | morse | IBM |
  | 5  | iloveEDA | National Yang Ming Chiao Tung University |
  | 6  | Frontier Design Automation | partner |
  | 7  | Impact Innovators | Cadence |
  | 8  | iitm_router | Indian Institute of Technology Madras |
  | 9  | Team Hippo | Peking University |
  | 10 | NTUGR | National Taiwan University |
  | 11 | iloveEDA | National Yang Ming Chiao Tung University |
  | 12 | Zaphod | Unknown |
  | 13 | MetaEDA | Institute of Science Tokyo |
  | 14 | metaRoute | Fudan University |
  | 15 | yzu_router | Yuan Ze University |
  | 16 | BisonCAD | North Dakota State University |
  | 17 | Anonymous | Unknown |
  | 18 | NaiveRoute | University of Science and Technology of China |
  | 19 | SGDrouter | Texas A&M University |
  | 20 | ^(*-(oo)-)^ | National Tsinghua University |
  | 21 | Routing_4-4 | Unknown |
  | 22 | Routing_team | Unknown |
  | 23 | NTHU-TCLAB-GR | National Tsinghua University |
  | 24 | runtu_eda | Unknown |
  | 25 | SCATOP3 | Institute of Computing Technology, Chinese Academy of Sciences |
  | 26 | etuReL2 | Fudan University |
  | 27 | FZU_Routing | Fuzhou University |
  | 28 | Kachow | National Yang Ming Chiao Tung University |
  | 29 | Swallow of Ambition | Institute of Science Tokyo |
  | 30 | SEU-Router | Southeast University |
  | 31 | It's MyRoute!!!!! | Fudan University, Wuhan University and University of California San Diego |
  | 32 | Dream wings | Fuzhou University |
  | 33 | NTHU-TCLAB-GR | National Tsinghua University |
  | 34 | ISMCEDA24 | South China University of Technology |
  | 35 | GODW Router | Unknown |
  | 36 | LX Router | Unknown |
  | 37 | RL-Route | The Chinese University of Hong Kong |
  | 38 | EDAteam | Unknown |
  | 39 | Bookworm Captain | Chung Yuan Christian University |
  | 40 | TeamName(){} | Fuzhou University |
  | 41 | no-idea | University of California, Berkeley |
  | 42 | Simprouter | Institute of Science Tokyo |
  | 43 | ADErouter | National Yang Ming Chiao Tung University |
  | 44 | FDU Team 3 | Fudan University |
  | 45 | routing_champs | Indian Institute of Technology Madras |
  | 46 | mRouter | Unknown |
  
  
  
### Important Dates

- Registration Open: Sep 16, 2024
- Registration Close: Nov 30, 2024
- Alpha Submission Deadline: Jan 12, 2025
- Beta Submission Deadline: Feb, 2, 2025
- Final Submission Deadline: Mar, 2, 2025
- Results Anouncement: March 19, 2025

### Downloads
- [Scripts that check the legality of the routing solution and compute the overflow score](https://drive.google.com/drive/u/1/folders/1zbemASyVpi7MttpQ3FtL0bbAyrA2-ock). Jan 12, 2025
- [Scripts to generate simplified input files from .def/.v files](https://drive.google.com/file/d/1BH5PZd9T3hppD4Fff739Y4vM3TtyoXL4/view?usp=sharing) Please make sure to use [our forked version of OpenROAD](https://github.com/liangrj2014/OpenROAD_ISPD25.git) to generate the simplified input files. The simplified input files are uloaded to [First set of testcases and example OpenROAD codes to evaluate global routing solutions](https://drive.google.com/drive/folders/12ei9JOKaMeSPgc9CZOWrVyLJRgb0f2is?usp=drive_link). Dec 25, 2024
- [An example of global routing segment file](https://drive.google.com/file/d/1HXlz7yF6MkJ7EJntViQhiOMfNZrMWgA9/view?usp=sharing) Oct 7, 2024
- [First set of testcases and example OpenROAD codes to evaluate global routing solutions](https://drive.google.com/drive/folders/12ei9JOKaMeSPgc9CZOWrVyLJRgb0f2is?usp=drive_link) Oct 7, 2024
- [Dockerfile for environment setup](https://github.com/liangrj2014/ISPD24_contest/blob/main/Dockerfile) Updated on Oct 7, 2024.
- [Introduction of the contest](https://github.com/liangrj2014/ISPD25_contest/blob/main/etc/ISPD2025_contest_Performance_Driven_Large_Scale_Global_Routing%20(1).pdf) Sep 13, 2024.

  
### Q&A

- Please post your questions in GitHub Issues

### Contact

- Email：~~ispd2024contest@gmail.com~~(Unfortunately, we lost access to the old email address:( ) ispd2025contest@gmail.com

### Contest Prizes

TBA

### Organizers

Rongjian Liang, Wen-Hao Liu, Anthony Agnesina and Haoxing Ren from NVIDIA

### Sponsor
 - Sponsored by NVIDIA
   
<img width="600" alt="profile" src="etc/nvidia_logo.png">
