# Welcome to the TUD-SUMO Wiki!

![header](img/header.png)
<br><br>
This is the documentation for the TUD-SUMO package, a research-oriented wrapper for SUMO<sup>[1]</sup>, developed for the DIAMoND lab at the Technische Universiteit Delft (TUD), the Netherlands.

The main goal of TUD-SUMO is to act as a simplified framework for microscopic traffic simulation that allows researchers and students to focus on the important aspects of their projects; <b>their own work</b>. TUD-SUMO provides an easy and standardised way to simulate a wide range of scenarios whilst facilitating complex interactions. Resulting data can then be saved, summarised and visualised with minimal code.

More information on "Simulation of Urban MObility" (SUMO) can be found in the SUMO documentation: [sumo.dlr.de/docs/](https://sumo.dlr.de/docs/)

The main features of TUD-SUMO include:

  - Automatic and standardised data collection.
  - Simple interface to interact with and control the simulation.
  - Traffic signal control logic.
  - Extendable controllers already implemented (ramp metering, route guidance and variable speed limits).
  - An event system with dynamic or scheduled incidents.
  - Plotting functions for a wide range of applications.
  - And <i>more in the future!</i>

![logos](img/logos.png)

## Latest Version

TUD-SUMO was last updated on 12/07/2024. The changenotes for this version are:

### Demand Generation, More Getters and General Fixes 

#### Added
  - Added dynamic demand generation with `Simulation.load_demand()` and `Simulation.add_demand()` functions.
  - Added demand to `sim_data` dictionary.
  - Added number of waiting vehicles to collected data.
  - Added more getters for data: `Simulation.get_[no_vehicles/no_waiting/tts/delay]()`.
  - Added `Simulation.get_[junction/tracked_junction/tracked_edge/event/controller]_exists()` functions.
  - Added `Simulation.get_[junction/tracked_junction/tracked_edge/event/controller]_ids()` functions.
  - Added `Simulation.remove_controllers()` function.
  - Added `Plotter.plot_od_demand()` function.
  - Added `utils.conver_units()` function and removed `utils.convert_time_units()`.
  - Added `EventScheduler.get_event_ids()` function to get status of event.
  - Added basic `Plotter.plot_fundamental_diagram()` function.
  - Added `'incoming_edges'`, `'outgoing_edges'`, `'junction_ids'`, `'ff_travel_time'` and `'curr_travel_time'` to `Simulation.get_geometry_vals()`.

#### Changes
  - Improved error handling.
  - Added vehicle type filter to `Simulation.get_all_vehicle_data()` function.
  - Changed `Simulation.vehicle_departed()` to `Simulation.vehicle_to_depart()`.
  - Changed `Simulation.tracked_juncs` to `Simulation.tracked_junctions`.
  - Changed 's' to 'seconds', 'm' to 'minutes', 'hr' to 'hours' wherever they appear.
  - `VSLController` data now stored in the same format as `RGController` with an activation times list.
  - Simplified activation times data in `RGController`.
  - Changed `VSLController.set_limit()` to `VSLController.set_speed_limit()`.
  - Changed `'EDGE'` and `'LANE'` to lowercase when getting geometry types.
  - Changed `'stopped'` to `'is_stopped'` in vehicle data.
  - Removed `'event_n_steps'` and `'event_duration'` from event parameters.

## Contact

TUD-SUMO is developed by Callum Evans in the DIAMoND lab of TU Delft. For any questions, feedback or bug reports, please contact Callum Evans or submit a query using the form [here](https://forms.office.com/e/pMnGaheier).

## Citations

  1. "Microscopic Traffic Simulation using SUMO"; Pablo Alvarez Lopez, Michael Behrisch, Laura Bieker-Walz, Jakob Erdmann, Yun-Pang Flötteröd, Robert Hilbrich, Leonhard Lücken, Johannes Rummel, Peter Wagner, and Evamarie Wießner. IEEE Intelligent Transportation Systems Conference (ITSC), 2018.