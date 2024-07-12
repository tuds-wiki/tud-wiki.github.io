# Tracked Objects

Tracked objects are a useful way to include extra objects in the automatic data collection. Whilst all data from every detector is automatically included, specific scenarios may contain a large number of edges or junctions. Therefore, important edges or junctions can be tracked to automatically collect more specific information, such as inflow or average speeds. Tracked objects are also supported within the [plotter](8_plotting.md) class, so that phase signal diagrams or space-time diagrams can easily be plotted with significantly less effort.

Currently, only edges and junctions can be tracked, although this may change in the future.

## Tracked Junctions

There are two types of tracked junctions; regular junctions and metered junctions. Most intersections will only need to be tracked as a regular junction, whilst metered junctions are a subclass of junction that include some extra tracking for motorway on-ramp metering.

!!! tip
    Note that traffic light/ramp meter control itself is not discussed here. <b>An adaptive traffic signal or ramp meter does not have to be tracked</b> and not all tracked junctions require some control. Tracking a junction with an adaptive traffic signal or ramp meter simply adds a level of data collection, however, <b>it is recommended</b> and makes evaluation of the controllers much easier.

All parameters to create a tracked junction are given as a dictionary and is done using the `Simulation.add_tracked_junctions()` function. This can be in code, or from a '<i>.json</i>' or '<i>.pkl</i>' file. Tracked junctions can also be included in the object parameters file when calling `Simulation.load_objects()`, under '<i>junctions</i>'. A tracked junction can be initialised at 3 different levels, depending on the amount of data collection required.

  1. Signal tracking only:
    - This is the lowest level of tracking, where only signal phases and average green/red times are tracked. Therefore, this level is only useful when the specified junction has a traffic light.
    - In this case, the only required parameters are the junction ID or a list of junction IDs. It is recommended to give junctions and their traffic lights the same ID in SUMO in order to avoid confusion.

  2. Inflow/outflow tracking:
    - Traffic-flow related variables can be tracked by tracked junctions, however, it is necessary to define the detectors used to do so.
    - Here, the input for `Simulation.add_tracked_junctions()` is a dictionary, where each junction has a '<i>flow_params</i>' value. This is a dictionary containing '<i>inflow_detectors</i>', '<i>outflow_detectors</i>' and '<i>vehicle_types</i>'. '<i>inflow_detectors</i>' and '<i>outflow_detectors</i>' are lists containing the detectors that will be used to register vehicles entering and exiting the junction, whilst '<i>vehicle_types</i>' is an optional parameter that lists the vehicle types that should be registered (defaults to all).

  3. Metered junctions:
    - Metered junctions adds options for tracking queuing variables; queue length, queue delay and spillback. This is primarily designed for use in ramp metering systems.
    - Here, the input for `Simulation.add_tracked_junctions()` is a dictionary, where each junction has a 'meter_params' value. This is a dictionary containing '<i>min_rate</i>', '<i>max_rate</i>', '<i>init_rate</i>', '<i>queue_detector</i>' and '<i>ramp_edges</i>'. These are detailed below, however, flow can still be tracked as above.
    - '<i>min_rate</i>' is a required parameter denoting the minimum allowed metering rate.
    - '<i>max_rate</i>' is a required parameter denoting the maximum allowed metering rate.
    - '<i>init_rate</i>' denotes the initial metering rate set after initialisation.
    - '<i>queue_detector</i>' is the ID of a <b>multi-entry-exit detector</b> that should be placed on the on-ramp. The entry detector should be placed at the start of the on-ramp, and the exit detector just before the traffic light. If given, queue length and delay at each step is tracked automatically.
    - If you would like to track vehicle spillback, provide the list of edge IDs that make up the ramp under the '<i>ramp_edges</i>' parameter. This can also be used instead of '<i>queue_detector</i>' to track queue length and delay, however, this approach is much slower. Tracking spillback in this way will only work when vehicles are inserted directly onto the on-ramp (ie. no urban roads connect to the on-ramp), and requires checking all unloaded vehicles of their insertion edge which can be slow.

Examples of these different levels are shown below.

```python
# Level 1: Only signal phases & green/red times are tracked
my_sim.add_tracked_junctions(["traffic_signal_1", "traffic_signal_2"])

# Level 2: Track inflow/outflow (& signal phases & green/red times)
my_sim.add_tracked_junctions({"traffic_signal_3":
                                {"flow_params":
                                    {"inflow_detectors":  ["det_in_1", "det_in_2"],
                                     "outflow_detectors": ["det_out_1", "det_out_2"],
                                     "vehicle_types": ["cars", "lorries", "motorcycles", "vans"]}
                                }
                            })

# Level 3: Track metering (& inflow/outflow & signal phases & green/red times)
my_sim.add_tracked_junctions({"ramp_meter":
                                {"meter_params":
                                    {"min_rate": 200,
                                    "max_rate": 2000,
                                    "queue_detector": "ramp_queue"},
                                "flow_params":
                                    {"inflow_detectors": ["ramp_inflow", "ramp_upstream"],
                                    "outflow_detectors": ["ramp_downstream"]}
                                }
                            })
```

All data from tracked junctions is stored in the `sim_data` dictionary under '<i>data/junctions/{junction_id}</i>'. This will contain the junction's '<i>position</i>', '<i>init_time</i>' and '<i>curr_time</i>' (referring to the start and ent time of the data collection). Phase data is then stored under '<i>tl</i>', whilst flow and metering data are under '<i>flow</i>' and '<i>meter</i>' respectively.

Data collection for tracked junctions can be reset using the function below, however, data collection is also reset when using `Simulation.reset_data()`.

```python
my_sim.tracked_junctions["traffic_signal_1"].reset()
```

## Tracked Edges

Tracked edges are very simple to initialise, but include a lot of useful data collection. Edges are only tracked using their ID with the `Simulation.add_tracked_edges()` function.

```python
# Add all edges as a tracked edge
my_sim.add_tracked_edges()

# Add a list of edges as tracked edges
my_sim.add_tracked_edges(["edge_1", "edge_2", "edge_3", "edge_4", "edge_5"])
```

All data from tracked edges is stored in the `sim_data` dictionary under '<i>data/edges/{edge_id}</i>'. The main data collected is under '<i>data/edges/{edge_id}/step_vehicles</i>'. This includes the vehicle ID, position, speed and lane index for all vehicles on the edge at each step. This data can be used to calculate vehicle counts and average speeds, as well as plot trajectories and space-time diagrams. Otherwise, '<i>linestring</i>', '<i>length</i>', '<i>to_node</i>', '<i>from_node</i>', '<i>n_lanes</i>', '<i>init_time</i>' and '<i>curr_time</i>' are also stored for each edge.

The step vehicle data is stored in a (4x1) array as follows:

  1. Vehicle ID
  2. Vehicle position measured as [0 - 1], denoting percent travelled along the edge
  3. Vehicle speed in `Simulation` class units
  4. Lane index [0 - no. lanes]

Data collection for tracked edges can be reset using the function below, however, data collection is also reset when using `Simulation.reset_data()`.

```python
my_sim.tracked_edges["edge_1"].reset()
```