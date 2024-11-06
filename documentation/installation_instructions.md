# System requirements

## Operating System
nnU-Net has been tested on Linux (Ubuntu 18.04, 20.04, 22.04; centOS, RHEL), Windows and MacOS! It should work out of the box!

## Hardware requirements
We support GPU (recommended), CPU and Apple M1/M2 as devices (currently Apple mps does not implement 3D 
convolutions, so you might have to use the CPU on those devices).

### Hardware requirements for Training
We recommend you use a GPU for training as this will take a really long time on CPU or MPS (Apple M1/M2). 
For training a GPU with at least 10 GB (popular non-datacenter options are the RTX 2080ti, RTX 3080/3090 or RTX 4080/4090) is 
required. We also recommend a strong CPU to go along with the GPU. 6 cores (12 threads) 
are the bare minimum! CPU requirements are mostly related to data augmentation and scale with the number of 
input channels and target structures. Plus, the faster the GPU, the better the CPU should be!

### Hardware Requirements for inference
Again we recommend a GPU to make predictions as this will be substantially faster than the other options. However, 
inference times are typically still manageable on CPU and MPS (Apple M1/M2). If using a GPU, it should have at least 
4 GB of available (unused) VRAM.

# Installation instructions
We strongly recommend that you install nnU-Net in a virtual environment! Pip or anaconda are both fine. If you choose to 
compile PyTorch from source (see below), you will need to use conda instead of pip. 

Use a recent version of Python! 3.9 or newer is guaranteed to work!

**nnU-Net v2 can coexist with nnU-Net v1! Both can be installed at the same time.**

1) Install [PyTorch](https://pytorch.org/get-started/locally/) as described on their website (conda/pip). Please 
install the latest version with support for your hardware (cuda, mps, cpu).
**DO NOT JUST `pip install nnunetv2` WITHOUT PROPERLY INSTALLING PYTORCH FIRST**. For maximum speed, consider 
[compiling pytorch yourself](https://github.com/pytorch/pytorch#from-source) (experienced users only!). 
2) Install nnU-Net depending on your use case:
    1) For use as **standardized baseline**, **out-of-the-box segmentation algorithm** or for running 
     **inference with pretrained models**:

       ```pip install nnunetv2```

    2) For use as integrative **framework** (this will create a copy of the nnU-Net code on your computer so that you
   can modify it as needed):
          ```bash
          git clone https://github.com/MIC-DKFZ/nnUNet.git
          cd nnUNet
          pip install -e .
          ```
3) nnU-Net needs to know where you intend to save raw data, preprocessed data and trained models. For this you need to
   set a few environment variables. Please follow the instructions [here](setting_up_paths.md).

Installing nnU-Net will add several new commands to your terminal. These commands are used to run the entire nnU-Net
pipeline. You can execute them from any location on your system. All nnU-Net commands have the prefix `nnUNetv2_` for
easy identification.

Note that these commands simply execute python scripts. If you installed nnU-Net in a virtual environment, this
environment must be activated when executing the commands. You can see what scripts/functions are executed by 
checking the entry_points in the setup.py file.

