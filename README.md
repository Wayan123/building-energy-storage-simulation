# Building Energy Storage Simulation

<img src="docs/imgs/overview.drawio.png" alt="isolated" width="600"/>

The Building Energy Storage Simulation serves as open Source OpenAI gym (now [gymnasium](https://github.com/Farama-Foundation/Gymnasium)) environment for Reinforcement Learning. The environment represents a building with an energy storage (in form of a battery) and a solar energy system. The aim is to control the energy storage in such a way that the energy of the solar system can be used optimally. 

The inspiration of this project and the data profiles come from the [CityLearn](https://github.com/intelligent-environments-lab/CityLearn) environment. Anyhow, this project focuses on the ease of usage and the simplicity of its implementation.

## Installation

1. clone it to your local filesystem: `git clone https://github.com/tobirohrer/building-energy-storage-simulation.git`		
2. cd into the cloned repository: `cd python-project-template`
3. optional:

	4. create new python environment: `conda create -n <env_name> python=3.X`
	5. activate new python environment: `conda activate <env_name>`
6. install the package by: `pip install .` (If you want to run tests, build the sphinx documentation locally, or want to continue developing, install it via `pip install -e .[docs,tests]`)
7. done :)

## Usage 

```
from building_energy_storage_simulation import Environment
env = Environment()
env.reset()
env.step(42)
...
```

You can change the capacity of the battery by

```
env = Environment()
env.simulation.building.battery.capacity = 50
```

**Important note:** This environment is implemented by using [gymnasium](https://github.com/Farama-Foundation/Gymnasium) (the proceeder of OpenAI gym). Meaning, if you are using a reinforcement learning library like [Stable Baselines3](https://github.com/DLR-RM/stable-baselines3) make sure it supports [gymnasium](https://github.com/Farama-Foundation/Gymnasium) environments. 

## Task Description

The simulation contains a building with an energy consumption profile attached to it. The resulting electricity demand is always automatically covered by

- primarily using electricity generated by the solar energy system,
- and secondary by using electricity provided by the grid to cover the full demand.

The simulated building has a battery which can be used to store energy. You can think of using the battery as shifting the electricity demand. By storing energy, you are increasing the energy demand of the building and by discharging/obtaining energy from the battery you are decreasing the energy demand of the building. **The task is to choose when to charge and when to discharge the battery.** Hereby, the goal is to utilize the battery so energy usage from the solar energy system is maximized and usage of the energy grid is minimized. Note, that excess energy from the solar energy system which is not used by the electricity load or used to charge the battery is considered lost. So better use the solar energy to charge the battery in this case ;-)

 
### Action Space

| Action      | Min      | Max |
| ----------- | ----------- | ----------- |
| Charge | -1      | 1       |

The actions lie in the interval of [-1;1]. The action represents the portion of the battery is to be charged or discharged:

- 1 means fully charging the battery during the upcoming time step 
- -1 means fully discharging the battery, meaning "gaining" electricity out of the battery
- 0 means don't charge or discharge

### Observation Space

| Index | Observation      | Min | Max |
| ----------- | ----------- | ----------- | ----------- |
| 0 |  State of Charge (in kWh)| 0| Max Battery Capacity |
| [1; n]|  Forecast Electric Load (in kWh) | 0 | Max Load in Profile |
| [n+1; 2*n]|  Forecast Solar Generation (in kWh) |0| Max Solar Generation in Profile |


The length of the observation depends on the length of the forecast used. By default, the simulation uses a forecast length of 4. This means 4 time steps of an electric load forecast and 4 time steps of a solar generation forecast are included in the observation. In addition to that, the information about the current state of charge of the battery is contained in the observation.

The length of the forecast can be defined by setting the parameter `num_forecasting_steps` of the `Environment()`.


### Reward

As our goal is to use as less energy as possible, the reward is defined by the energy consumed at every time step (times -1): 

$$ r_t = -1 * electricity\_consumed_t $$ 

It is important to note, that the term `electricity_consumed` cannot be negative. This means, excess energy from the solar energy system which is not consumed by the electricity load or by charing the battery is considered lost (`electricity_consumed` is 0 in this case). 

### Episode Ends

The episode ends if the `max_timesteps` of the `Environment()` are reached.


### Parameters 

| Name | Description | Default Value | Set by |
| ----------- | ----------- | ----------- | ----------- |
| Battery Capacity |  The capacity of the battery (in kWh)| 100 | `Environment().simulation.building.battery.capacity = X` |
| Episode Length |  The length of one episode | 2000 | `Environment(max_timesteps)` |
| Forecast Length|  The length of the electric load and solar generation forecast |4| `Environment(num_forecasting_steps)` |
