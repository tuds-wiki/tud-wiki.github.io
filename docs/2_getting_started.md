# Getting Started

## Requirements

To run TUD-SUMO, first install SUMO. Platform specific information on how to do this can be found in the SUMO documentation [here](https://sumo.dlr.de/docs/Installing/index.html).

Python 3.10 or later is required to run TUD-SUMO, otherwise, the required dependencies are:

  - `traci`
  - `sumolib`
  - `matplotlib`
  - `shapely`
  - `tqdm`

## Creating an Environment

It is recommended to use install the required packages in an environment, such as using [Anaconda](https://docs.anaconda.com/anaconda/install/) or [Miniconda](https://docs.anaconda.com/miniconda/). A conda environment ready for TUD-SUMO can be created using the following commands.

```
conda create --name tud-sumo
conda activate tud-sumo
conda install matplotlib tqdm
pip install traci sumolib shapely
```

Conda can then be deactivated later using the command:

```
conda deactivate tud-sumo
```