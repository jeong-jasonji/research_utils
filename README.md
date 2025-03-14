# research_utils
Common commands and utils to take note of when working in lab.

Feel free to add to the README or add scripts that makes life easier here.

./pytorch_training: contains common pytorch training tips, tricks, and mistakes (dataloading, transforms, modes, etc.)

# envrionment files
### ncsnv2.yml
> Environment.yml file for "Improved Techniques for Training Score-Based Generative Models." https://github.com/ermongroup/ncsnv2
### SAM.yml
> Environment.yml file for "Segment Anything" [https://github.com/ermongroup/ncsnv2](https://github.com/facebookresearch/segment-anything)
### transformers.yml
> Environment.yml file for [HuggingFace](https://huggingface.co/)
### uvcgan2.yml
> Environment.yml file for "UVCGAN v2: An Improved Cycle-Consistent GAN for Unpaired Image-to-Image Translation" [UVCGANv2](https://github.com/LS4GAN/uvcgan2/tree/main)

# conda environments
```python
# create a new conda environment
conda create --name <my-env>

# create environment from yml file
conda env create -f environment.yml

# export current environment to yml file
conda env export > environment.yml

# check list of currently available environments
conda info --envs
```

# switching from conda (not free) to miniconda
0. double check you are using anaconda using by activating conda and using: ``` which python3 ```
1. backup all your environments (including base) ``` activate the environment you want to back up and run the command: 'conda env export > environment.yml' ```
2. delete the conda folder in your home directory ./anaconda
3. install miniconda - during installation, allow it to install miniconda as a default
4. double check your .condarc and .bashrc file to see if bash will use miniconda
5. make sure in your miniconda folder, the .condarc doesn't use ```- https://repo.anaconda.com/pkgs/main or - https://repo.anaconda.com/pkgs/r ``` in its channels
6. instead use the free channels: ```- https://repo.anaconda.com/pkgs/free``` and conda-forge
7. double check the channels in your miniconda environment with ``` conda config --show channels ```
8. if default is still in your channels, use the command ``` conda config --remove channels defaults ```

# Jupyter Notebook:
### Using Jupyter Notebook on server (MobaXTerm)
First set up an SSH tunnel with the following parameters (with your credentials):
![SSH Tunnel setup](images/port_forwarding_servers.png)
Then start the SSH tunnel, bash and then run the following command:
```python
jupyter notebook --no-browser
```
### Adding envrionments to Jupyter notebooks
```python
# in the activated environment first install ipykernel:
conda install -c anaconda ipykernel
# then install the environment as a usable kernel:
python -m ipykernel install --user --name=env_name
# list the kernels available:
jupyter kernelspec list
# if you want to remove a kernel:
jupyter kernelspec uninstall kernel_name
```
### changing the default start location of jupyter notebooks
```python
# generate a jupyter config file if it's already not already there (/.jupyter/jupyter_notebook_config.py)
jupyter notebook --generate-config
# find the config file (.../.jupyter/jupyter_notebook_config.py) and modify the default notebook directory
c.NotebookApp.notebook_dir = 'path_to_new_dir'
# uncomment the notebook_dir and save.
```
for further details see the following link: [How to change the Jupyter start-up folder](https://stackoverflow.com/questions/35254852/how-to-change-the-jupyter-start-up-folder)
# Tmux (running processes in the server without disconnecting)
```python
# create a new session with a session name (easier to figure out which session is which)
tmux new -s session_name

#Detaching from Tmux Session:
Ctrl+b d

#Listing current tmux sessions:
tumx ls

#Attaching to tmux sessions:
tmux attach-session -t named_session

#Killing sessions:
tmux kill-session -t 3

# scrolling through errors in copy mode:
Ctrl+b,[

# renaming sessions:
tmux rename-session -t current_name new_name
OR: Ctrl+b,$ (within the session to rename it)
```

# Cleaning up disk space:
### check disk space to a directory:
```python
du -sh directory_name
# to check hidden directories use:
du -sh .[^.]*
```
### remove directory recursively:
```python
rm -r directory_name
```
### cleaning up cache
> sometimes cache might be full and you might need to do some cleaning in the cache folder
```python
# if ./.cache/pip/ is quite full you can do a purge:
python -m pip cache purge
```
And check out [pip cache documentation](https://pip.pypa.io/en/stable/cli/pip_cache/) for more information.
### move conda envrionments to a different directory
> If you have many conda environments and lots of packages, your home directory might get large. To reduce the disk space in home (or any) directory, change the conda environment's default directories to one in a larger storage center (like DatacenterStorage). Refer to this guide for specific directions: [guide](https://stackoverflow.com/questions/67610133/how-to-move-conda-from-one-folder-to-another-at-the-moment-of-creating-the-envi) Otherwise a quick how-to are shown in the steps below:
1. Change default conda envrionment's pkgs_dirs and envs_dirs
```python
# change Conda packages directory
conda config --add pkgs_dirs /big_partition/users/user/.conda/pkgs
# change Conda environments directory
conda config --add envs_dirs /big_partition/users/user/.conda/envs
```
2. if starting from scratch, this is enough and you can start creating envirionments and they will be saved to the new default directories
3. if you're wanting to move conda environment directories, there's no direct way so you have to do the following steps:
4. Archive environments.
```python
conda env export -n foo > foo.yaml # One per environment.
```
5. Move package cache. (e.g. Copy contents of old package cache (/home/users/user_name/.conda/envs/.pkgs/) to new package cache.) This is mainly if you want to be very thorough about transferring and not having to redownload stuff for environments you already created.
6. Recreate environments.
```python
conda env create -n foo -f foo.yaml
```

# Copying to and from different servers
Use rsync: ```python rsync -r username@server1_IP:source_dir username@server2_IP:destination_dir ```
## if the source or destination is a local:
Use scp: 
```python  
scp username@serverIP:/server_dir/ local_dir  # server to local
scp local_dir username@serverIP:/server_dir/  # local to server
```

# Copying to and from  GCP
1. bash (to use gcloud)
2. Auth login first using:
```gcloud auth login --no-launch-browser ```
3. Do all the login stuff as necessary and use the authentication code.
4. Use gsutil to copy from one dir to another in the server:
```gsutil -m cp -r "gs://GCP_location /server_location/ ```

# Memory issues:
### Memory not being released (cuda running out of memory error)
> if you train a model and it runs for a few loops (batches or epochs) but then suddenly runs out of memory, the issue is probably that some variable is compounding and its memory is not being released. A few things you can do to debug is:
0) check memory usage after each loop using torch.cuda.memory_allocated()
```python
# this code will print device memory usage in MB, i being batch or epoch number
print('batch {}: {:.2f}MB'.format(i, float(torch.cuda.memory_allocated(device=DEV) / (1024 * 1024))))
```

1) collect garbage and release cache ([https://docs.python.org/3/library/gc.html](https://stackoverflow.com/questions/59129812/how-to-avoid-cuda-out-of-memory-in-pytorch))
```python
import gc
import torch

gc.collect()
torch.cuda.empty_cache()
```
2) zero grad the optimizers - PyTorch accumulates the gradients on subsequent backward passes and if you don't the gradient would be a combination of the old gradient, which you have already used to update your model parameters and the newly-computed gradient. (https://stackoverflow.com/questions/48001598/why-do-we-need-to-call-zero-grad-in-pytorch)
```python
optimizer.zero_grad(set_to_none=True)
```
3) make sure to add .items() not tensors in history or anything that will be evaluated at the end of the loop
```python
# if loss is a tensor and used in gradient calculations then loss_sum will accumulate memory
loss = loss_fn(x, y)
loss_sum += loss
# print(loss) will give something like: "tensor(0.3652, device='cuda:0', grad_fn=<MulBackward0>)"

# the .item() of the tensor will just give the value and remove any gradient 
loss = loss_fn(x, y)
loss_sum += loss.item()
# print(loss.item()) will give something like: "0.3651849031448364"
```

### pip running out of memory (can't install because of OS memory)
> this might occur if you have a new conda environment and trying to install a separate pip and packages on it
if so, try conda clean (Remove unused packages and caches.): https://docs.conda.io/projects/conda/en/latest/commands/clean.html
```python
conda clean -a
```

# Random Errors and quality of life stuff
### RuntimeError: main thread is not in main loop
Sometimes when running optuna training, this error will occur and for me, it was because of matplotlib and plotting losses within the trials. See this [issue](https://github.com/r9y9/deepvoice3_pytorch/issues/5) for more context. So you can either remove the plotting during optuna training or use the following lines:

```python
import matplotlib
matplotlib.use('Agg')
import matplotlib.pyplot as plt
```

### PIL image mode lookup
Most pytorch transforms and image manipulations will use PIL as the base so make sure the modes are correct (e.g. 8bit, 16bit, etc.) so that there's no clipping issues. See [PIL documentation](https://pillow.readthedocs.io/en/stable/handbook/concepts.html) for more details.
> quick lookup: RGB (3x8-bit pixels, true color), L (8-bit pixels, grayscale), I (32-bit signed integer pixels)

### Plus Minus sign (±)
```python
print('\u00B1')  # will give you the ±
```

# Git issues:
### commited large files to git repo and need to remove (blob.txt as an example):
> Github won't let you push large files to your repo and if you somehow got to the limit and wanted to push something small that would put it over the limit, it will cause issues. However, since Github keeps the history of your commits with the files, the solution is not as simple as removing the large files from your repo. So you'll need to delete the large files from your repo history through BFG Repo-Cleaner. (There are other ways to remove from the history but for me, it was the easiest and most straight forward method).

Use BFG Repo-Cleaner: https://rtyley.github.io/bfg-repo-cleaner/
3
