# ISPD25 Contest: GPU/ML-Enhanced Large Scale Global Routing

<img width="1000" alt="profile" src="etc/ispd_logo.png">

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

To enable teams from diverse backgrounds to participate, we have extracted routing resource information and netlist data from LEF and DEF files and organized them in simplified formats. Consequently, participants can approach the contest as a mathematical optimization problem within the GCell grid graph. The desired outcome is global routing solutions described within the GCell grid graph. The evaluation process is centered on several key metrics, including total wirelength, via count and routing congestion of the global routing solution, as well as the execution runtime of the global router.

Please check [Introduction of the contest](https://drive.google.com/file/d/11wSwOaLQ0ZMEq2Znb3gjCAtLhNn1DV1w/view?usp=sharing) for more details. 

### Submission Guidance

Teams are required to build a Docker image on top of the provided [Dockerfile](https://github.com/liangrj2014/ISPD24_contest/blob/main/Dockerfile). Within the Docker environment, please create a directory named "router" under the "/workspace" folder and place the global router binary/scripts in this directory (/workspace/router). We expect that the global route can accept the following command line:

> ./route -cap $data.cap -net $data.net -output $data.output

During the evaluation process, the Docker images will be pulled and executed on a NVIDIA platform equipped with 4 NVIDIA A100 GPUs. Specifically, we will mount a "benchmarks" folder (containing the input files) to /workspace/benchmarks and an "evaluation" folder (containing the evaluation scripts) to /workspace/evaluation. The [evaluation.sh](https://drive.google.com/file/d/1V_5WSwkD8uk_IJw07m0vjWyCROsK4vfj/view?usp=sharing) script will be executed to run the submitted global router and evaluate the generated solutions.

Please kindly archive your Docker image to a tar file (refer to https://docs.docker.com/engine/reference/commandline/save/). Subsequently, upload the tar archive to Google Drive and share the link with us. Please grant access to the Docker image for anyone with the link. The global router binary/scripts are expected to be stored in /workspace/router within the Docker image. Therefore, there is no need to upload the global router binary/scripts separately. If you have any questions regarding this, please feel free to ask. Thank you!

Routing resource limit:
- RAM: 200 GB
- CPU Cores: 8 cores
- GPUs: 4 NVIDIA A100 GPUs
  
  

### Leaderboard



### Anouncement
- We released the hidden benchmarks and contest results- March 26, 2024
- We released the slides for the contest results/summary - March 19, 2024
- We released the routing resource limit - Feb 2, 2024
- We released a testcase with around 50M cells and 60M nets! - Jan 29, 2024
- We evaluated the alpha submissioms on public benchmarks and created the leader board. - Jan 22, 2024
- We updated the submission guidance. -Jan 8, 2024
- We updated the benchmark input files. And a new design ("cluster") with around 10 million cells is released. - Jan 4, 2024
- We postponed the alpha submission date to Jan 12, 2024. - Jan 2, 2024.
- We updated the global routing solution format and evaluation metrics. Please kinldy check the updated [Introduction of the contest](https://drive.google.com/file/d/1YiDORsgiImMg6vIO6EfwFj4VNg8Hb5k3/view?usp=sharing). - Jan 2, 2024.
- We released the updated version of evaluation script. - Jan 2, 2024.
- The evaluation platform is configured with CUDA version 11.7 and driver version 515.00. We have prepared a Dockerfile (https://github.com/liangrj2014/ISPD24_contest/blob/main/Dockerfile) that will install deep learning toolkits compatible with the CUDA version on our evaluation platform. Generally, participants are welcome to modify the Dockerfile as necessary, ensuring compatibility with our evaluation platform for utilizing the GPUs on the system. - Dec 02, 2023.
- We've identified some bugs in our evaluation script (thanks to the participants for bringing them to our attention!). An updated version will be released shortly, addressing these bugs and significantly improving runtime speed. Stay tuned for the latest updates! - Nov 30, 2023.
- We've released example global routing solutions on Oct 28, 2023.
- We've released the evaluation scripts on Oct 28, 2023.
- To simplify the entry process for the competition, we've extracted routing resource information and netlist data from LEF and DEF files and stored them in simplified formats. As a result, participants can tackle the contest challenge as a mathematic optimization problem within the GCell grid graph. - Oct 28, 2023.
- Registration opens on Sep 13, 2023!
- Released the first set of benchmarks on Sep 13, 2023
- Released the docker for environment setup on Sep 13, 2023

### Registration

- Please fill in this [online registration form](https://form.jotform.com/232454622032143)
- Registration window: Sep 16, 2023 - Dec 1, 2023

  
### Important Dates

- ~~Registration Open: Sep 13, 2023~~
- ~~Registration Close: Dec 1, 2023~~
- Alpha Submission: ~~Jan 5, 2024~~ Jan 12, 2024
- Beta Submission: Feb, 2, 2024
- Final Submission: Mar, 1, 2024
- Results Anouncement: March 15, 2024

### Downloads
- [Final contest result](https://github.com/liangrj2014/ISPD24_contest/blob/main/16_1_contest_slides_final.pptx) - March 19, 2024.
- [Introduction of the contest](https://drive.google.com/file/d/1yEgcjHAZOyFHKlfYhzHe8ZeZuefEK2sP/view?usp=drive_link) Note that the introduction has been updated on Jan 29, 2024.
- [First set of benchmarks with Nangate45 technology node](https://drive.google.com/drive/folders/1afrsbeS_KuSeHEVfuQOuLWPuuZqlDVlw?usp=sharing)
Please note that all the essential input information for global routing is contained within the .cap files and .net files located in the "Simple_inputs" folder. We also release the LEF/DEF files of the circuits just for reference. Note that the benchmarks have been updated on Jan 29, 2024.
- [Evaluation Scripts](https://drive.google.com/drive/folders/1VTnIFtCa6X7cRRx9xBtPDu-kHPdnhCzL?usp=sharing) Note that the evaluation scripts have been updated on Jan 29, 2024.
- [Example global routing solutions](https://drive.google.com/drive/folders/1901Cn31zsq1bNs8lrHBC_CUwEF0eUk8Z?usp=drive_link) Note that the example solutions have been updated on Jan 03, 2024.
- [Dockerfile for environment setup](https://github.com/liangrj2014/ISPD24_contest/blob/main/Dockerfile) Note that the Dockerfile has been updated on Dec 02, 2023.
  
### Q&A

- Please post your questions in GitHub Issues

### Contact

- Email：ispd2024contest@gmail.com

### Contest Prizes
- 1st place: $1000 + one NVIDIA GPU of similar value
- 2nd place: $500 + one NVIDIA GPU of similar value
- 3rd place: $250 + one NVIDIA GPU of similar value

### Sponsor
 - Sponsored by NVIDIA
   
<img width="600" alt="profile" src="etc/nvidia_logo.png">
