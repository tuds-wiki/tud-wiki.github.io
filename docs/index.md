# Welcome to the TUD-SUMO Wiki!
<p align="center">
  <img src="img/header.png" />
  <br><br>
  <a href="https://github.com/tud-sumo/tud_sumo" alt="GitHub">
        <img src="https://img.shields.io/badge/v3.0.7-%2338A6D6?logo=github&link=https%3A%2F%2Fgithub.com%2Ftud-sumo%2Ftud_sumo
        " /></a>
  <a href="https://pypi.org/project/tud-sumo/" alt="PyPI">
        <img src="https://img.shields.io/badge/PyPI-%2338A6D6?logo=pypi&logoColor=white&link=https%3A%2F%2Fgithub.com%2Ftud-sumo%2Ftud_sumo
        " /></a>
</p>

This is the documentation for the TUD-SUMO package, a research-oriented wrapper for SUMO<sup>[1]</sup>, developed for the DIAMoND lab at the Technische Universiteit Delft (TUD), the Netherlands.

The main goal of TUD-SUMO is to act as a simplified framework for microscopic traffic simulation that allows researchers and students to focus on the important aspects of their projects; <b>their own work</b>, instead of simulation code. TUD-SUMO provides an easy and standardised way to simulate a wide range of scenarios whilst facilitating complex interactions. Resulting data can then be saved, summarised and visualised with minimal code.

More information on "Simulation of Urban MObility" (SUMO) can be found in the SUMO documentation, here: [sumo.dlr.de/docs/](https://sumo.dlr.de/docs/)

The main features of TUD-SUMO include:

  - Automatic and standardised data collection.
  - Simple interface to interact with and control the simulation.
  - Traffic signal control logic.
  - Extendable controllers already implemented (ramp metering, route guidance and variable speed limits).
  - An event system with dynamic or scheduled incidents.
  - Plotting functions for a wide range of applications.
  - And <i>more in the future!</i>

![logos](img/logos.png)

## Links

1. Simulation of Urban MObility (SUMO) documentation: [sumo.dlr.de/docs/](https://sumo.dlr.de/docs/)
2. TUD-SUMO source code: [github.com/tud-sumo/tud_sumo](https://github.com/tud-sumo/tud_sumo/)
3. TUD-SUMO PyPI distribution: [pypi.org/project/tud-sumo/](https://pypi.org/project/tud-sumo/)
4. TUD-SUMO example: [github.com/tud-sumo/example](https://github.com/tud-sumo/example)

## Latest Version

The Latest version of TUD-SUMO is v3.0.7, and was released on 26/07/2024. The changenotes for this version are:

### Speed, Flow & Density Update and Console Improvements

#### Changes & Improvements

  - Added speed, flow and density data to tracked edges.
  - Added `lane_idx` parameter `Plotter.plot_trajectories()` to allow for plotting trajectories by lane.
  - Added `suppress_traci_warnings` parameter to hide emergency braking, collision warnings etc.
  - Added `SimulationError` for simulation-specific errors.
  - Improved progress bar (changed to automatically show, added `pbar_max_steps` parameter to allow for persistent progress bars through multiple `Simulation.step_through()` calls).
  - Removed `ignore_TraCI_err` parameter.
  - Changed `Plotter.plot_od_demand()` to plot network-wide demand by default.
  - Added `Simulation.get_demand_table()` function to fetch demand inputs.

#### Bug Fixes

  - Fixed error in `Simulation.cause_incident()` where no `EventScheduler` object was created.
  - Fixed incorrect `scenario_name` error.
  - Fixed `suppress_warnings` for TUD-SUMO warnings.
  - Fixed no `'vehicle_type_dists'` parameter error when adding demand.
  - Fixed `Simulation.get_geometry_ids()` returning empty arrays.
  - Fixed `sim_dur` parameter in `Simulation.step_through()`.
  - Removed invalid '<i>route_edges</i>' subscription.

## Contact

TUD-SUMO is developed by Callum Evans in the DIAMoND lab of TU Delft. For any questions, feedback or bug reports, please contact Callum Evans or submit a query using the form [here](https://forms.office.com/e/pMnGaheier).

## Citations

  1. "<i>Microscopic Traffic Simulation using SUMO</i>"; Pablo Alvarez Lopez, Michael Behrisch, Laura Bieker-Walz, Jakob Erdmann, Yun-Pang Flötteröd, Robert Hilbrich, Leonhard Lücken, Johannes Rummel, Peter Wagner, and Evamarie Wießner. <i>IEEE Intelligent Transportation Systems Conference (ITSC)</i>, 2018.