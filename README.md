# Neuroevolution of Recurrent Architectures on Control Tasks

![Videos](videos.gif)

This repository contains the instructions on how to reproduce the experiments from the paper ["Neuroevolution of Recurrent Architectures on Control Tasks"](https://openreview.net/pdf?id=rhMG7QDWIZq) published at GECCO 2022 & the ALOE ICLR 2022 Workshop.

The code itself is located in a larger library called [Nevo](https://github.com/MaximilienLC/nevo).


### Installation (tested on Ubuntu 20.04)

```
# Debian packages                       ----------mpi4py---------- ~~~~~~~~~~~~~~~~~Gym~~~~~~~~~~~~~~~~~~~
sudo apt install git python3-virtualenv python3-dev libopenmpi-dev g++ swig libosmesa6-dev patchelf ffmpeg

# MuJoCo
wget https://mujoco.org/download/mujoco210-linux-x86_64.tar.gz -P ~/Downloads/
mkdir -p ~/.mujoco/ && tar -zxf ~/Downloads/mujoco210-linux-x86_64.tar.gz -C ~/.mujoco/
echo -e "\n# MuJoCo\nMUJOCO_PATH=~/.mujoco/mujoco210/bin
export LD_LIBRARY_PATH=\$MUJOCO_PATH:\$LD_LIBRARY_PATH" >> ~/.bashrc
source ~/.bashrc

# Library & Dependencies
git clone https://github.com/MaximilienLC/nevo && cd nevo/
virtualenv venv && source venv/bin/activate && pip3 install -r requirements.txt
```

### Execution

`<task>` = `acrobot`, `cart_pole`, `mountain_car`, `mountain_car_continuous`, `pendulum`, `bipedal_walker`, `bipedal_walker_hardcore`, `lunar_lander`, `lunar_lander_continuous`, `ant`, `half_cheetah`, `hopper`, `humanoid`, `humanoid_standup`, `inverted_double_pendulum`, `inverted_pendulum`, `reacher`, `swimmer` or `walker_2d`  
`<net>` = `static/rnn` or `dynamic/rnn`  

```
mpiexec -n <n> python3 main.py --env_path envs/multistep/score/control.py \
                               --bots_path bots/network/<net>/control.py \
                               --nb_generations <nb_generations> \
                               --population_size <population_size> \
                               --additional_arguments '{"task" : "<task>"}'
```

Example from the paper: (you can increase the number of MPI processes if your machine allows it)
```
mpiexec -n 2 python3 main.py --env_path envs/multistep/score/control.py \
                             --bots_path bots/network/dynamic/rnn/control.py \
                             --nb_generations 100 \
                             --population_size 16 \
                             --additional_arguments '{"task" : "acrobot"}'
```

### Downloading the Paper's Results & Final Dynamic States

```
wget https://www.dropbox.com/s/if66cy3ydep6xj0/envs.multistep.control.score.zip -P ~/Downloads/
unzip -o ~/Downloads/envs.multistep.control.score.zip -d data/states/
```

You can now run additional generations ...
```
mpiexec -n 4 python3 main.py --env_path envs/multistep/score/control.py \
                             --bots_path bots/network/dynamic/rnn/control.py \
                             --nb_elapsed_generations 100 \
                             --nb_generations 10 \
                             --population_size 16 \
                             --additional_arguments '{"task" : "cart_pole"}'
```

... Evaluate the new agents ...
```
mpiexec -n 4 python3 utils/evaluate.py --states_path data/states/envs.multistep.score.control/steps.0~task.cart_pole~transfer.no~trials.1/bots.network.dynamic.rnn.control/16/
```

... And both record the elite's behaviour and obtain its architecture.
```
python3 utils/record.py --state_path data/states/envs.multistep.score.control/steps.0~task.cart_pole~transfer.no~trials.1/bots.network.dynamic.rnn.control/16/110/
```

### Reproducing the Paper's Figures

```
# Verify Stable Baselines 3 Results (not necessary for later steps)
git clone --recursive https://github.com/DLR-RM/rl-baselines3-zoo ~/rl-baselines3-zoo
jupyter notebook utils/notebooks/neuroevolution-recurrent-architectures/sb3_baselines.ipynb

# Visualize the dynamic networks
jupyter notebook utils/notebooks/neuroevolution-recurrent-architectures/dynamic_rnn.ipynb

# Reproduce the paper's figures  
jupyter notebook utils/notebooks/neuroevolution-recurrent-architectures/figures.ipynb
```
