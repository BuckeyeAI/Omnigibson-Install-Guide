# Omnigibson-Install-Guide
An installation guide on installing Omnigibson on Ubuntu 20.04

# Install OmniGibson (Docker Image)

Follow the official guide to install the docker image. When running the `run_docker.sh` script, if prompted with a license (EULA) acceptance, type 'y' for yes.

## Possible issue #1 (path mismatch in `run_docker.sh`)

In the `run_docker.sh` file provided by the official guide, you might need to change the default path according to your local nvidia driver installation destination. For my case (Ubuntu 20.04 and NVIDIA Driver 525.60.11), the modified paths are:

```
ICD_PATH="/etc/vulkan/icd.d/nvidia_icd.json"
LAYERS_PATH="/etc/vulkan/implicit_layer.d/nvidia_layers.json"
```

Exit docker container: `Ctrl + D`.

## Issue of `AttributeError: 'ContactParticles' object has no attribute 'reset'`
This is a common error mentioned in this [issue post](https://github.com/StanfordVL/OmniGibson/issues/111). The remedy is to comment out the line `self.reset()` in `omnigibson/object_states/object_state_base.py`. But editing files inside a docker requires installing the editors inside the container environment:
```
apt-get update
apt-get install vim
```

## Issue of Isaac Sim not showing up in Docker container
This is not resolved. Symptom: running the example scripts in the docker image does not trigger Isaac Sim to be opened.


# Install OmniGibson (from source)


1. **Install conda if you haven't yet** by going to the official [site](https://docs.conda.io/projects/conda/en/latest/user-guide/install/linux.html) and download the Miniconda installer for Linux. Make sure your OS python version is greater than 3.7, because the conda environmnent specified in `setup_conda_env.sh` specifies python 3.7. Then open terminal in your local folder containing the file, type:
```
bash Miniconda3-latest-Linux-x86_64.sh
```
(in my case it is `bash Miniconda3-py38_23.1.0-1-Linux-x86_64.sh`)

Try verify the installation by typing `conda --version`. If it says `conda: command not found` it is likely that the `bin` folder of your miniconda installation was not added to your `PATH` variable automatically. To manually add it, add this line at the end of your `~/.bashrc` file:
```
export PATH="/home/username/miniconda/bin:$PATH"
```
assuming `/home/username/miniconda` is your (default) miniconda installation directory. In my case, I use
```
export PATH="/home/tong/miniconda3/bin:$PATH"
```


2. Setup the conda environment listed in `setup_conda_env.sh`. But before running `sh ./setup_conda_env.sh`, make sure you have setup the environment variable `ISAAC_SIM_PATH` as the path to your local installation of Isaac Sim. For my case, it is `/home/tong/.local/share/ov/pkg/isaac_sim-2022.2.0/`, you can find yours using the `find` command: `find / -name isaac`. so set up the variable using:

```
nano ~/.bashrc
```
then inside `.bashrc`, at the end of the file, add:
```
export ISAAC_SIM_PATH=/home/tong/.local/share/ov/pkg/isaac_sim-2022.2.0
```
**NOTE:** do NOT add an extra `/` at the end of the path, it will conflict with `setup_conda_env.sh`, which expect there to be no `/` at the end of `ISAAC_SIM_PATH`.
press `Ctrl+X` to exit the file, `Y` to save changes, then type:
```
source ~/.bashrc
```
verify by typing
```
echo $ISAAC_SIM_PATH
```

3. Execute `sh ./setup_conda_env.sh` or manually do it line by line if more comfortable that way. If everything is successful, you have installed Omnigibson in its full extent. What this script does: creates a conda env called `omnigibson`, then link the Isaac sim PATHs into this env. When deactivating, these paths will be also removed. Then it installs the dependencies specfied in `setup.py`.

4. Exit the terminal and restart the terminal (important).

5. Reactivate the conda environment. Download the dataset by calling the `setup.py` script:
```
python -m omnigibson.scripts.setup.py
```

6. Then try execute some Omnigibson functions, such as
```
python -m omnigibson.examples.scenes.scene_selector 
```

## Possible errors during installation/running

It is possible that you encounter error runing the code above. 

1. For example, you can get an error about `omni` when executing the examples:
```
ModuleNotFoundError: No module named 'omni'
```
This is likely caused by you not installing the omniverse python environment, or because you just installed the conda environment `omnigibson` and did not restart the terminal yet. This [official guide](https://docs.omniverse.nvidia.com/app_isaacsim/app_isaacsim/install_python.html) walks through how to set up the python access to Isaac Sim.


2. Or you can get an error about not finding the dataset, such as:
```
FileNotFoundError: [Errno 2] No such file or directory: '/home/tong/pm/OmniGibson/omnigibson/data/og_dataset/scenes'
```
This is likely caused by not having downloaded the dataset. I have no idea yet where to download the dataset. But I did find out that the dataset is automatically downloaded using the [Docker container installation approach](https://behavior.stanford.edu/omnigibson/getting_started/installation.html). It asks you to provide a `[DATA_PATH]` during the call `sh ./run_docker.sh [DATA_PATH]`, and it will dump the dataset to this `[DATA_PATH]`. You can manually copy the content under this `[DATA_PATH]` to where Omnigibson is complaining, i.e., `/home/tong/pm/OmniGibson/omnigibson/data/og_dataset/scenes` in my case. In my case, `[DATA_PATH] = /home/tong/.local/shared/ov/data`, so I just needed to copy the existing dataset in my `[DATA_PATH]` to `/home/tong/pm/OmniGibson/omnigibson/data` by:
```
cp -r ~/.local/shared/ov/data/datasets ~/pm/OmniGibson/omnigibson/data/
```

3. The `reset()` issue. It is possible that you will get error related to the `reset()` attribute not found. This is a [known](https://github.com/StanfordVL/OmniGibson/issues/111) issue, and the hot-fix is to comment out line 89 in `omnigibson/object_states/object_state_base.py`.

```
AttributeError: 'InsideRoomTypes' object has no attribute 'reset'
```