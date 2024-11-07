# nnU-Net dataset format
The only way to bring your data into nnU-Net is by storing it in a specific format.

Datasets consist of three components: raw images, corresponding segmentation maps and a dataset.json file specifying 
some metadata. 

## What do training cases look like?
Each training case is associated with an identifier = a unique name for that case. This identifier is used by nnU-Net to 
connect images with the correct segmentation.

A training case consists of images and their corresponding segmentation. 

**Images** is plural because nnU-Net supports arbitrarily many input channels. In order to be as flexible as possible, 
nnU-net requires each input channel to be stored in a separate image (with the sole exception being RGB natural 
images). So these images could for example be a T1 and a T2 MRI (or whatever else you want). The different input 
channels MUST have the same geometry (same shape, spacing (if applicable) etc.) and
must be co-registered (if applicable). Input channels are identified by nnU-Net by their FILE_ENDING: a four-digit integer at the end 
of the filename. Image files must therefore follow the following naming convention: {CASE_IDENTIFIER}_{XXXX}.{FILE_ENDING}. 
Hereby, XXXX is the 4-digit modality/channel identifier (should be unique for each modality/chanel, e.g., “0000” for T1, “0001” for 
T2 MRI, …) and FILE_ENDING is the file extension used by your image format (.png, .nii.gz, ...). See below for concrete examples.
The dataset.json file connects channel names with the channel identifiers in the 'channel_names' key (see below for details).

**Segmentations** must share the same geometry with their corresponding images (same shape etc.). Segmentations are 
integer maps with each value representing a semantic class. The background must be 0. If there is no background, then 
do not use the label 0 for something else! Integer values of your semantic classes must be consecutive (0, 1, 2, 3, 
...). Of course, not all labels have to be present in each training case. Segmentations are saved as {CASE_IDENTIFER}.{FILE_ENDING} .

Within a training case, all image geometries (input channels, corresponding segmentation) must match. Between training 
cases, they can of course differ. nnU-Net takes care of that.

## Supported file formats
nnU-Net expects the same file format for images and segmentations! These will also be used for inference. For now, it 
is thus not possible to train .png and then run inference on .jpg.

One big change in nnU-Net V2 is the support of multiple input file types. Gone are the days of converting everything to .nii.gz!

Note that internally (for storing and accessing preprocessed images) nnU-Net will use its own file format, irrespective 
of what the raw data was provided in! This is for performance reasons.

By default, the following file formats are supported:
- NaturalImage2DIO: .png, .bmp, .tif
- NibabelIO: .nii.gz, .nrrd, .mha
- NibabelIOWithReorient: .nii.gz, .nrrd, .mha. This reader will reorient images to RAS!
- SimpleITKIO: .nii.gz, .nrrd, .mha
- Tiff3DIO: .tif, .tiff. 3D tif images! Since TIF does not have a standardized way of storing spacing information, 

## Dataset folder structure
Datasets must be located in the `nnUNet_raw` folder (which you either define when installing nnU-Net or export/set every 
time you intend to run nnU-Net commands!).
Each segmentation dataset is stored as a separate 'Dataset'. Datasets are associated with a dataset ID, a three digit 
integer, and a dataset name (which you can freely choose): For example, Dataset005_Prostate has 'Prostate' as dataset name and 
the dataset id is 5. Datasets are stored in the `nnUNet_raw` folder like this:

    nnUNet_raw/
    ├── Dataset001_BrainTumour
    ├── Dataset002_Heart
    ├── Dataset003_Liver
    ├── Dataset004_Hippocampus
    ├── Dataset005_Prostate
    ├── ...

Within each dataset folder, the following structure is expected:

    Dataset001_BrainTumour/
    ├── dataset.json
    ├── imagesTr
    ├── imagesTs  # optional
    └── labelsTr


When adding your custom dataset, take a look at the [dataset_conversion](../nnunetv2/dataset_conversion) folder and 
pick an id that is not already taken. IDs 001-010 are for the Medical Segmentation Decathlon.

- **imagesTr** contains the images belonging to the training cases. nnU-Net will perform pipeline configuration, training with 
cross-validation, as well as finding postprocessing and the best ensemble using this data. 
- **imagesTs** (optional) contains the images that belong to the test cases. nnU-Net does not use them! This could just 
be a convenient location for you to store these images. Remnant of the Medical Segmentation Decathlon folder structure.
- **labelsTr** contains the images with the ground truth segmentation maps for the training cases. 
- **dataset.json** contains metadata of the dataset.

The scheme introduced [above](#what-do-training-cases-look-like) results in the following folder structure. Given 
is an example for the first Dataset of the MSD: BrainTumour. This dataset hat four input channels: FLAIR (0000), 
T1w (0001), T1gd (0002) and T2w (0003). Note that the imagesTs folder is optional and does not have to be present.

    nnUNet_raw/Dataset001_BrainTumour/
    ├── dataset.json
    ├── imagesTr
    │   ├── BRATS_001_0000.nii.gz
    │   ├── BRATS_001_0001.nii.gz
    │   ├── BRATS_001_0002.nii.gz
    │   ├── BRATS_001_0003.nii.gz
    │   ├── BRATS_002_0000.nii.gz
    │   ├── BRATS_002_0001.nii.gz
    │   ├── BRATS_002_0002.nii.gz
    │   ├── BRATS_002_0003.nii.gz
    │   ├── ...
    ├── imagesTs
    │   ├── BRATS_485_0000.nii.gz
    │   ├── BRATS_485_0001.nii.gz
    │   ├── BRATS_485_0002.nii.gz
    │   ├── BRATS_485_0003.nii.gz
    │   ├── BRATS_486_0000.nii.gz
    │   ├── BRATS_486_0001.nii.gz
    │   ├── BRATS_486_0002.nii.gz
    │   ├── BRATS_486_0003.nii.gz
    │   ├── ...
    └── labelsTr
        ├── BRATS_001.nii.gz
        ├── BRATS_002.nii.gz
        ├── ...

Here is another example of the second dataset of the MSD, which has only one input channel:

    nnUNet_raw/Dataset002_Heart/
    ├── dataset.json
    ├── imagesTr
    │   ├── la_003_0000.nii.gz
    │   ├── la_004_0000.nii.gz
    │   ├── ...
    ├── imagesTs
    │   ├── la_001_0000.nii.gz
    │   ├── la_002_0000.nii.gz
    │   ├── ...
    └── labelsTr
        ├── la_003.nii.gz
        ├── la_004.nii.gz
        ├── ...

Remember: For each training case, all images must have the same geometry to ensure that their pixel arrays are aligned. Also 
make sure that all your data is co-registered!

See also [dataset format inference](dataset_format_inference.md)!!

## dataset.json
The dataset.json contains metadata that nnU-Net needs for training. We have greatly reduced the number of required 
fields since version 1!

Here is what the dataset.json should look like at the example of the Dataset005_Prostate from the MSD:

    { 
     "channel_names": {  # formerly modalities
       "0": "T2", 
       "1": "ADC"
     }, 
     "labels": {  # THIS IS DIFFERENT NOW!
       "background": 0,
       "PZ": 1,
       "TZ": 2
     }, 
     "numTraining": 32, 
     "file_ending": ".nii.gz"
     "overwrite_image_reader_writer": "SimpleITKIO"  # optional! If not provided nnU-Net will automatically determine the ReaderWriter
     }

The channel_names determine the normalization used by nnU-Net. If a channel is marked as 'CT', then a global 
normalization based on the intensities in the foreground pixels will be used. If it is something else, per-channel 
z-scoring will be used. 

There is a utility with which you can generate the dataset.json automatically. You can find it 
[here](../nnunetv2/dataset_conversion/generate_dataset_json.py). 
See our examples in [dataset_conversion](../nnunetv2/dataset_conversion) for how to use it. And read its documentation!

## How to use Totalsegmentator dataset datasets
See [TotalSegmentator_dataset.md
.md](TotalSegmentator_dataset.md
.md)
