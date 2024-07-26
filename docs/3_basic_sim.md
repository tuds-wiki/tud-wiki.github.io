# Basic Simulations

## SUMO Scenarios

Before anything can be simulated, all the necessary SUMO scenario files are required. This will typically include a '<i>.sumocfg</i>', '<i>.neteditcfg</i>', '<i>.net.xml</i>', '<i>.rou.xml</i>' and '<i>.add.xml</i>'. The simplest way to create a scenario and these files is using netedit, about which there is more information [here](https://sumo.dlr.de/docs/Netedit/index.html).

## Initialising the Simulation

All simulations in TUD-SUMO are created and run using the `Simulation` class, which is initialised as below. A scenario name and description are optional, but can be useful when running multiple similar simulations.

```python
from tud_sumo.simulation import Simulation

my_sim = Simulation(scenario_name="example", scenario_desc="Example simulation.")
```

To then start the simulation and create the connection to SUMO through TraCI, use `Simulation.start()`. This can very easily be done using the '<i>.sumocfg</i>' file created by netedit, which links all to all other files, however, the `net_file`, `route_file` and `add_file` parameters can also be used to give each file individually. Whether or not to use the GUI is also set here, although note this cannot be changed throughout the simulation.

```python
my_sim.start("example_scenario.sumocfg",
             gui=True,
             get_individual_vehicle_data=False,
             units="metric",
             seed=1
            )
```

The `get_individual_vehicle_data` is an important parameter denoting whether to collect and save dynamic information for all vehicles (ie. position, speed, acceleration etc.) at each step. This can be useful for small scenarios where this data may be required and computation time is less of an issue, such as single intersections, however, this should be set to false for large scenarios, such as large motorway networks.

3 unit settings are supported '<i>metric</i>' (km/kmph), '<i>imperial</i>' (mi/mph) and '<i>UK</i>' (km/mph). All data collected and saved are in these units, and this setting cannot be changed later.

The `seed` parameter is optional and affects both the SUMO simulation and the `Simulation` class. Random seeds in SUMO primarily affects how vehicles are added into the simulation (more information can be found [here](https://sumo.dlr.de/docs/Simulation/Randomness.html)). The seed can be set to random either by not using the parameter or by setting it to '<i>random</i>'.

Tracked junctions/edges, controllers etc. can be added at this point. These objects can be added individually, or if all parameters are saved in a dictionary or '<i>.json</i>' or '<i>.pkl</i>' file, these can be read using the `Simulation.load_objects()` function. More information on these objects can be found in their respective sections.

```python
my_sim.load_objects("parameters.json")
```

## Adding Demand

There are two approaches to adding demand to a simulation. The default approach is to define demand with routes or by flow within a '<i>.rou.xml</i>' file, in which case, nothing else needs to be done. Alternatively, demand can be generated more dynamically within TUD-SUMO. This is done with either the `Simulation.load_demand()` or `Simulation.add_demand()` functions.

!!! warning

    When adding demand using `Simulation.load_demand()` or `Simulation.add_demand()`, custom vehicle types and routes will still need to be pre-defined in a '<i>.rou.xml</i>' file. Any trips defined in this file will also still occur.

`Simulation.load_demand()` can be used to load a pre-defined demand profile from a '<i>.csv</i>' file, in the format below. For a route, either an '<i>origin</i>' and '<i>destination</i>' or a '<i>route_id</i>' is required. If using a route ID, the route must be pre-defined in the '<i>.rou.xml</i>' file. A time range for the demand is also required, either with a '<i>start_time/end_time</i>' or '<i>start_step/end_step</i>'. The demand value can either be given as a flow value in vehicles/hour under '<i>demand</i>' or as a raw number of vehicles under '<i>number</i>'.

If a flow value is given, vehicles are spawned throughout the demand period at this specified rate. Vehicles are inserted into the simulation using a Gaussian distribution with an average of '<i>demand</i>' vehicle per hour. '<i>insertion_sd</i>' is an optional float parameter that can be used to change the standard deviation of this distribution, and defaults to 1/3. Note that the actual standard deviation used is calculated using <i>demand * insertion_sd</i>. When the vehicles per step is below 1, vehicles are inserted at each step with this rate as a probability.

The other parameters are optional. '<i>vehicle_types</i>' can be a list of vehicle type IDs or a single ID and can optionally be given with '<i>vehicle_type_dists</i>'. When adding demand of multiple potential vehicle types, this allows for the distribution of types to be defined. If '<i>vehicle_types</i>', the default vehicle type is used, and when '<i>vehicle_types</i>' is given without a '<i>vehicle_type_dists</i>', vehicle types have an equal distribution. '<i>initial_speed</i>' defines the initial speed of vehicles at insertion and can either be '<i>max</i>', '<i>random</i>' or a number > 0, but defaults to '<i>max</i>'. '<i>origin_lane</i>' defines which lane vehicles are inserted at. This can either be '<i>random</i>', '<i>free</i>', '<i>allowed</i>', '<i>best</i>', '<i>first</i>' or a specific lane index, but defaults to '<i>best</i>'.

Two examples of the contents of a '<i>demand.csv</i>' file are shown below. It is possible to link a demand file in an object parameters dictionary under '<i>demand</i>' when calling `Simulation.load_objects()`.

| origin | destination | start_time | end_time | demand |    vehicle_types    | vehicle_type_dists |
|:------:|:-----------:|:----------:|:--------:|:------:|:-------------------:|:------------------:|
| edge_1 |   edge_10   |      0     |    600   |  1200  | "cars,vans,lorries" |    "0.7,0.2,0.1"   |
|   ...  |     ...     |     ...    |    ...   |   ...  |         ...         |         ...        |

| route_id | start_step | end_step | number | initial_speed | origin_lane | insertion_sd |
|:--------:|:----------:|:--------:|:------:|:-------------:|:-----------:|:------------:|
|  route_1 |      0     |   1200   |   200  |      max      |      1      |      0.3     |
|    ...   |     ...    |    ...   |   ...  |      ...      |     ...     |      ...     |

Otherwise, demand can be added in code using the `Simulation.add_demand()` function. This uses the same set of parameters as the demand files above, except '<i>origin/destination/route_id</i>' is replaced by a single `routing` parameter, and `step_range` is used instead of '<i>start_time/end_time</i>' or '<i>start_step/end_step</i>'. Demand is also defined as a flow rate in vehicles/hour. Examples are shown below.

```python
my_sim.add_demand(routing=("edge_1", "edge_10"),
                  step_range=(0, 1200),
                  demand=1200,
                  vehicle_types=["cars", "vans", "lorries"],
                  vehicle_type_dists=[0.7, 0.2, 0.2]
                 )

my_sim.add_demand(routing="route_1",
                  step_range=(0, 1200),
                  demand=200,
                  initial_speed="max",
                  origin_lane=1
                 )
```

!!! warning

    Adding demand dynamically (`Simulation.load_demand()` and `Simulation.add_demand()`) reduces performance compared to pre-defined demand in a '<i>.rou.xml</i>' file. Average trip times will also be longer as vehicles added this way are registered when they are loaded, not when they are inserted.

## Running the Simulation

The simulation is run using the `Simulation.step_through()` function. When no parameters are given, the simulation will run through one step by default. Otherwise, using `n_steps` will run the simulation for a specific number of steps, `end_step` will run the simulation until a specific step and `n_seconds` will run the simulation for a specific amount of time (in seconds).

```python
# Run 1 step
my_sim.step_through()

# Run for 100 steps
my_sim.step_through(100)

# Run until step 200
my_sim.step_through(end_step=200)

# Run for 100 seconds (where step length = 0.5)
my_sim.step_through(n_seconds=200)
```

A control loop can, therefore, be created as below. A progress bar is automatically created when simulating for 10 or more steps in one call of `Simulation.step_through()`. In order to create a progress bar that is consistent between separate calls, use the `pbar_max_steps` parameter and set this to the total length of the simulation.

```python
n, sim_dur = 100, 2500
while my_sim.curr_step < sim_dur:

    # Step through n steps.
    my_sim.step_through(n_steps=n, pbar_max_steps=sim_dur)

    # Perform control
    # ...
```

## Ending the Simulation

A simulation can be run until it is finished as below. The `Simulation.is_running()` function returns false once the simulation is over, which is determined as happening once all defined vehicles have finished their run through the simulation. It is then best to end the simulation using `Simulation.end()`, which closes the connection to TraCI.

```python
while my_sim.is_running():
    my_sim.step_through()

my_sim.end()
```

## Automatic Data Collection

One of the major advantages of TUD-SUMO is the automatic data collection. This involves automatically collecting all basic simulation information into a `sim_data` dictionary. An example of this can be seen in examples directory in the main TUD-SUMO repository, however, the main structure is as follows:

```JSON
{
    "data":
        {
            "detectors": {},
            "junctions": {},
            "edges": {},
            "controllers": {},
            "vehicles": {},
            "demand": {},
            "trips": {},
            "events": {},
            "all_vehicles": {}
        },
    "start": 0,
    "end": 1000,
    "step_len": 1.0,
    "units": "METRIC",
    "seed": 10,
    "sim_start": "08/07/2024, 13:00:00",
    "sim_end": "08/07/2024, 13:00:10"
}
```

Detectors will automatically collect vehicle speeds and counts for each time step, as well as the IDs of each vehicle that passed over it, whilst occupancies are also collected for mutli-entry-exit detectors. These will be stored under '<i>data/detectors/{detector_id}</i>', with '<i>type</i>', '<i>position</i>', '<i>speeds</i>', '<i>vehicle_counts</i>', '<i>vehicle_ids</i>' and '<i>occupancies</i>'.

Vehicle data will include the number of vehicles at each step, '<i>tts</i>' (N. vehicles * step length) and '<i>delay</i>' (calculated as the time spent by vehicles waiting in each time step).

Demand is only included when dynamically adding demand, ie. not when demand is solely defined in the '<i>.rou.xml</i>' file. This dictionary will contain two objects; '<i>headers</i>' and a '<i>table</i>'. '<i>table</i>' will contain a (6 x n) sized array, with its headers listed under '<i>headers</i>'. Only demand added through `Simulation.load_demand()` and `Simulation.add_demand()` will be included here.

Trip data will contain data for incomplete and completed trips. This is stored under '<i>data/trips</i>' and then either '<i>incomplete</i>' or '<i>completed</i>'. Each trip will store the '<i>route_id</i>', '<i>vehicle_type</i>', '<i>departure</i>', '<i>arrival</i>' (or removal), '<i>origin</i>' and '<i>destination</i>'.

Junction, edge, controller and event data are only included when necessary. The `all_vehicles` data will contain all the individual vehicle data at each time step, so it is only included when `get_individual_vehicle_data` is set to true when starting the simulation.

The `sim_data` dictionary can be reset at any point using the `Simulation.reset_data()` function.

```python
my_sim.reset_data()
```

## Saving & Summarising Data

All the data collected throughout the simulation can be saved at any point using the `Simulation.save_data()` function. This will save the `sim_data` dictionary as a file in the specified directory. Either JSON or pickle files are supported, simply denoted by a '<i>.json</i>' or '<i>.pkl</i>' extension in the filename.

```python
my_sim.save_data("data/example_data.json")
my_sim.save_data("data/example_data.pkl")
```

All data saved by a simulation or a simulation data file can be summarised using the `Simulation.print_summary()` or `print_summary()` functions. This will print a summary of the collected data (ie. number of vehicles, TTS, controllers, events etc.), as well as some information about the simulation itself (ie. scenario name/description, runtime, seed etc.). This summary can be saved to a '<i>.txt</i>' file using the `save_file` parameter. An example summary is shown below.

```python
my_sim.print_summary(save_file="data/example_summary.txt")

# Print summary without creating a Simulation object
from tud_sumo.simulation import print_summary
print_summary("data/example_data.pkl")
```

```
 *============================================================*
 |                          A20_ITCS                          | 
 *============================================================*
 |                        Description:                        | 
 |   Example traffic controllers, with 2 ramp meters, 1 VSL   | 
 |        controller and 1 route guidance controller.         | 
 *============================================================*
 |                 Simulation Run: 10/07/2024                 | 
 |               12:02:14 - 12:02:25 (0:00:11)                | 
 *------------------------------------------------------------*
 | Number of Steps:                                       500 | 
 | Step Length:                                           1.0 | 
 | Avg. Step Duration:                                 0.022s | 
 | Units Type:                              Metric (km, km/h) | 
 | Seed:                                                    1 | 
 *============================================================*
 |                            Data                            | 
 *============================================================*
 |                        Vehicle Data                        | 
 *------------------------------------------------------------*
 | Avg. No. Vehicles:                                  774.66 | 
 | Peak No. Vehicles:                                    1277 | 
 | Avg. No. Waiting Vehicles:                           34.26 | 
 | Peak No. Waiting Vehicles:                             104 | 
 | Final No. Vehicles:                                   1277 | 
 | Individual Data:                                        No | 
 * ---------------------------------------------------------- *
 | Total TTS:                                       387332.0s | 
 | Total Delay:                                      17132.0s | 
 *------------------------------------------------------------*
 |                         Trip Data                          | 
 *------------------------------------------------------------*
 | Incomplete Trips:                             1281 (67.2%) | 
 | Completed Trips:                               625 (32.8%) | 
 *------------------------------------------------------------*
 |                         Detectors                          | 
 *------------------------------------------------------------*
 |               Induction Loop Detectors: (15)               | 
 |       a13_ramp_inflow, cw_down_occ_0, cw_down_occ_1,       | 
 |  cw_down_occ_2, cw_down_occ_3, cw_up_occ_0, cw_up_occ_1,   | 
 |      cw_up_occ_2, rerouter_2, utsc_e_in, utsc_e_out,       | 
 |      utsc_n_in_1, utsc_n_in_2, utsc_w_in, utsc_w_out       | 
 |                                                            | 
 |              Multi-Entry-Exit Detectors: (7)               | 
 |    a13_ramp_queue, a13_rm_downstream, a13_rm_upstream,     | 
 |      cw_ramp_inflow, cw_ramp_queue, cw_rm_downstream,      | 
 |                       cw_rm_upstream                       | 
 *------------------------------------------------------------*
 |                       Tracked Edges                        | 
 *------------------------------------------------------------*
 | 126730026, 1191885773, 1191885771, 126730171, 1191885772,  | 
 |         948542172, 70944365, 308977078, 1192621075         | 
 *------------------------------------------------------------*
 |                     Tracked Junctions                      | 
 *------------------------------------------------------------*
 |                     utsc (Signalised)                      | 
 |           crooswijk_meter (Signalised, Metered)            | 
 |              a13_meter (Signalised, Metered)               | 
 *------------------------------------------------------------*
 |                        Controllers                         | 
 *------------------------------------------------------------*
 |                    Route Guidance: (1)                     | 
 |                          rerouter                          | 
 |                                                            | 
 |                 Variable Speed Limits: (1)                 | 
 |                            vsl                             | 
 *------------------------------------------------------------*
 |                    Event IDs & Statuses                    | 
 *------------------------------------------------------------*
 |                 Active: incident_response                  | 
 |             Completed: incident_1, bottleneck              | 
 *------------------------------------------------------------*
```

The structure of a simulation data file can also be printed using the `Simulation.print_sim_data_struct()` or `print_sim_data_struct()` functions. This will print the structure of the `sim_data` dictionary as a tree, allowing you to see the exact keys and data types used in the data. An example of the output of `print_sim_data_struct()` is shown below.

```python
my_sim.print_sim_data_struct()

# Print structure without creating a Simulation object
from tud_sumo.simulation import print_sim_data_struct
print_sim_data_struct("data/example_data.pkl")
```

```
A20_ITCS:
  ├─- scenario_name: str
  ├─- scenario_desc: str
  ├─- data:
  |     ├─- detectors:
  |     |     ├─- a13_ramp_queue:
  |     |     |     ├─- type: str
  |     |     |     ├─- position:
  |     |     |     |     ├─- entry_lanes: list (1x1)
  |     |     |     |     ├─- exit_lanes: list (1x1)
  |     |     |     |     ├─- entry_positions: list (1x1)
  |     |     |     |     └─- exit_positions: list (1x1)
  |     |     |     ├─- speeds: list (1x500)
  |     |     |     ├─- vehicle_counts: list (1x500)
  |     |     |     ├─- vehicle_ids: list (500x27*)
  |     |     |     └─- occupancies: list (1x0)
  .     .     .
  .     .     .
  .     .     .
  |     ├─- vehicles:
  |     |     ├─- no_vehicles: list (1x500)
  |     |     ├─- no_waiting: list (1x500)
  |     |     ├─- tts: list (1x500)
  |     |     └─- delay: list (1x500)
  |     ├─- trips:
  |     |     ├─- incomplete: dict (1x1311)
  |     |     └─- completed: dict (1x601)
  |     └─- events:
  |           ├─- scheduled: list (1x1)
  |           ├─- active: list (1x1)
  |           └─- completed: list (1x1)
  ├─- start: int
  ├─- end: int
  ├─- step_len: float
  ├─- units: str
  ├─- seed: int
  ├─- sim_start: str
  └─- sim_end: str
```