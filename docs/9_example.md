# Example

An complex example of TUD-SUMO with multiple controllers and object tracking can be found here, at [github.com/tud-sumo/example](https://github.com/tud-sumo/example), however, this example is broken down below.

1. Initialise the simulation object.
```python
my_sim = Simulation(scenario_name="A20_ITCS", scenario_desc="Example traffic controllers, with ramp metering, a VSL controller and a route guidance controller.")
```

2. Start the simulation and connection to TraCI and define the sumo config file. Here, metric units are used and a seed of 1 is given.
```python
my_sim.start("example_scenario/a20.sumocfg", get_individual_vehicle_data=False, gui=False, seed="1", units="metric")
```

3. Add a tracked junction to the intersection with ID "utsc", which will track signal phases/times.
```python
my_sim.add_tracked_junctions({"utsc":
                                 {"flow_params":
                                     {"inflow_detectors": ["utsc_n_in_1", "utsc_n_in_2", "utsc_w_in", "utsc_e_in"],
                                      "outflow_detectors": ["utsc_w_out", "utsc_e_out"],
                                      "vehicle_types": ["cars", "lorries", "motorcycles", "vans"]
                                     }
                                 }
                            })
```

4. Set traffic signal phases for the signal with ID '<i>utsc</i>'. Here, there are 4 phases in the 60s cycle and the junction has 4 movements.
```python
my_sim.set_phases({"utsc":
                      {"phases": ["GGrr", "yyrr", "rrGG", "rryy"],
                       "times": [27, 3, 17, 3]
                      }
                 })
```

5. Track the junction '<i>crooswijk_meter</i>', where there is an active ramp meter. Both meter and flow parameters are given to track meter specific variables (queue length, queue delay) and inflow/outflow.
```python
my_sim.add_tracked_junctions({"crooswijk_meter":
                                {'meter_params':
                                    {'min_rate': 200,
                                     'max_rate': 2000,
                                     'queue_detector': "cw_ramp_queue"
                                    },
                                 'flow_params':
                                    {'inflow_detectors': ["cw_ramp_inflow", "cw_rm_upstream"],
                                     'outflow_detectors': ["cw_rm_downstream"]
                                    }
                                }
                            })
```

6. Add Route Guidance (RG) & Variable Speed Limit (VSL) controllers. A RG controller is placed on the detector '<i>rerouter_det</i>' and redirects drivers to the edge '<i>urban_out_w</i>'. The VSL controller is active on edges with IDs '<i>126729982</i>', '<i>126730069</i>' and '<i>126730059</i>'.
```python
my_sim.add_controllers({"rerouter":
                          {"type": "RG",
                           "detector_ids": ["rerouter_det"],
                           "new_destination": "urban_out_w",
                           "diversion_pct": 1,
                           "highlight": "00FF00"
                          },
                        "vsl":
                          {"type": "VSL",
                           "geometry_ids": ["126729982", "126730069", "126730059"]
                          }
                      })
```

7. Start tracking edges with IDs '<i>126730026</i>', '<i>1191885773</i>', '<i>1191885771</i>', '<i>126730171</i>' and '<i>1191885772</i>'.
```python
my_sim.add_tracked_edges(["126730026", "1191885773", "1191885771", "126730171", "1191885772"])
```

8. Add scheduled events from an '<i>example_incident.json</i>' JSON file.
```python
my_sim.add_events("example_scenario/example_incident.json")
```

9. Run a control loop, checking current step with `my_sim.curr_step`.
```python
n, sim_dur, new_veh_idx = 1 / my_sim.step_length, 500 / my_sim.step_length, 0
while my_sim.curr_step < sim_dur:
```

10. Perform (random) ramp metering control.
```python
    if my_sim.curr_step % 50 / my_sim.step_length == 0:
        my_sim.set_tl_metering_rate(rm_id="crooswijk_meter", metering_rate=randint(1200, 2000))
        my_sim.set_tl_metering_rate(rm_id="a13_meter", metering_rate=randint(1200, 2000))
```

11. Step through simulation.
```python
    my_sim.step_through(n_steps=n, pbar_max_steps=sim_dur)
```

12. Dynamically add new vehicles driving from '<i>urban_in_e</i>' to '<i>urban_out_w</i>'.
```python
    # Add new vehicles going from "urban_in_e" to "urban_out_w"
    if my_sim.curr_step % 50 / my_sim.step_length == 0:
        od_pair = ("urban_in_e", "urban_out_w")
        my_sim.add_vehicle(vehicle_id="lorry_"+str(new_veh_idx), vehicle_type="lorries", routing=od_pair, origin_lane="first")
        my_sim.add_vehicle(vehicle_id="car_"+str(new_veh_idx), vehicle_type="cars", routing=od_pair)
        new_veh_idx += 1
```

13. Dynamically create an incident with 2 vehicles at 100s, lasting for 100s. Edge speed is reduced to 5kmph.
```python
    if my_sim.curr_step == 100 / my_sim.step_length:
        my_sim.cause_incident(100, n_vehicles=2, edge_speed=5)
```

14. Activate the RG and VSL controllers.
```python
    if my_sim.curr_step == 250 / my_sim.step_length:
        # Activate controllers & update UTSC phases.
        my_sim.controllers["rerouter"].activate()
        my_sim.controllers["vsl"].set_speed_limit(60)

        my_sim.set_phases({"utsc": {"phases": ["GGrr", "yyrr", "rrGG", "rryy"], "times": [37, 3, 7, 3]}}, overwrite=False)
```

15. Once the control loop is complete, end the simulation then save (& summarise) the resulting data.
```python
my_sim.end()

my_sim.save_data("example_data.json")
my_sim.print_summary(save_file="example_summary.txt")
```