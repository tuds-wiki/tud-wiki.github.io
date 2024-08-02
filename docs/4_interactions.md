# Interacting with the Simulation

## Testing Values

Several functions exist to check values in the simulation, primarily to check wether objects exist and get their type or status. All of these are within the `Simulation` class, and are:

### Vehicles
| Function                  | Return Value                                                                            |
|---------------------------|-----------------------------------------------------------------------------------------|
| `vehicle_exists(ID)`      | Whether or not a vehicle with the given ID exists in the simulation.                    |
| `vehicle_loaded(ID)`      | Whether or not a vehicle has been scheduled to enter (or is already in) the simulation. |
| `vehicle_to_depart(ID)`   | Whether or not a vehicle has been scheduled to enter (but is not in) the simulation.    |
| `vehicle_type_exists(ID)` | Whether or not a vehicle type with the given ID exists.                                 |
| `route_exists(ID)`        | Route edges if the ID exists, else returns None.                                        |

### Other Objects
| Function                      | Return Value                                               |
|-------------------------------|------------------------------------------------------------|
| `detector_exists(ID)`         | Detector type if the ID exists, else returns None.         |
| `controller_exists(ID)`       | Whether a controller with the ID exists.                   |
| `event_exists(ID)`            | Event status if the ID exists, else returns None.          |
| `geometry_exists(ID)`         | Geometry type if ID exists (lane/edge), else returns None. |
| `tracked_edge_exists(ID)`     | Whether a tracked edge with the ID exists.                 |
| `junction_exists(ID)`         | Whether a junction with the ID exists.                     |
| `tracked_junction_exists(ID)` | Whether a tracked junction with the ID exists.             |

## Getting Values

### Object IDs

The most basic getter functions in the `Simulation` class return the IDs of objects within the simulation. These are:

|                Function                |                                      Return Value                                                           |
|----------------------------------------|-------------------------------------------------------------------------------------------------------------|
| `get_vehicle_ids(vehicle_types)`       | All vehicle IDs, or those of specific type(s).                                                              |
| `get_detector_ids(detector_types)`     | All detector IDs, or those of specific type(s) - '<i>multientryexit</i>' or '<i>inductionloop</i>'.         |
| `get_controller_ids(controller_types)` | All controller IDs, or those of specific type(s) - '<i>VSLController</i>' or '<i>RGController</i>'.         |
| `get_event_ids(event_statuses)`        | All event IDs, or those of specific status(es) - '<i>scheduled</i>', '<i>active</i>' or '<i>completed</i>'. |
| `get_geometry_ids(geometry_types)`     | All controller IDs, or those of specific type(s) - '<i>edge</i>' or '<i>lane</i>'.                          |
| `get_tracked_edge_ids()`               | All tracked edge IDs.                                                                                       |
| `get_junction_ids()`                   | All junction IDs.                                                                                           |
| `get_tracked_junction_ids()`           | All tracked junction IDs.                                                                                   |

### Other Data

All data collected throughout the simulation is stored in the `sim_data` dictionary, however, its data (and other dynamic vehicle data) can be fetched using the `Simulation.get_[x]()` functions below.

  - `get_no_vehicles()`:
    - Returns the number of vehicles in the last step of the simulation.
  - `get_tts()`:
    - Returns the Total Time Spent (TTS) by vehicles in the simulation during the last step.
  - `get_delay()`:
    - Returns total vehicle delay during the last simulation step (calculated as the number of vehicles where speed < 0.1m/s<sup>2</sup>, multiplied by the simulation step length).
  - `get_vehicle_data(vehicle_ids)`:
    - Returns a dictionary containing all information on a vehicle. This is; '<i>type</i>', '<i>longitude</i>', '<i>latitude</i>', '<i>altitude</i>', '<i>heading</i>', '<i>speed</i>', '<i>acceleration</i>', '<i>stopped</i>', '<i>length</i>', '<i>departure</i>', '<i>destination</i>' and '<i>origin</i>'.
    - TUD-SUMO stores static information (route origin, vehicle length etc.) to avoid unnecessary repeated calls to TraCI. This is automatically done when calling `get_vehicle_data()` on a vehicle for the first time.
    - `vehicle_ids` can be a single vehicle ID (string), or a list of IDs (list/tuple). If multiple IDs are given, a dictionary is returned with each vehicle's data stored under its ID.
  - `get_all_vehicle_data(vehicle_types, all_dynamic_data):`
    - Returns the total number of vehicles in the simulation, the total number of waiting vehicles, and if `all_dynamic_data == True`, the static & dynamic data for all vehicles.
    - If `all_dynamic_data == False`, an empty dictionary is returned as the last variable.
  - `get_last_step_detector_vehicles(detector_ids, vehicle_types, flatten)`:
    - Returns the IDs of all vehicles who passed over the specified detector(s) in the last simulation step.
    - `detector_ids` and `vehicle_types` can either be a single value (string) or list of values (list/tuple). If multiple detector IDs are given, IDs for all detectors can either be returned in a single list (`flatten = True`), or the IDs can be returned in a dictionary with lists of IDs separated by detector (`flatten = False`).
  - `get_last_step_geometry_vehicles(geometry_ids, vehicle_types, flatten)`:
    - Returns the IDs of all vehicles on the specified geometry in the last simulation step.
    - `geometry_ids` and `vehicle_types` can either be a single value (string) or list of values (list/tuple). If multiple geometry IDs are given, IDs for all geometries can either be returned in a single list (`flatten = True`), or the IDs can be returned in a dictionary with lists of IDs separated by geometry (`flatten = False`).
  - `get_interval_detector_data(detector_id, n_steps, data_keys, avg_vals)`:
    - Returns data collected by a detector between during the time range (`curr_step - n_steps`, `curr_step`).
    - `data_keys` can either be a single value (string) or a list of values (list/tuple). The valid keys are '<i>vehicle_counts</i>', '<i>speeds</i>' and '<i>occupancies</i>', although '<i>occupancies</i>' is only valid for induction loop detectors.
    - If `avg_vals == True`, then values are returned averaged, otherwise, raw values are returned.

To query routes and paths in the network, use the functions below.

  - `is_valid_path(edge_ids)`:
    - Tests whether a list of edges is a valid path. This is determined by checking whether each edge connects to the subsequent edge in the list.
    - Valid paths can be added as a route using the `Simulation.add_route()` function.
  - `get_path_travel_time(edge_ids, curr_tt, unit)`:
    - Calculate the travel time of a path (list of edges). If `curr_tt` is true, the travel time is calculated using current mean speed on edges. If `curr_tt` is false, the travel time is calculated using free-flow speed.
  - `get_path_edges(origin, destination, curr_optimal)`:
    - Uses the A* algorithm to find the optimal route between two edges (`origin` to `destination`).
    - If `curr_optimal` is true, the route is based on current conditions, using current mean speed on edges to find travel time. If `curr_optimal` is false, the route is calculated based on free-flow travel time.

### Advanced Getters

Vehicles, detectors and geometries all use a `Simulation.get_[x]_vals()` function, which allow you to very easily get a set of values for multiple objects. All 3 use the same structure, with the same type of parameters:

  1. `vehicle_ids`/`detector_ids`/`geometry_ids`: A single ID (string) or list of IDs (list/tuple)
  2. `data_keys`: A single data key (string) or list of data keys (list/tuple)

If one data key is given, the raw value is returned, otherwise, a dictionary is returned containing the values separated by data key. Similarly, if one object ID is given, the raw data value/dictionary is returned, otherwise, a dictionary is returned containing the data separated by vehicle ID. For example:

```python
>>> my_sim.get_vehicle_vals("vehicle_1", "speed")
25

>>>  my_sim.get_vehicle_vals("vehicle_1", ("speed", "acceleration"))
{"speed": 25, "acceleration": -1}

>>>  my_sim.get_vehicle_vals(("vehicle_1", "vehicle_2"), "speed")
{"vehicle_1": 25, "vehicle_2": 30}

>>>  my_sim.get_vehicle_vals(("vehicle_1", "vehicle_2"), ("speed", "acceleration"))
{"vehicle_1": {"speed": 25, "acceleration": -1}, "vehicle_2": {"speed": 30, "acceleration": 2}}
```

Subscriptions and static vehicle data are used whenever possible to reduce TraCI calls. The valid data keys for each function are listed below.

  - `get_vehicle_vals()`:
    - '<i>type</i>': Vehicle type
    - '<i>length</i>': Vehicle length
    - '<i>speed</i>': Current vehicle speed
    - '<i>is_stopped</i>': Bool denoting whether the vehicle is stopped
    - '<i>max_speed</i>': Vehicle maximum speed
    - '<i>acceleration</i>': Current vehicle acceleration
    - '<i>position</i>': Current vehicle coordinates
    - '<i>altitude</i>': Current vehicle altitude
    - '<i>heading</i>': Current vehicle heading
    - '<i>departure</i>': Vehicle departure time
    - '<i>edge_id</i>': Current vehicle's edge ID
    - '<i>lane_idx</i>': Index of the vehicle's current lane
    - '<i>origin</i>': Departure edge ID of the vehicle
    - '<i>destination</i>': Current destination edge ID of the vehicle
    - '<i>route_id</i>': Current vehicle route ID
    - '<i>route_idx</i>': The index of the vehicle's edge on its route
    - '<i>route_edges</i>': The list of edges that the vehicle's route consists of
  - `get_detector_vals()`:
    - '<i>type</i>': Detector type ('mutlientryexit' or 'inductionloop')
    - '<i>position</i>': Detector coordinates
    - '<i>vehicle_count</i>': Number of vehicles that passed over the detector in the last step
    - '<i>vehicle_ids</i>': IDs of vehicles that passed over the detector in the last step
    - '<i>lsm_speed</i>': Average speed of vehicles that passed over the detector in the last step
    - '<i>halting_no</i>': Number of halting vehicles in the detector area <i>(multi-entry-exit only)</i>
    - '<i>lsm_occupancy</i>': Average occupancy during the last step <i>(induction loop only)</i>
    - '<i>last_detection</i>': Time since last detection <i>(induction loop only)</i>
  - `get_geometry_vals()`:
    - '<i>vehicle_count</i>': Number of vehicles on the edge/lane
    - '<i>vehicle_ids</i>': IDs of vehicles on the edge/lane
    - '<i>vehicle_speed</i>': Average speed of vehicles on the edge/lane
    - '<i>halting_no</i>': Number of halting vehicles on the edge/lane
    - '<i>vehicle_occupancy</i>': Vehicle occupancy of edge/lane
    - '<i>curr_travel_time</i>': Estimated travel time (calculated using length and average speed)
    - '<i>ff_travel_time</i>': Estimated free-flow travel time (calculated using length and maximum speed)
    - '<i>emissions</i>': CO, CO<sub>2</sub>, HC, PMx and NOx emissions, stored in a dictionary
    - '<i>length</i>': Length of the edge/lane
    - '<i>max_speed</i>': Maxmimum speed (edge max_speed is the average of its lanes)
    - '<i>connected_edges</i>': Dictionary containing 'incoming' and 'outgoing' edges <i>(edge only)</i>
    - '<i>incoming_edges</i>': Incoming edges <i>(edge only)</i>
    - '<i>outgoing_edges</i>': Outgoing edges <i>(edge only)</i>
    - '<i>street_name</i>': Street name, defined in SUMO <i>(edge only)</i>
    - '<i>n_lanes</i>': Number of lanes in the edge <i>(edge only)</i>
    - '<i>lane_ids</i>': IDs of all lanes in the egde <i>(edge only)</i>
    - '<i>edge_id</i>': Edge ID for the lane <i>(lane only)</i>
    - '<i>n_links</i>': Number of linked lanes <i>(lane only)</i>
    - '<i>allowed</i>': List containing allowed vehicle types <i>(lane only)</i>
    - '<i>disallowed</i>': List containing all prohibited vehicle types <i>(lane only)</i>
    - '<i>left_lc</i>': Bool denoting whether left lane changes are allowed <i>(lane only)</i>
    - '<i>right_lc</i>': Bool denoting whether right lane changes are allowed <i>(lane only)</i>

## Setting Values

Both vehicle and geometries (edges/lanes) also allow for dynamically setting variables with the `Simulation.set_vehicle_vals()` and `Simulation.set_geometry_vals()` functions. Both use the same types of parameters:

  1. `vehicle_ids`/`geometry_ids`: A single ID (string) or list of IDs (list/tuple). If multiple IDs are given, the values are set for all objects.
  2. `data_keys`: Unlike the `get_[x]_vals()` functions, data_keys are used as arguments. There is no limit on the number of data keys given.

For example:

```python
>>> my_sim.set_geometry_vals("edge_1", max_speed=40)
>>> my_sim.set_vehicle_vals(("vehicle_1", "vehicle_2"), max_speed=20, highlight="#FF0000")
```

The valid data keys and their accepted data type are listed below. Note that the keys aim to be the same as those in the `get_[x]_vals()` functions (whenever possible).

  - `set_vehicle_vals()`:
    - `colour`: Changes the vehicle's colour (either valid hex code or list of rgb(a) values)
    - `highlight`: Sets vehicle highlighting (boolean)
    - `speed`: Sets a new speed value for the vehicle (integer or float)
    - `max_speed`: Sets a new maximum speed for the vehicle (integer or float)
    - `acceleration`: Sets a new acceleration for a given duration (list/tuple containing speed value and duration)
    - `lane_idx`: Sets a vehicle to try and change lane index for a given duration (list/tuple containing lane index and duration)
    - `destination`: Sets a new destination by edge ID (string)
    - `route_id`: Sets the vehicle to another route by its ID (string)
    - `route_edges`: Sets the vehicle to a new route by edge IDs (list of strings)
    - `speed_safety_checks`: Indefinitely sets whether speed/acceleration safety constraints are followed when setting speed - can be boolean to turn on/off all checks or [bitset](https://sumo.dlr.de/docs/TraCI/Change_Vehicle_State.html#lane_change_mode_0xb6) (boolean/int)
    - `lc_safety_checks`: Indefinitely sets whether lane changing safety constraints are followed when changing lane - can be boolean to turn on/off all checks or [bitset](https://sumo.dlr.de/docs/TraCI/Change_Vehicle_State.html#speed_mode_0xb3) (boolean/int)
  - `set_geometry_vals()`:
    - `max_speed`: Set a new maximum speed/speed limit (lane or for all contained lanes) (integer or float)
    - `allowed`: Set a new list of allowed vehicle types, with an empty list allowing all (list of strings) <i>(lane only)</i>
    - `disallowed`: Set a new list of prohibited vehicle types (list of strings) <i>(lane only)</i>
    - `left_lc`: Sets the list of vehicle types that are allowed to change to the left lane (list of strings) <i>(lane only)</i>
    - `right_lc`: Sets the list of vehicle types that are allowed to change to the right lane (list of strings) <i>(lane only)</i>

## Subscriptions

Subscriptions are a useful tool in TraCI that aim to reduce runtime by limiting the amount of API calls. This can be particularly effective when data needs to be collected at each simulation step. TUD-SUMO automatically adds vehicle subscriptions for basic variables including position, speed and acceleration as these are already used within the automatic data collection and are necessary for calculating vehicle delay at each step. Tracked edges will also automatically subcribe to vehicle IDs, as this is needed to collect step vehicle data.

It is possible to disable automatic vehicle subscriptions by setting `automatic_subscriptions = False` in `Simulation.start()`. However, this is most likely unncessary and not recommened, particularly for applications where a lot of data needs to be repeatedly collected from the simulation.

Otherwise, depending on the use case, it may be necessary to subscribe/unsubscribe to other variables. This can be done using the functions below. These use the same structure as the `get_[x]_vals()` functions with two parameters:

  1. `vehicle_ids`/`detector_ids`/`geometry_ids`: A single ID (string) or list of IDs (list/tuple). Note that individual objects have their own set of subscriptions.
  2. `data_key`: A single data key (string) or list of data keys (list/tuple).

The valid data keys for each object type and respective function are listed below. Note that individual objects have their own set of subscriptions, so if you would like to subscribe to collect all vehicles' edge ID at each step, you must call `add_vehicle_subscriptions()` for all vehicle IDs.

  - `add_vehicle_subscriptions()`/`remove_vehicle_subscriptions()`:
    - '<i>speed</i>'
    - '<i>is_stopped</i>'
    - '<i>max_speed</i>'
    - '<i>acceleration</i>'
    - '<i>position</i>'
    - '<i>altitude</i>'
    - '<i>heading</i>'
    - '<i>edge_id</i>'
    - '<i>lane_idx</i>'
    - '<i>route_id</i>'
    - '<i>route_idx</i>'

  - `add_detector_subscriptions()`/`remove_detector_subscriptions()`:
    - '<i>vehicle_count</i>'
    - '<i>vehicle_ids</i>'
    - '<i>lsm_speed</i>'
    - '<i>halting_no</i>'
    - '<i>lsm_speed</i>'
    - '<i>last_detection</i>'

  - `add_geometry_subscriptions()`/`remove_geometry_subscriptions()`:
    - '<i>vehicle_count</i>'
    - '<i>vehicle_ids</i>'
    - '<i>vehicle_speed</i>'
    - '<i>halting_no</i>'
    - '<i>occupancy</i>'

## Adding/removing Vehicles

Vehicles can be added to or removed from the simulation using the `Simulation.add_vehicle()` and `Simulation.remove_vehicles()` functions respectively. These functions and their parameters are detailed below.

  - `add_vehicle()`:
    - `vehicle_id`: (Unique) ID for the new vehicle.
    - `vehicle_type`: Vehicle type.
    - `routing`: Denotes how the vehicle will route through the network (either route ID or (2x1) list of edge IDs for an OD pair).
    - `initial_speed`: Initial speed of the vehicle at insertion, defaults to maximum (either '<i>max</i>', '<i>random</i>' or a number > 0).
    - `origin_lane`: Lane for insertion at origin, defaults to best (either '<i>random</i>', '<i>free</i>', '<i>allowed</i>', '<i>best</i>', '<i>first</i>' or lane index).
  - `remove_vehicles()`:
    - `vehicle_ids`: A single vehicle ID (string) or list of IDs (list/tuple).

When repeatedly adding new vehicles, it may be useful to create a new route that vehicles can be assigned to. This can be done using `Simulation.add_route()`, which is detailed below.

  - `add_route()`:
    - `routing`: List of edge IDs. If 2 disconnected edges are given, vehicles calculate an optimal path at insertion.
    - `route_id`: (Unique) route ID. If this is not given, the ID is generated using the origin and destination edge IDs.
    - `assert_new_id`: If true, an error is thrown for duplicate route IDs.

## Custom Vehicle Functions

`Simulation.add_vehicle_in_funcs()` and `Simulation.add_vehicle_out_funcs()` can be used to easily add custom functions that are called with each vehicle that enters or leaves the simulation. An example of this is shown below. Multiple functions can be added and are called in the order in which they are added.

```python
def new_vehicle_1(vehicle_id, origin):
    print(vehicle_id, "entered simulation from edge", origin)

def new_vehicle_2(curr_step):
    print("new vehicle at step", curr_step, "\n")

def exiting_vehicle(vehicle_id, curr_step):
    print(vehicle_id, "left simulation at step", curr_step)

my_sim.add_vehicle_in_funcs([new_vehicle_1, new_vehicle_2])
my_sim.add_vehicle_out_funcs(exiting_vehicle)

while my_sim.is_running():
    my_sim.step_through()
```

Running this example would therefore produce:

```
>>> run_sim.py
car_1 entered simulation from edge west_1
new vehicle at step 0

car_2 entered simulation from edge west_1
new vehicle at step 0

car_1 left simulation at step 10
car_2 left simulation at step 10
```

Only specific parameters (`curr_step`, `vehicle_id`, `route_id`, `vehicle_type`, `departure`, `origin`, `destination`) can be used, although `add_vehicle_out_funcs()` only takes functions that use `curr_step` and/or `vehicle_id`.

These functions can be removed using `Simulation.remove_vehicle_in_funcs()` and `Simulation.remove_vehicle_out_funcs()`.