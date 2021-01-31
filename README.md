**Nurse Scheduling Problem using Discrete Quadratic Model**

**Team QuACK**

**Problem Statement**

We performed DQM (Discrete Quadratic Model) implementation of Nurse Scheduling Model developed by Ikeda, Nakamura and Humble (INH) using hybrid quantum annealing. 
The objective of the NSP (Nurse Scheduling problem) is to find the optimal way of assigning shifts and rests to nurses to ensure maximum productivity and satisfaction. 

For the INH NSP we had to assign nurses to shifts complying by three constraints: 
1)	hard nurse constraint: no consecutive shifts for a nurse;
Giving nurses enough rest to avoid burn-outs
2)	hard shift constraint: at least one nurse present for one shift;
Ensuring the service of the nursing team 
3)	soft nurse constraint: nurses should have “approximately” even schedule (the “softness” of the constraint is due to the approximation) ;
Ensuring equality of work and efficiency of the nursing team

This is an important problem to solve for service-dependent large organizations where workforce management takes top priority for it to function. And for a hospital it is critical that it is staffed sufficiently with a satisfied workforce as a deviation from optimum management means a matter of life and death. 
For this problem, the hard constraints are obligatory to be met while we try to minimize the penalty cost for violating the soft constraint meaning that the soft nurse constraint energy can be non-zero whereas the hard constraint energies must be zero.

The nurse scheduling problem is a Non-deterministic Polynomial Time Hard Problem. (NP-hard problem) This means with the increase of configuration size it becomes difficult to find a feasible solution to this problem. This is where hybrid quantum annealing methods comes in to improve on the classical methods of scheduling. 

This read me contains a holistic overview of the problem and the approaches we took to solve it. 

**Discrete Quadratic Model (DQM)**

The DQM is a quadratic polynomial which takes discrete variables such as {yellow, red, green} or {6.77, 3.45, 33.44}. Using DQM, encoding for discrete variable problems has become easier. The change from binary variables of BQM to discrete variables opens the door to solve new types of problems using a quantum computer. 
The DQM function can be represented by a general form of:


<p align="center">
<img src="https://render.githubusercontent.com/render/math?math=E(\bf{x})= \sum_{i=1}^N \sum_{u=1}^{n_i} a_{i,u} x_{i,u}+ \sum_{i=1}^N \sum_{j=i+1}^N \sum_{u=1}^{n_i} \sum_{v=1}^{n_j} b_{i,j,u,v} x_{i,u} x_{j,v}+ c,">
</p>

where there are N discrete variables with  n_i cases each. 

**Hybrid Solvers**
Hybrid solvers include the use of both CPU and QPU to solve a problem. Hybrid solvers ensure the optimum use of the QPU and delegate other tasks to the CPU and thus ensuring faster results and drastically increase the scope for the configuration space of the problems that we are solving. 

**Formulations of the DQM**

hard nurse constraint: no consecutive shifts for a nurse
```bash

# G.add_edges_from([(0, 1), (1, 2),(2, 3), (4, 5),(5, 6), (6, 7), (8, 9),(9, 10),(10, 11)]) #edges_for_4
# Parameters for hard nurse constraint
# a is a positive correlation coefficient for implementing the hard nurse
# constraint - value provided by Ikeda, Nakamura, Humble

for p in G.nodes:
    dqm.add_variable(num_colors, label=p)
for p in G.nodes:
    dqm.set_linear(p, colors)
for p0, p1 in G.edges:
    if(p0,p1) in cons_1_valid_edges:
        dqm.set_quadratic(p0, p1, {(c, c): a for c in colors})

```
hard shift constraint: at least one nurse present for one shift

```bash
lagrange_hard_shift = 1.3
workforce = 1     # Workforce function W(d) - set to a constant for now
effort = 1 

linear_term = np.linspace(0,1,2)*(lagrange_hard_shift * (effort ** 2 - (2 * workforce * effort)))
for p in G.nodes:
    dqm.set_linear(p,linear_term)
for p0, p1 in G.edges:
    if(p0 != p1) and (p0,p1) in cons_2_valid_edges:
        dqm.set_quadratic(p0, p1, {(c, c): 2 * lagrange_hard_shift * effort ** 2  for c in colors})

```

soft nurse constraint: nurses should have “approximately” even schedule

```bash
lagrange_soft_nurse = 0.3

linear_term_cons3 = np.linspace(0,1,2)*(lagrange_soft_nurse * (preference ** 2 - (2 * min_duty_days * preference)))
for p in G.nodes:
    dqm.set_linear(p,linear_term_cons3)
for p0, p1 in G.edges:
    if(p0 != p1) and (p0,p1) in cons_3_valid_edges and (p0,p1) not in cons_1_valid_edges and (p0,p1) not in cons_2_valid_edges :
        dqm.set_quadratic(p0, p1, {(c, c): 2 * lagrange_soft_nurse * preference ** 2  for c in colors})

```


**Code Input and Output**
The code takes an input of the number of nurses and number of number of days and gives an output of an optimized schedule. 

**Applications:**
Nurse Scheduling by hybrid QA can be used in hospitals to minimize operations as well as maximize the satisfaction of the nursing teams. Volkswagen currently uses D-Wave’s hybrid solver to optimize its paint shop scheduling applications. Hospitals can follow this example and can use our DQM nurse scheduling solution to boost their organizational efficiency.  Other industries also could follow and use this as a solution to their resource allocation problem and maximize their productivity of their workforce. 

**References:**
1)	https://www.nature.com/articles/s41598-019-49172-3
2)	https://github.com/dwave-examples/nurse-scheduling
3)	https://docs.dwavesys.com/docs/latest/index.html





