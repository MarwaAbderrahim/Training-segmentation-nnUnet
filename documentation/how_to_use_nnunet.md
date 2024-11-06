## How to run nnU-Net on a new dataset
Given some dataset, nnU-Net fully automatically configures an entire segmentation pipeline that matches its properties.
nnU-Net covers the entire pipeline, from preprocessing to model configuration, model training, postprocessing
all the way to ensembling. After running nnU-Net, the trained model(s) can be applied to the test cases for inference.

### Experiment planning and preprocessing
Given a new dataset, nnU-Net will extract a dataset fingerprint (a set of dataset-specific properties such as
image sizes, voxel spacings, intensity information etc). This information is used to design three U-Net configurations. 
Each of these pipelines operates on its own preprocessed version of the dataset.

The easiest way to run fingerprint extraction, experiment planning and preprocessing is to use:

```bash
nnUNetv2_plan_and_preprocess -d DATASET_ID --verify_dataset_integrity
```

Where `DATASET_ID` is the dataset id (duh). We recommend `--verify_dataset_integrity` whenever it's the first time 
you run this command. This will check for some of the most common error sources!

You can also process several datasets at once by giving `-d 1 2 3 [...]`. If you already know what U-Net configuration 
you need you can also specify that with `-c 3d_fullres` (make sure to adapt -np in this case!). For more information 
about all the options available to you please run `nnUNetv2_plan_and_preprocess -h`.

nnUNetv2_plan_and_preprocess will create a new subfolder in your nnUNet_preprocessed folder named after the dataset. 
Once the command is completed there will be a dataset_fingerprint.json file as well as a nnUNetPlans.json file for you to look at 
(in case you are interested!). There will also be subfolders containing the preprocessed data for your UNet configurations.

[Optional]
If you prefer to keep things separate, you can also use `nnUNetv2_extract_fingerprint`, `nnUNetv2_plan_experiment` 
and `nnUNetv2_preprocess` (in that order). 

### Model training
#### Overview
You pick which configurations (2d, 3d_fullres, 3d_lowres, 3d_cascade_fullres) should be trained! In our case we used 3d_fullres

nnU-Net trains all configurations in a 5-fold cross-validation over the training cases. This is 1) needed so that 
nnU-Net can estimate the performance of each configuration and tell you which one should be used for your 
segmentation problem and 2) a natural way of obtaining a good model ensemble (average the output of these 5 models 
for prediction) to boost performance.

Training models is done with the `nnUNetv2_train` command. The general structure of the command is:
```bash
nnUNetv2_train DATASET_NAME_OR_ID UNET_CONFIGURATION FOLD [additional options, see -h]
```

UNET_CONFIGURATION is a string that identifies the requested U-Net configuration (defaults: 2d, 3d_fullres, 3d_lowres, 
3d_cascade_lowres). DATASET_NAME_OR_ID specifies what dataset should be trained on and FOLD specifies which fold of 
the 5-fold-cross-validation is trained.

nnU-Net stores a checkpoint every 50 epochs. If you need to continue a previous training, just add a `--c` to the
training command.

IMPORTANT: If you plan to use `nnUNetv2_find_best_configuration` (see below) add the `--npz` flag. This makes 
nnU-Net save the softmax outputs during the final validation. They are needed for that. Exported softmax
predictions are very large and therefore can take up a lot of disk space, which is why this is not enabled by default.
If you ran initially without the `--npz` flag but now require the softmax predictions, simply rerun the validation with:
```bash
nnUNetv2_train DATASET_NAME_OR_ID UNET_CONFIGURATION FOLD --val --npz
```

You can specify the device nnU-net should use by using `-device DEVICE`. DEVICE can only be cpu, cuda or mps. If 
you have multiple GPUs, please select the gpu id using `CUDA_VISIBLE_DEVICES=X nnUNetv2_train [...]` (requires device to be cuda).

See `nnUNetv2_train -h` for additional options.


### 3D full resolution U-Net
For FOLD in [0, 1, 2, 3, 4], run:
```bash
nnUNetv2_train DATASET_NAME_OR_ID 3d_fullres FOLD [--npz]
```

The trained models will be written to the nnUNet_results folder. Each training obtains an automatically generated
output folder name:

nnUNet_results/DatasetXXX_MYNAME/TRAINER_CLASS_NAME__PLANS_NAME__CONFIGURATION/FOLD

For Dataset002_Heart (from the MSD), for example, this looks like this:

    nnUNet_results/
    ├── Dataset002_Heart
        └── nnUNetTrainer__nnUNetPlans__3d_fullres
             ├── fold_0
             ├── fold_1
             ├── fold_2
             ├── fold_3
             ├── fold_4
             ├── dataset.json
             ├── dataset_fingerprint.json
             └── plans.json

In each model training output folder (each of the fold_x folder), the following files will be created:
- debug.json: Contains a summary of blueprint and inferred parameters used for training this model as well as a 
bunch of additional stuff. Not easy to read, but very useful for debugging ;-)
- checkpoint_best.pth: checkpoint files of the best model identified during training. Not used right now unless you 
explicitly tell nnU-Net to use it.
- checkpoint_final.pth: checkpoint file of the final model (after training has ended). This is what is used for both 
validation and inference.
- network_architecture.pdf (only if hiddenlayer is installed!): a pdf document with a figure of the network architecture in it.
- progress.png: Shows losses, pseudo dice, learning rate and epoch times ofer the course of the training. At the top is 
a plot of the training (blue) and validation (red) loss during training. Also shows an approximation of
  the dice (green) as well as a moving average of it (dotted green line). This approximation is the average Dice score 
  of the foreground classes. **It needs to be taken with a big (!) 
  grain of salt** because it is computed on randomly drawn patches from the validation
  data at the end of each epoch, and the aggregation of TP, FP and FN for the Dice computation treats the patches as if
  they all originate from the same volume ('global Dice'; we do not compute a Dice for each validation case and then
  average over all cases but pretend that there is only one validation case from which we sample patches). The reason for
  this is that the 'global Dice' is easy to compute during training and is still quite useful to evaluate whether a model
  is training at all or not. A proper validation takes way too long to be done each epoch. It is run at the end of the training.
- validation_raw: in this folder are the predicted validation cases after the training has finished. The summary.json file in here
  contains the validation metrics (a mean over all cases is provided at the start of the file). If `--npz` was set then 
the compressed softmax outputs (saved as .npz files) are in here as well. 

During training it is often useful to watch the progress. We therefore recommend that you have a look at the generated
progress.png when running the first training. It will be updated after each epoch.

Training times largely depend on the GPU. 

Example:


**Important: The first time a training is run nnU-Net will extract the preprocessed data into uncompressed numpy 
arrays for speed reasons! This operation must be completed before starting more than one training of the same 
configuration! Wait with starting subsequent folds until the first training is using the GPU! Depending on the 
dataset size and your System this should oly take a couple of minutes at most.**

### Automatically determine the best configuration
Once the desired configurations were trained (full cross-validation) you can tell nnU-Net to automatically identify 
the best combination for you:

```commandline
nnUNetv2_find_best_configuration DATASET_NAME_OR_ID -c CONFIGURATIONS 
```

`CONFIGURATIONS` hereby is the list of configurations you would like to explore. Per default, ensembling is enabled 
meaning that nnU-Net will generate all possible combinations of ensembles (2 configurations per ensemble). This requires 
the .npz files containing the predicted probabilities of the validation set to be present (use `nnUNetv2_train` with 
`--npz` flag, see above). You can disable ensembling by setting the `--disable_ensembling` flag.

See `nnUNetv2_find_best_configuration -h` for more options.

nnUNetv2_find_best_configuration will also automatically determine the postprocessing that should be used. 
Postprocessing in nnU-Net only considers the removal of all but the largest component in the prediction (once for 
foreground vs background and once for each label/region).
