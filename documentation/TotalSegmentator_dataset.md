## Training nnU-Net v2 with TotalSegmentator Dataset
For our training of nnU-Net v2, we utilized the open-source TotalSegmentator dataset, available here. This dataset contains 1,217 CT images, each annotated with 124 ROI (Region of Interest) masks.

To create six distinct models based on specific regions of interest, we organized the data into the following classes:

291: "class_map_part_organs"
292: "class_map_part_vertebrae"
293: "class_map_part_cardiac"
294: "class_map_part_muscles"
295: "class_map_part_ribs"
253: "class_map_part_cardiac_v1"

# Data Preparation
Before starting the training process, we combined masks to create multi-label masks using [../combine_masks.py]. Each model is dedicated to a specific ROI category, with the following regions of interest (ROIs) included in each category:

### Organs
1: spleen, 2: kidney_right, 3: kidney_left, 4: gallbladder, 5: liver, 6: stomach,
7: pancreas, 8: adrenal_gland_right, 9: adrenal_gland_left, 10: lung_upper_lobe_left,
11: lung_lower_lobe_left, 12: lung_upper_lobe_right, 13: lung_middle_lobe_right,
14: lung_lower_lobe_right, 15: esophagus, 16: trachea, 17: thyroid_gland,
18: small_bowel, 19: duodenum, 20: colon, 21: urinary_bladder, 22: prostate,
23: kidney_cyst_left, 24: kidney_cyst_right

### Vertebrae
1: sacrum, 2: vertebrae_S1, 3: vertebrae_L5, 4: vertebrae_L4, 5: vertebrae_L3,
6: vertebrae_L2, 7: vertebrae_L1, 8: vertebrae_T12, 9: vertebrae_T11, 10: vertebrae_T10,
11: vertebrae_T9, 12: vertebrae_T8, 13: vertebrae_T7, 14: vertebrae_T6, 15: vertebrae_T5,
16: vertebrae_T4, 17: vertebrae_T3, 18: vertebrae_T2, 19: vertebrae_T1, 20: vertebrae_C7,
21: vertebrae_C6, 22: vertebrae_C5, 23: vertebrae_C4, 24: vertebrae_C3, 25: vertebrae_C2,
26: vertebrae_C1

### Cardiac
1: esophagus, 2: trachea, 3: heart_myocardium, 4: heart_atrium_left, 5: heart_ventricle_left,
6: heart_atrium_right, 7: heart_ventricle_right, 8: pulmonary_artery, 9: brain,
10: iliac_artery_left, 11: iliac_artery_right, 12: iliac_vena_left, 13: iliac_vena_right,
14: small_bowel, 15: duodenum, 16: colon, 17: urinary_bladder, 18: face

### Muscles
1: humerus_left, 2: humerus_right, 3: scapula_left, 4: scapula_right, 5: clavicula_left,
6: clavicula_right, 7: femur_left, 8: femur_right, 9: hip_left, 10: hip_right, 11: spinal_cord,
12: gluteus_maximus_left, 13: gluteus_maximus_right, 14: gluteus_medius_left,
15: gluteus_medius_right, 16: gluteus_minimus_left, 17: gluteus_minimus_right,
18: autochthon_left, 19: autochthon_right, 20: iliopsoas_left, 21: iliopsoas_right,
22: brain, 23: skull

### Ribs
1: rib_left_1, 2: rib_left_2, 3: rib_left_3, 4: rib_left_4, 5: rib_left_5, 6: rib_left_6,
7: rib_left_7, 8: rib_left_8, 9: rib_left_9, 10: rib_left_10, 11: rib_left_11, 12: rib_left_12,
13: rib_right_1, 14: rib_right_2, 15: rib_right_3, 16: rib_right_4, 17: rib_right_5,
18: rib_right_6, 19: rib_right_7, 20: rib_right_8, 21: rib_right_9, 22: rib_right_10,
23: rib_right_11, 24: rib_right_12, 25: sternum, 26: costal_cartilages

## Dataset Structure
After combining the masks, we structured the dataset according to the format described in the dataset_format.md.

