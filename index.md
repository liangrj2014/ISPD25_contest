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

Teams are required to build a Docker image on top of the provided [Dockerfile](https://github.com/liangrj2014/ISPD24_contest/blob/main/Dockerfile). Within the Docker environment, please create a directory named "router" under the **"/app"** folder and place the global router binary/scripts in this directory (**/app/router**, rather than **/workspace/app/router**). We expect that the global route can accept the following command line:


> ./route -library ${library folder} -def ${design}.def -v ${design}.v.gz -sdc ${design}.sdc -cap ${design}.cap -net ${design}.net -output ${design}.route

**Please kindly check [run_evaluation.sh](https://drive.google.com/file/d/1U9BvXxsjDbewBR1jb6kznb52vui4f5N6/view?usp=drive_link) for details.**

Notes: 
1) The router is not required to utilize all the provided input files. However, it should be capable of accepting the file names as inputs via the command.
2) Metal1 should not be used for net routing.
3) For stacked vias, such as a via from metal1 to metal3 represented by 409500 1614900 metal1 409500 1614900 metal3, the output format should be:
   
   "409500 1614900 metal1 409500 1614900 metal2
   
    409500 1614900 metal2 409500 1614900 metal3"
   
   Please see the reason here: https://github.com/liangrj2014/ISPD25_contest/issues/12
5) Since the OpenROAD router neither recognizes the GCELLGRID keyword in the DEF file nor supports manually specifying Gcell shapes (it only supports square Gcells), please ignore the GCELLGRID information in DEF files. Instead, use the Gcell definitions provided in the .cap files, which create Gcells with a fixed size of 4200 × 4200.
6) The alpha submission primarily serves to resolve formatting issues. The weights in the scoring function will be determined empirically based on the solutions from the alpha submissions. Alpha submission scores will be provided to each team for debugging purposes but will not be released publicly.
7) **Kindly name your Docker image as {TeamID}:final, save it as {TeamID}_final.tar.gz using the docker save command, and upload it to Google Drive. Please ensure the file is accessible to anyone with the link and share the link (please share the link to the Docker image file rather than the link to the folder) with us.**
8) Evaluation metrics details:
   
   original_score = w1*(WNS -WNS_{ref}) + w2*(TNS - TNS_{ref})/N_{endpoint} + w3*(TotalPower - TotalPower_{ref}) + w4*OverflowScore

   **scaled_score = original_score * (1 + signed_runtime_factor)**
   
   **T = 0.02*log_{2}(𝐺𝑅𝑜𝑢𝑡𝑒𝑟_𝑊𝑎𝑙𝑙_𝑇𝑖𝑚e/𝑀𝑒𝑑𝑖an_𝑊𝑎𝑙𝑙_𝑇𝑖𝑚e)**
   
   **signed_runtime_factor = sign(original_score) * min(0.2, max(-0.2, T))**
   
   where WNS_{ref}, TNS_{ref}, and TotalPower_{ref} are the **median** WNS, TNS and total power of submitted global routing solutions, and OverflowScore is the congestion cost reported by the evaluator. Our overall principle is: overflow > timing > power, and runtime efficiency matters :)

 |  Testcase (visible)   | w1  | w2 | w3 | w4 | N_{endpoint} | {N_net} | GCell graph dimensions |
  |  ----  | ----  | ----  | ----  | ---- | ---- | ---- | ---- | 
  | ariane  | **-10** | **-100** | 300 | 3e-7 | 20218 | 123900 | 10x761x761 |
  | bsg  | **-10** | **-100** | 25 | 4e-8 | 214821 | 736883 | 10x1384x1384 |
  | NVDLA  | **-0.05** | **-0.5** | 25 | 1.5e-7 | 45925 | 199481 | 10x1120x1120 |
  | mempool_tile  | **-1** | **-10** | 300 | 7e-7 | 13350 | 136120 | 10x428x428 |
  | mempool_group  | **-1** | **-10** | 20 | 3e-8 | 347869 | 3274611 | 10x1611x1610 |
  | mempool_cluster  | **-1** | **-10** | 0.3 | 5e-9 | 1082397 | 12047279 | 10x3175x3175 |

 |  Testcase (blind)   | w1  | w2 | w3 | w4 | N_{endpoint} | {N_net} | GCell graph dimensions |
  |  ----  | ----  | ----  | ----  | ---- | ---- | ---- | ---- | 
  | ariane  | -0.2 | -2 | 100 | 0.0000004 | 20218 | 105924 | 10x646x646 |
  | bsg  | -0.1 | -1 | 50 | 0.00000002 | 214915 | 768239 | 10x1384x1384 |
  | NVDLA  | -0.01 | -0.1 | 100 | 0.0000001 | 45925 | 157744 | 10x1120x1120 |
  | mempool_tile  | -3 | -30 | 100 | 0.000001 | 13350 | 135814 | 10x386x386 |
  | mempool_group  | -0.5 | -5 | 3 | 0.00000004 | 347869 | 3218496 | 10x1611x1610 |
  | mempool_cluster  | -0.4 | -4 | 2 | 0.00000001 | 1082397 | 12168735 | 10x3719x3719 |

  

During the evaluation process, the Docker images will be pulled and executed on a NVIDIA platform equipped with NVIDIA GPUs. Specifically, we will mount a "benchmarks" folder (containing the input files) to /app/benchmarks, a "NanGate45" folder (containing a "lib" folder, a "dbs" foler and a "lef" folder) to /app/NanGate45, and an "evaluation" folder (containing the evaluation scripts) to /app/evaluation. The evaluation script will be executed to run the submitted global router and evaluate the generated solutions. 
**Kindly send the link to your Docker image to ispd2025contest@gmail.com using the following format. Please set the email subject as "{TeamID} final submission" and submit it by March 7, 2025 (AOE).**


**TeamID       Link_to_the_Docker_image**

**!!!Note that we added the PPA weights to the second line of the cap files so that teams can optimize the scores according to the weights. Please remember to update your parser!!!**

######





Routing resource limit:
- RAM: 200 GB
- CPU Cores: 8 cores
- GPUs: 1 NVIDIA PG506-230 with 100GB
  
  

### Leaderboard

Beta submission

| Design (visible) | WNS_{ref} | TNS_{ref} | Total_power_{ref} | Median_wall_time/s | WNS | TNS | Total_power | Congestion | runtime | Original_score | Scaled_score |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | 
| Ariane | -0.485 | -1398.39 | 0.646 | 9.5 | -0.45 | -1355.39 | 0.646 | 5836484.4516 | 6 | 1.188263567 | 1.172508023 |
| Bsg_chip | -0.44 | -10802.7 | 3.05 | 32.5 | -0.44 | -10719.7 | 3.05 | 38128669.4702 | 26 | 1.48647272 | 1.476901973 |
| NVDLA | -94.78 | -669471 | 2.96 | 17 | -75.87 | -594382 | 2.94 | 13885843.4112 | 11 | -0.180141833 | -0.182404527 |
| Mempool_tile | -0.695 | -3590.83 | 0.1455 | 10 | -0.69 | -3580.1 | 0.145 | 1984587.6026 | 29 | 1.226177614 | 1.263847088 |
| Mempool_group | -0.815 | -41740.2 | 7.82 | 111.5 | -0.34 | -23173.8 | 7.77 | 57209536.75 | 65 | -0.292434493 | -0.296987886 |
| Mempool_cluster | -0.68 | -79748 | 23.7 | 372.5 | -0.34 | -68724.1 | 23.7 | 240104020.7 | 265 | 0.75867323 | 0.751219295 |




Note that we only evaluated the beta submissions of a few teams on the hidden testcase. And the below statistics are just for your reference.
| Design (blind) | WNS_{ref} | TNS_{ref} | Total_power_{ref} | Median_wall_time/s | WNS | TNS | Total_power | Congestion | runtime | Original_score | Scaled_score |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | 
| Ariane | -1.67 | 524.66 | 0.156 | 36 | -0.16 | -524.66 | 0.156 | 4418340 | 36 | 1.721748047 | 1.721748047 |
| Bsg_chip | -2.85 | -3582 | 0.305 | 31 | 0 | 0 | 0.305 | 130832985.7308 | 44 | 2.314985367 | 2.338377614 |
| NVDLA | -50.655 | -194395 | 0.137 | 15.5 | -66.93 | -265240.31 | 0.137 | 14602048.9336 | 20 | 1.777217985 | 1.790288776 |
| Mempool_tile | -0.59 | -2545.31 | 0.146 | 9 | -0.59 | -2545.31| 0.145 | 1731121.9681 | 9 | 1.631121968 | 1.631121968 |
| Mempool_group | -0.58974 | -38679.56641 | 8.50774956 | 74 | -0.52224779 | -37333.29297 | 8.48 | 53239999.88 | 58 | 1.978705632 | 1.964796425 | 
| Mempool_cluster | -0.34325 | -55970.21094 | 24.4876995 | 336 | -0.24935344 | -44483.90625 | 2.4208 | 270405583.1 | 1408 | 2.063907288 | 2.149233929 |


Final Submissions

Note that the rankings of the teams are based on the results on blind testcases.

Results on visible testcases:

1st place
| Design (visible) | WNS | TNS | Total_power | Congestion | runtime | 
| ---- | ---- | ---- | ---- | ---- | ---- | 
| Ariane | -0.432401|-1325.782227 |0.646092415 | 5846460.692| 7|
| Bsg_chip | -0.4201518|-10366.19336 | 3.05441332| 21332547.19|29 |
| NVDLA | -56.91934586| -521344.5625|2.94053936 |13315606.243 |10 |
| Mempool_tile | -0.37867799| -1610.013794| 0.143104538| 3019425.0695| 7|
| Mempool_group |-0.30521387 |-20473.81055 |7.69207478 | 55005746.4225| 120|
| Mempool_cluster | -0.26587465| -59712.49219| 23.4496517| 236920426.9094| 468|

2nd place
| Design (visible) | WNS | TNS | Total_power | Congestion | runtime | 
| ---- | ---- | ---- | ---- | ---- | ---- | 
| Ariane |-0.39832926 |-1340.189453 | 0.646077931| 5902618.739| 7|
| Bsg_chip |-0.42405224 | -10774.87207| 3.05300736| 20468992.4856| 22|
| NVDLA |-55.84056473 |-514660.2813 | 2.94011188| 13686713.5105| 20|
| Mempool_tile | -0.48879835| -2225.456055| 0.145150483| 2478317.562| 12|
| Mempool_group | -0.34660277| -21709.12891| 7.73386288| 49071539.851| 285|
| Mempool_cluster | -0.31490123| -62459.98047| 23.5781002| 192012708.2409| 1018|

3rd place
| Design (visible) | WNS | TNS | Total_power | Congestion | runtime | 
| ---- | ---- | ---- | ---- | ---- | ---- | 
| Ariane |-0.38071746 |-1138.369019 |0.646074593 | 8003814.629| 18|
| Bsg_chip |-0.40696481 | -9344.333008|3.05345058 |25412296.28 | 42|
| NVDLA | -55.91420746| -514674.25| 2.93805909| 15687876.1569| 21|
| Mempool_tile |-0.51995385 | -2524.328857| 0.145220235| 3469019.268| 19|
| Mempool_group |-0.2752274 | -19338.69531| 7.5529623| 97506552.6193 | 201|
| Mempool_cluster | -0.27623194| -59533.47266| 23.2199707| 289712328.3253| 726|

Results on blind testcases:
1st place
| Design (blind) | WNS_{ref} | TNS_{ref} | Total_power_{ref} | Median_wall_time/s | WNS | TNS | Total_power | Congestion | runtime | Original_score | Scaled_score |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | 
| Ariane |-1.628424105 | -523.0376587| 0.15612001| 19 | -1.75610375| -650.1010742| 0.156143963| 4349151.569| 10| 1.780161242| 1.747192677|
| Bsg_chip | 0| 0| 0.304779679| 21656927.89| -3.98435068| -2859.788086| 0.304725498| 22567636.29| 44| 0.860385344| 0.866786074|
| NVDLA | 0| 0| 0.13628386| 14281178.56| 0| 0| 0.136246324| 14061155.6212| 12| 1.402362012| 1.380693009|
| Mempool_tile | -0.48325035| -1920.723755| 0.144852504| 2122543.077| -0.34229153| -1356.072144| 0.14302583| 2232008.372| 9| 0.357584486| 0.355514025|
| Mempool_group | -0.67990416| -40487.88281| 8.54667377| 50934910.6| -0.43341908| -32016.09961| 8.36879635| 52783070.92| 117| 1.332681176| 1.330446761|
| Mempool_cluster | -0.289977835| -53248.99805| 24.26071835| 236560368.8| -0.22249137| -42564.08203| 24.2439041| 242737067.8| 414| 2.327261467| 2.32799536|

2nd place
| Design (blind) | WNS | TNS | Total_power | Congestion | runtime | Original_score | Scaled_score |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| Ariane | -0.73648113| -149.5187683| 0.156133637| 4651055.702| 12| 1.646447292| 1.624616553|
| Bsg_chip | 0| 0| 0.304772198| 21307154.05| 21| 0.425769031| 0.419849603|
| NVDLA | 0| 0| 0.136257023| 16352692.6| 20| 1.63258561| 1.631422429|
| Mempool_tile | -0.47176221| -1920.723755| 0.144852504| 2122543.077| 13| 2.088078657| 2.098143535|
| Mempool_group | -0.4597598| -34846.05078| 8.35858917| 49722314.53| 281| 1.23347527| 1.262590961|
| Mempool_cluster | -0.29351634| -50372.4375| 24.380127| 214110807| 1038| 2.370710438| 2.434334194|

3rd place
| Design (blind) | WNS | TNS | Total_power | Congestion | runtime | Original_score | Scaled_score |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| Ariane | -0.63291508| -162.3764648| 0.156107366| 4386728.17| 24| 1.518647876| 1.528884625|
| Bsg_chip | -2.74678063| -2936.033936| 0.304783702| 20564539.87| 48| 0.699831383| 0.706794701|
| NVDLA |0 | 0| 0.136282533| 13637302.35| 22| 1.363597585| 1.366376041|
| Mempool_tile | -0.44003779| -1806.78894| 0.145195842| 2168599.635| 22| 1.817262464| 1.853607713|
| Mempool_group | -0.3730889| -26801.4375| 7.94192028| 132027917.5| 185| 3.11673022| 3.152708828|
| Mempool_cluster | -0.20036262| -39516.22656| 23.9679279| 335020596.9| 672| 2.678029502| 2.716303781|


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
- Final Submission Deadline: ~~Mar, 5, 2025~~ Mar, 7, 2025 (Anywhere on earth, and it is a hard deadline)
- Results Anouncement: March 19, 2025

### Downloads
- [Hidden testcase](https://drive.google.com/drive/folders/1J3yoVZ07ifQiJ8l7SQ_J1Y5y0zXcEVmR?usp=drive_link). Mar 28, 2025
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

1st place: $1000 + one NVIDIA GPU of similar value

2nd place: $500 + one NVIDIA GPU of similar value

3rd place: $250 + one NVIDIA GPU of similar value


### Organizers

Rongjian Liang, Wen-Hao Liu, Anthony Agnesina and Haoxing Ren from NVIDIA

### Sponsor
 - Sponsored by NVIDIA
   
<img width="600" alt="profile" src="etc/nvidia_logo.png">
