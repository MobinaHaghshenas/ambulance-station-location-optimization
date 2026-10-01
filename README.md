# Ambulance Station Location & Allocation Optimization

A mixed-integer optimization model for determining EMS station locations, station types, ambulance allocation, and emergency coverage across a network of 65 demand areas.

The project formulates an Emergency Medical Services (EMS) network design problem as a mathematical optimization model and solves it using Python, PuLP, and the CBC solver.

The model integrates:

- EMS station location
- Station type selection
- Ambulance allocation
- First-stage coverage
- Second-stage coverage
- Population-per-ambulance constraints
- Operating and response-related costs
- Net Present Value (NPV) over a five-year planning horizon

---

## Project Overview

Emergency medical service systems must balance two competing requirements:

1. Providing sufficient geographic and population coverage
2. Controlling the infrastructure and operational cost of the EMS network

The purpose of this project was to develop a mathematical optimization model that determines an efficient configuration of EMS stations and ambulances while satisfying predefined coverage and capacity requirements.

Instead of evaluating station locations independently, the model jointly decides:

- Which candidate locations should host an EMS station
- Which station type should be selected
- How many ambulances should be assigned to each station
- Which demand areas receive first-stage coverage
- Which demand areas receive second-stage coverage
- How the resulting configuration affects the total system cost over the planning horizon

---

## Problem Structure

The study considers:

- **65 demand areas / candidate sites**
- **Two station types:** Type A and Type B
- **Two ambulance types:** Type A and Type B
- **First-stage emergency coverage**
- **Second-stage backup coverage**
- **A five-year planning horizon**

The model uses population, demand, coverage relationships, station costs, ambulance costs, and response-related parameters to determine the optimized EMS configuration.

---

## Optimization Workflow

The overall analytical workflow is:

![Optimization Workflow](results/figures/optimization-workflow.png)

```text
EMS Planning Problem
        ↓
65 Demand Areas + Candidate Sites
        ↓
Coverage Relationships
        ↓
Decision Variables
        ↓
Mixed-Integer Optimization Model
        ↓
Cost + NPV Objective
        ↓
CBC Optimization
        ↓
Optimal EMS Configuration
Optimal EMS Configuration
Mathematical Optimization Model

The core of this project is a mixed-integer mathematical programming model.

The model contains both:

Binary decision variables for station selection and coverage assignment
Integer decision variables for ambulance allocation

The objective is to minimize the total cost of the EMS network while satisfying coverage, station, and ambulance-capacity constraints.

Decision Variables
First-Stage Assignment Variables

For each demand area i and eligible station location j:

$$ X_{ij}^{A} $$

is a binary variable equal to 1 when demand area i is assigned to a Type-A station at location j.

Similarly,

$$ X_{ij}^{B} $$

is a binary variable equal to 1 when demand area i is assigned to a Type-B station at location j.

Second-Stage Coverage Variables
$$ X_{ij}^{S} $$

is a binary variable representing second-stage coverage of demand area i by station j.

These variables model backup coverage relationships between demand areas and EMS stations.

Ambulance Allocation Variables

The number of ambulances assigned to each station is represented by:

$$ n_j^A $$

for Type-A ambulances, and

$$ n_j^B $$

for Type-B ambulances.

Both are integer decision variables.

Objective Function

The optimization model minimizes the total cost of establishing and operating the EMS network:

$$ \min Z = \sum_j CFA X_{jj}^{A} + \sum_j CFB X_{jj}^{B} + \sum_j CA n_j^A + \sum_j CB n_j^B + NPV $$

where:

CFA = Type-A station cost
CFB = Type-B station cost
CA = Type-A ambulance cost
CB = Type-B ambulance cost
NPV = Net Present Value of the modeled operating/response-related costs

The implementation uses the following cost parameters:

Parameter	Value
Type-A ambulance cost (CA)	56,202
Type-B ambulance cost (CB)	47,869
Type-A station cost (CFA)	494,152
Type-B station cost (CFB)	169,948
Coverage Constraints
First-Stage Coverage

Each demand area must receive first-stage coverage from an eligible station.

Conceptually:

$$ \sum_{j \in F_i} \left( X_{ij}^{A} + X_{ij}^{B} \right) = 1 \qquad \forall i $$

where F_i represents the set of stations capable of providing first-stage coverage to demand area i.

This ensures that every demand area is assigned to a primary EMS station.

Second-Stage Coverage

The model also incorporates predefined second-stage coverage relationships.

For demand areas requiring backup coverage, the model determines whether an eligible station provides the required second-stage service.

The candidate second-stage relationships are explicitly encoded in the optimization model.

This allows the network to account for both primary and backup emergency coverage.

Station Type Constraints

The model distinguishes between Type-A and Type-B stations.

Candidate station eligibility is encoded through the model's station-type parameters.

The model prevents a location from simultaneously being selected as both station types.

Conceptually:

$$ X_{jj}^{A} + X_{jj}^{B} \leq 1 $$

for each candidate location j.

Station–Ambulance Consistency

The ambulance allocation must be consistent with the selected station configuration.

For Type-A stations, the number of allocated Type-A ambulances is linked to the station activation variable.

For Type-B stations, the corresponding ambulance allocation is linked to the Type-B station decision.

This prevents the model from allocating ambulances to a station configuration that has not been selected.

Population-per-Ambulance Constraint

The model also controls the amount of population served by each ambulance.

The implementation specifies:

$$ P_{min}=35,000 $$

and

$$ P_{max}=55,000 $$

people per ambulance.

The resulting station allocation must therefore satisfy the population-per-ambulance limits encoded in the model.

This constraint links:

Population
Station assignment
Number of ambulances

and prevents the optimization from creating configurations with excessive population assigned to a single ambulance.

Net Present Value (NPV)

The model incorporates the economic effect of the planning horizon through a Net Present Value formulation.

The first-year cost is calculated from demand, population, response-related parameters, coverage assignments, and second-stage coverage.

The model then calculates NPV over the planning horizon:

$$ NPV = C_1 \frac{ 1-(1+g)^T(1+h)^{-T} }{ h-g } $$

when:

$$ h \neq g $$

where:

C₁ = first-year modeled operating/response-related cost
g = annual growth rate
h = discount rate
T = planning horizon

The implementation uses:

Parameter	Value
Planning horizon (T)	5 years
Discount rate (h)	0.08
Growth rate (g)	0.0217
Second-stage probability parameter (Ps)	0.10
Minimum population per ambulance	35,000
Maximum population per ambulance	55,000
Daily calls	120

The NPV component allows the optimization to consider the economic implications of the EMS configuration beyond a single year.

Model Implementation

The optimization model was implemented in Python using PuLP.

The main steps are:

Define the number of demand areas
Define population and demand parameters
Define candidate station locations
Define first-stage coverage relationships
Define second-stage coverage relationships
Create binary station and coverage variables
Create integer ambulance allocation variables
Add coverage constraints
Add station-type constraints
Add ambulance allocation constraints
Calculate first-year operating/response-related cost
Calculate NPV
Construct the total objective function
Solve the mixed-integer optimization problem
Extract the selected stations and ambulance allocation

The model is solved using the CBC mixed-integer optimization solver through PuLP.

Optimization Results

The optimization run reported:

Status: Optimal

The resulting configuration selects 14 EMS station locations.

The selected locations are:

4, 8, 11, 12, 17, 23, 24, 25, 29, 30, 32, 33, 47, 51

The optimized configuration contains:

20 Type-A ambulances
9 Type-B ambulances
29 ambulances in total
Selected Station Configuration
Site	Station Type	Ambulances	Population / Ambulance
4	B	1	50,863.00
8	A	3	43,019.67
11	A	2	54,677.00
12	B	1	54,185.00
17	A	2	51,392.50
23	B	1	54,414.00
24	B	1	54,724.00
25	B	1	54,588.00
29	A	3	42,015.33
30	A	10	54,918.60
32	B	1	53,884.00
33	B	1	54,918.00
47	B	1	54,079.00
51	B	1	51,914.00
Fleet Composition

The optimized fleet consists of:

20 Type-A ambulances
9 Type-B ambulances
29 total ambulances
Station Allocation

The optimized ambulance allocation across selected station locations is shown below.

The result demonstrates that the optimization does not simply distribute ambulances equally across stations.

Instead, ambulance allocation varies according to the population assigned to each station and the constraints of the mathematical model.

For example, Site 30 receives 10 Type-A ambulances and covers a substantially larger set of demand areas than most other selected stations.

Population Coverage

Population per ambulance for the selected stations is shown below.

The values remain within the model's specified population-per-ambulance range of 35,000 to 55,000.

This provides a direct view of how the optimized ambulance allocation balances population coverage across the selected EMS stations.

First-Stage Coverage

The number of first-stage covered demand areas varies between the selected stations.

Examples from the optimized solution include:

Site 30 covers 18 demand areas in the first stage.
Site 47 covers 6 demand areas.
Sites 8, 11, and 29 each cover 4 demand areas.
Several Type-B stations cover smaller local groups of demand areas.

This highlights how the optimization combines station location and ambulance allocation rather than treating them as separate decisions.

Key Results

The optimization produced the following network configuration:

Metric	Result
Demand areas	65
Planning horizon	5 years
Selected stations	14
Type-A ambulances	20
Type-B ambulances	9
Total ambulances	29
Minimum population / ambulance	35,000
Maximum population / ambulance	55,000
Solver status	Optimal
Solver	CBC

The selected network provides first-stage coverage across the 65-area system while also incorporating predefined second-stage coverage relationships.

Why This Project Matters

This project demonstrates the use of data-driven mathematical optimization to solve a real operational planning problem.

Rather than only performing descriptive analysis, the model converts operational data and business constraints into actionable decisions:

Data
  ↓
Operational Constraints
  ↓
Mathematical Model
  ↓
Optimization
  ↓
Station Location Decisions
  ↓
Ambulance Allocation
  ↓
EMS Network Configuration

The project therefore demonstrates skills in:

Operations Research
Mathematical Optimization
Mixed-Integer Programming
Healthcare Analytics
Resource Allocation
Facility Location
Network Design
Python
PuLP
CBC Solver
Cost Modeling
NPV Analysis
Technologies
Python
PuLP
CBC Solver
NumPy
Mathematical Optimization
Mixed-Integer Programming
Project Structure
ambulance-station-location-optimization/
│
├── README.md
├── requirements.txt
├── .gitignore
│
├── src/
│   └── ambulance_station_location_optimization.py
│
└── results/
    ├── README.md
    │
    └── figures/
        ├── optimization-workflow.png
        ├── model-formulation.png
        ├── station-allocation.png
        ├── ambulance-type-distribution.png
        ├── population-coverage-per-ambulance.png
        └── coverage-by-station.png
How to Run
1. Clone the repository
git clone https://github.com/MobinaHaghshenas/ambulance-station-location-optimization.git
cd ambulance-station-location-optimization
2. Install dependencies
pip install -r requirements.txt
3. Run the optimization model
python src/ambulance_station_location_optimization.py

The model will create the optimization problem, solve it using CBC, and print the selected station configuration and ambulance allocation.

Reproducibility Note

The original project output reports 1.017 Type-B ambulances at Site 25.

However, the implementation defines the ambulance allocation variable as an integer variable. Therefore, the GitHub visualization represents the Site 25 allocation as 1 Type-B ambulance, consistent with the integer formulation of the model.

The optimization results and visualizations in this repository should therefore be interpreted according to the integer decision-variable formulation implemented in the Python model.

Limitations

This project is a mathematical optimization model and therefore depends on the assumptions and parameters defined in the model.

Important limitations include:

The analysis is based on predefined population and demand parameters.
Coverage relationships are explicitly encoded in the model.
Station-type eligibility is predefined.
Response-related cost parameters are modeled using the assumptions specified in the implementation.
The model focuses on location and allocation decisions rather than dynamic ambulance dispatching.
Real-world uncertainty in emergency demand and travel conditions is not explicitly simulated.

These limitations provide opportunities for extending the model with stochastic demand, dynamic dispatching, travel-time uncertainty, or simulation-based validation.

Potential Extensions

Possible extensions of this project include:

Stochastic EMS demand modeling
Time-dependent emergency demand
Travel-time-based coverage
Dynamic ambulance relocation
Multi-period optimization
Robust optimization under demand uncertainty
Integration with GIS data
Simulation-based validation
Multi-objective optimization of cost and response performance
References

The project was developed in the context of research on emergency medical service facility location, ambulance deployment, and optimization.

Selected references from the project report include:

Zhen, L., et al. (2015). Decision rules for ambulance scheduling decision support systems. Applied Soft Computing, 26, 350–356.
Coskun, N., & Erol, R. (2010). An optimization model for locating and sizing emergency medical service stations. Journal of Medical Systems, 34, 43–49.
Toregas, C., et al. (1971). The location of emergency service facilities. Operations Research, 19(6), 1363–1373.
Church, R., & ReVelle, C. (1974). The maximal covering location problem.
ReVelle, C. (1991). Siting ambulances and fire companies: New tools for planners. Journal of the American Planning Association, 57(4), 471–484.
Hatami-Marbini, A., et al. (2022). An emergency medical services system design using mathematical modeling and simulation-based optimization approaches. Decision Analytics Journal, 3, 100059.
Chen, Y., & Lai, Z. (2022). A multi-objective optimization approach for emergency medical service facilities location-allocation in rural areas. Risk Management and Healthcare Policy, 473–490.
Author

Mobina Haghshenas

M.Sc. in Industrial Engineering — Health Systems
Amirkabir University of Technology

Interested in:

Data Science
Machine Learning
Healthcare Analytics
Operations Research
Mathematical Optimization
AI for Healthcare
Data-driven Decision Making
