
# Microworkflows

Making advanced simulation tools more accessible and impactful

Improving Usability and Broadening Adoption of Simulation with Microworkflows
- related publication

## motivation
- advanced modeling and simulation of products leads to
	- faster time to market
	- cost saving on prototypes
- bu there are barriers to adoption
	- gen-purpose sim codes, software licenses, training, 
	- expert sim-workflow consultation for complexity

## goals
- improve usability
	- show only whats needed for specific use cases
	- hide the machine-readable complexity
- broaden adoption of sim tech
	- computational science engineers handle complexity
	- stakeholders can use results to perform simulations and run analysis standalone

simulation adoption is cumbersome to use, steep learning curves, and analyzing test results is hard to comprehend


### Workflows components

- Define Experiments
	- model verification, parameter estimation, optimizations, and inference
- Depict Domain
	- space of interest, discrete geometry, CAD models
- Describe Simulation
	- collection of files for roadmap for simulation
- Discretize Domain
	- discretize continuous surface or volume ...
- Execute Simulation
- Analyze and Visualize Results


### How do we?
- reduce the complexity involved in carrying out a sim workflow?
- make it easier to learn how to configure componentss of a workflow
- andd what about communicating with stakeholders?


## Microworkflows
- bundle workflow components inttot small complete applications
- expose modeling and simulation capabilities through a limited set of predefined use cases
- define use cases with templates and present limited set of configuration options

#### Defining a microworkflow
- Computational Science Engineer(CSE) w/ knowledge of full simulation workflow formulates a use casse with sstakeholders
- Stakeholder and CSE id parameteers toi explore for sim exp
- CSE maps these parameters ....



The web is a pseudo application delivery mechanism for those under IT administration.

the workflow is well-defined, basically a application core-driven workflow,

the micro part is the laser-focus goal that we are doing. you pick a template, define parameters, and run simulation, then partition results you want. 
- the template structure is creating the simulation itself

the orchestration of core app functionality in the microworkflow simpler to debug because it is hiding behind integrated cross-platform libraries, right?  
- 
  
what are the disadvantages relying on efficient application delivery mechanism through microworkflows?  
- maybe resource-allocation
  
Also what environments will work best in for microworkflows?
- computational environments
- local-first and serverless
- remote connection to your browser running on an HPC GUI-less