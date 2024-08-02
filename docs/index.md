# Welcome to the TUD-SUMO Wiki!
<p align="center">
  <img src="img/header.png" />
  <br><br>
  <a href="https://github.com/tud-sumo/tud_sumo" alt="GitHub">
        <img src="https://img.shields.io/badge/v3.0.8-%2338A6D6?logo=github&link=https%3A%2F%2Fgithub.com%2Ftud-sumo%2Ftud_sumo
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

The Latest version of TUD-SUMO is <i>v3.0.8</i>, and was released on 02/08/2024. The changenotes for this version are:

### Paths & Vehicle Functions Update

#### Changes & Improvements

  - Added `Simulation.add_vehicle_in_funcs()` and `Simulation.add_vehicle_out_funcs()` to add functions called with each vehicle that enters/exits the simulation. Also added `Simulation.remove_vehicle_in_funcs()` and `Simulation.remove_vehicle_out_funcs()`.
  - Added `Simulation.get_path_edges()` to calculate a route between two edges using the A* algorithm.
  - Added `Simulation.get_path_travel_time()`, `Simulation.is_valid_path()` and `Simulation.add_route()`.
  - Added `Plotter.plot_trip_time_histogram()` to plot trip time distribution.
  - Added `Plotter.plot_throughput()` to plot throughput (rate of completed trips).
  - Added `aggregation_steps` to `Plotter.plot_[vehicle/detector]_data()`, to aggregate/smooth data when plotting.
  - Changed `line_colour` to a more general `plt_colour` in `Plotter` functions.
  - More standardisation of docstrings/function definitions.
  - Started implementing more standardised type checking (`validate_list_types()` and `validate_type()`).

#### Bug Fixes

  - Fixed error where vehicle data was not being saved.
  - Fixed `Simulation.get_geometry_vals()` function not recognising `"length"` as a valid data key.
  - Fixed incorrect `"curr_travel_time"` and `"ff_travel_time"` calculations.
  - Fixed `Plotter.plot_od_trip_times()` not using `vehicle_types` parameter.
  - Removed TraCI calls from `RGController`.

## Contact

TUD-SUMO is developed by Callum Evans in the DIAMoND lab of TU Delft. For any questions, feedback or bug reports, please contact Callum Evans or submit a query using the form [here](https://forms.office.com/e/pMnGaheier).

## Citations

  1. "<i>Microscopic Traffic Simulation using SUMO</i>"; Pablo Alvarez Lopez, Michael Behrisch, Laura Bieker-Walz, Jakob Erdmann, Yun-Pang Flötteröd, Robert Hilbrich, Leonhard Lücken, Johannes Rummel, Peter Wagner, and Evamarie Wießner. <i>IEEE Intelligent Transportation Systems Conference (ITSC)</i>, 2018.