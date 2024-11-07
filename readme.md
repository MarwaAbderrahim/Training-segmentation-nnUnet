# Welcome to the nnU-Net!


# What is nnU-Net?
**nnU-Net is a semantic segmentation method that automatically adapts to a given dataset. It will analyze the provided 
training cases and automatically configure a matching U-Net-based segmentation pipeline. No expertise required on your 
end! You can simply train the models and use them for your application**.
nnU-Net relies on supervised learning, which means that you need to provide training cases for your application. The number of 
required training cases varies heavily depending on the complexity of the segmentation problem. No 
one-fits-all number can be provided here! 

## How does nnU-Net work?
Given a new dataset, nnU-Net will systematically analyze the provided training cases and create a 'dataset fingerprint'. 
nnU-Net then creates several U-Net configurations for each dataset: 
- `2d`: a 2D U-Net (for 2D and 3D datasets)
- `3d_fullres`: a 3D U-Net that operates on a high image resolution (for 3D datasets only)
- `3d_lowres` → `3d_cascade_fullres`: a 3D U-Net cascade where first a 3D U-Net operates on low resolution images and 
then a second high-resolution 3D U-Net refined the predictions of the former (for 3D datasets with large image sizes only)

**Note that not all U-Net configurations are created for all datasets. In datasets with small image sizes, the 
U-Net cascade (and with it the 3d_lowres configuration) is omitted because the patch size of the full 
resolution U-Net already covers a large part of the input images.**

nnU-Net configures its segmentation pipelines based on a three-step recipe:
- **Fixed parameters** are not adapted. During development of nnU-Net we identified a robust configuration (that is, certain architecture and training properties) that can 
simply be used all the time. This includes, for example, nnU-Net's loss function, (most of the) data augmentation strategy and learning rate.
- **Rule-based parameters** use the dataset fingerprint to adapt certain segmentation pipeline properties by following 
hard-coded heuristic rules. For example, the network topology (pooling behavior and depth of the network architecture) 
are adapted to the patch size; the patch size, network topology and batch size are optimized jointly given some GPU 
memory constraint. 
- **Empirical parameters** are essentially trial-and-error. For example the selection of the best U-net configuration 
for the given dataset (2D, 3D full resolution, 3D low resolution, 3D cascade) and the optimization of the postprocessing strategy.

## Getting Started

### 1. Install nnUNetv2

To get started, you'll need to install nnUNetv2 and its required dependencies. Please follow the [installation instructions](documentation/installation_instructions.md).

### 2. Prepare the Dataset

You can use the [TotalSegmentator dataset](documentation/TotalSegmentator_dataset.md) for segmentation tasks. Ensure the dataset is in the correct format as per the [dataset conversion guide](documentation/dataset_format.md).

### 3. Set Up Paths

Set up the necessary paths for data storage and configuration by following the [setting up paths guide](setting_up_paths.md).

### 4. Set Environment Variables

Ensure that the environment variables are set correctly for smooth execution. Detailed instructions can be found in the [set environment variables guide](set_environment_variables.md).

### 5. Preprocessing and Training

Once your dataset is ready, you can begin preprocessing and training your model by following the [usage instructions](documentation/how_to_use_nnunet.md).

### 6. Understand Configuration Parameters

You can customize the configuration of nnUNetv2 by editing the configuration files manually. For more details, check the [manually editing nnU-Net configurations guide](documentation/explanation_plans_files.md).




