---
layout: default
title: Case Study
parent: ModelicaOrch
nav_order: 2
---

## Case Study: Self-adaptive Edge Computing System

### Objective

To develop a self-sufficient edge computing system powered by PV, capable of dynamically adjusting its **CPU cores** and **frequency** based on **available energy** and **user demand**. 

The system aims to meet **user demand** while **remaining energy** ≥ 0 for each time unit.

### Input

* The energy available (time unit: hour)

* User demand (time unit: hour)

### Goals:

  - **Meet User Demand:** Ensure the system provides the required **performance** (computing resource the system can provide) to meet **user demand** (computing resource the user needs). 
    - User demand is satisfied when **performance** ≥ **user demand**.

  - **Maximize Energy Efficiency:** Optimize the use of energy consumption to prolong system operation and efficiency.
    - **remaining energy** should be ≥ 0 at the end of the simulation

### Outcome 

A dynamically self-adaptive edge computing system that efficiently manages its resources based on **available energy** and **user demand** for each time unit (hour), ensuring sustainable and efficient operation for autonomous vehicles requiring edge computing capabilities.

### File Structure

```shell
/src
  |-- MOO4Modelica
  	|-- config.json
  	|-- config.py
  	|-- optimize_main.py
  	|-- parallel_computing.py
  	|-- optimization_libraries.py
  |-- OrchestrationWorkflow
      |-- ITSystem.mo  # Modelica model
      |-- energy_available_and_user_demand_data.txt  # input data
      |-- orchestration_config.json  # config file of the orchestration workflow
      |-- orchestrator.py
      |-- orchestration_wrapper.py
      |-- orchestration_configurator.py
```

### Steps

**Step 1:** Set up the configuration in `orchestration_config.json`:

```shell
{
    "DATA_FILE_PATH": "data/energy_available_and_user_demand.txt",
    "CONFIG_PATH": "config.json",
    "MODEL_FILE": "ITSystem.mo",
    "MODEL_NAME": "ITSystem",
    "SIMULATION_TIME": 100,
    "TIME_CONFIG": {
        "START_TIME": 8,
        "END_TIME": 12,
        "TIME_UNIT": "hour"
    },
    "OBJECTIVES": [
        {"name": "remainingEnergy", "maximize": true},
        {"name": "performance", "maximize": true}  # computing power the system provides 
    ],
    "TUNABLE_PARAMETERS": {
        "PARAMETERS": ["activeCores", "cpuFrequency"],
        "PARAM_BOUNDS": {
            "activeCores": {
                "bounds": [1, 4],
                "type": "int"
            },
            "cpuFrequency": {
                "bounds": [1.0, 3.0],
                "type": "float"
            }
        }
    },
    "INPUT_PARAMETERS": {
        "available_energy": "availableEnergy",
        "user_demand": "userDemand"  # computing power the user needs
    },
    "CRITERIA": {
        "GOAL_EXPRESSION": [
            "evaluation_results['performance'] >= simulation_inputs['user_demand']",
            "evaluation_results['remainingEnergy'] >= 0"
        ]
    },
    "OPTIMIZATION_CONFIG": {
        "USE_SINGLE_OBJECTIVE": false,
        "ALGORITHM_NAME": "nsga2",
        "POP_SIZE": 5,
        "N_GEN": 2
    },
    "PLOT_CONFIG": {
        "PLOT_X": "",
        "PLOT_Y": "",
        "PLOT_TITLE": "",
        "ENABLE_PLOT": false
    },
    "LIBRARY_CONFIG": {
        "LOAD_LIBRARIES": false,
        "LIBRARIES": [
            {"name": "", "path": ""}
        ]
    },
    "N_JOBS": -1
}
```

**Step 2:** run `python orchestrator.py`:

Finally, you will see the final report:

```shell
Processing hour: 8  # 8 am
Parameters set: {'activeCores': 4, 'cpuFrequency': 3.00}
Performance: 930  # at 8 am user demand is high, the evaluated performance is 930 which satisfies user demand
User demand satisfied.

...

Processing hour: 12  # 12 am
Parameters set: {'activeCores': 2, 'cpuFrequency': 1.80}
Performance: 372  # at 12 am user demand is medium, the evaluated performance is 372 which satisfies user demand
User demand satisfied.

Final Report:
Hour 8: User demand satisfied with configuration {'activeCores': 4, 'cpuFrequency': 2.5}.

...

Hour 12: User demand satisfied with configuration {'activeCores': 2, 'cpuFrequency': 1.8}.
```

You can also visualize the final result:

<img src="../../assets/ModelicaOrch_result.png" alt="result" style="zoom:100%;" />

At 8 AM,  both goals are not satisfied. 

At 9 AM, the first goal is not satisfied. 

From 10 AM to 12 AM, both goals are satisfied. 