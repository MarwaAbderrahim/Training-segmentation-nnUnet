# nnU-Net dataset format
The only way to bring your data into nnU-Net is by storing it in a specific format.

Datasets consist of three components: raw images, corresponding segmentation maps and a dataset.json file specifying 
some metadata. 

## What do training cases look like?
Each training case is associated with an identifier = a unique name for that case. This identifier is used by nnU-Net to 
connect images with the correct segmentation.

A training case consists of images and their corresponding segmentation. 

**Images** Input channels are identified by nnU-Net by their FILE_ENDING: a four-digit integer at the end 
of the filename. Image files must therefore follow the following naming convention: {CASE_IDENTIFIER}_{XXXX}.{FILE_ENDING}. 
Hereby, XXXX is the 4-digit modality/channel identifier (should be unique for each dataset, e.g., “0000” for the first dataset, “0001” for 
the second, …) and FILE_ENDING is the file extension used by your image format (.nii.gz, ...). See below for concrete examples.
The dataset.json file connects channel names with the channel identifiers in the 'channel_names' key (see below for details).

**Segmentations** must share the same geometry with their corresponding images (same shape etc.). Segmentations are 
integer maps with each value representing a semantic class. The background must be 0. If there is no background, then 
do not use the label 0 for something else! Integer values of your semantic classes must be consecutive (0, 1, 2, 3, 
...). Of course, not all labels have to be present in each training case. Segmentations are saved as {CASE_IDENTIFER}.{FILE_ENDING} .

Within a training case, all image geometries (input channels, corresponding segmentation) must match. Between training 
cases, they can of course differ. nnU-Net takes care of that.

## Supported file formats
nnU-Net expects the same file format for images and segmentations! These will also be used for inference. 

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
integer, and a dataset name (which you can freely choose): For example, Dataset005_organs has 'organs' as dataset name and 
the dataset id is 5. Datasets are stored in the `nnUNet_raw` folder like this:

    nnUNet_raw/
    ├── Dataset001_organs
    ├── Dataset002_Heart
    ├── Dataset003_muscles
    ├── Dataset004_ribs
    ├── Dataset005_vertebare
    ├── ...

Within each dataset folder, the following structure is expected:

    Dataset001_organs/
    ├── dataset.json
    ├── imagesTr
    ├── imagesTs  # optional
    └── labelsTr


- **imagesTr** contains the images belonging to the training cases. nnU-Net will perform pipeline configuration, training with 
cross-validation, as well as finding postprocessing and the best ensemble using this data. 
- **imagesTs** (optional) contains the images that belong to the test cases. nnU-Net does not use them! This could just 
be a convenient location for you to store these images. Remnant of the Medical Segmentation Decathlon folder structure.
- **labelsTr** contains the images with the ground truth segmentation maps for the training cases. 
- **dataset.json** contains metadata of the dataset.

The scheme introduced [above](#what-do-training-cases-look-like) results in the following folder structure. Given 
is an example for the first Dataset: organs. Note that the imagesTs folder is optional and does not have to be present.

    nnUNet_raw/Dataset001_organs/
    ├── dataset.json
    ├── imagesTr
    │   ├── case_001_0000.nii.gz
    │   ├── case_001_0000.nii.gz
    │   ├── case_001_0000.nii.gz
    │   ├── case_001_0000.nii.gz
    │   ├── ...
    ├── imagesTs
    │   ├── case_389_0000.nii.gz
    │   ├── case_390_0000.nii.gz
    │   ├── case_391_0000.nii.gz
    │   ├── case_392_0000.nii.gz
    │   ├── case_393_0000.nii.gz
    │   ├── ...
    └── labelsTr
        ├── case_001.nii.gz
        ├── case_002.nii.gz
        ├── ...


Remember: For each training case, all images must have the same geometry to ensure that their pixel arrays are aligned. Also 
make sure that all your data is co-registered!

## dataset.json
The dataset.json contains metadata that nnU-Net needs for training.

Here is what the dataset.json should look like at the example of the Dataset005_Prostate from the MSD:

    { 
     "channel_names": {  # formerly modalities
       "0": "CT", 
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

## How to use Totalsegmentator dataset
See [TotalSegmentator_dataset.md](TotalSegmentator_dataset.md)
