This project implements a complete computer-vision pipeline for colony classification using:
YOLO for initial object detection
CVAT for ground-truth annotation refinement
PyTorch (ResNet-18) for classification
Transfer Learning from a public dataset to an in-house dataset
Visualization & Evaluation using confusion matrices and classification reports
--------------------------------------------------------------------------------------------
Overall workflow:

Raw Images
   ↓
YOLO Object Detection
   ↓
YOLO Output ZIP
   ↓
CVAT (Ground Truth Annotation)
   ↓
Ground Truth ZIP (YOLO format)
   ↓
Patch Extraction + Dataset Creation
   ↓
10-Class Public Model Training
   ↓
Transfer Learning + Fine-Tuning (4-Class In-House)
   ↓
Evaluation + Visualization
---------------------------------------------------------------------------------------------
Step 1: Object Detection using YOLO

* Raw plate images are first processed using YOLO.

* YOLO detects colonies and outputs:

   *Bounding boxes

   *Class labels

   *YOLO-format annotation files (.txt)

yolo_output/
 ├── images/
 ├── labels/
 └── yolo_predictions.zip



---------------------------------------------------------------------------------------------
Step 2: Ground Truth Creation using CVAT

*The YOLO output ZIP is uploaded into CVAT.

*In CVAT:

  *Bounding boxes are verified, corrected, or refined

  *Mis-detections are removed

  *Class labels are corrected if needed

*CVAT exports annotations in YOLO format.

CVAT Export ZIP:
ground_truth_dataset.zip
 ├── images/
 └── labels/

 
-----------------------------------------------------------------------------------------------
Step 3: Parsing CVAT Ground Truth

*The CVAT ZIP is parsed using:
   parse_cvat_yolo_zip_to_df(...)

*This creates a structured DataFrame containing:

  *image_path

  *image_name

  *bbox_x, bbox_y, bbox_width, bbox_height

  *label_name

  *label_idx

This DataFrame is the single source of truth used by all datasets.



------------------------------------------------------------------------------------------------
Step 4: Patch Extraction from Bounding Boxes

*Each bounding box is cropped from the original image.

*Key preprocessing steps:

   *Convert image to grayscale (L)

   *Crop bounding box region

   *Resize patch to fixed size (e.g., 100 × 100)

   *Convert back to RGB (for ResNet compatibility)

Implemented in: 

ColonyPatchDatasetCached / ColonyPatchDatasetFixed
This ensures consistent tensor shapes for batching.



-----------------------------------------------------------------------------------------------
Step 5: 10-Class Public Dataset Training

A ResNet-18 model is trained on a public 10-class dataset.

Best model is saved as:

resnet18_pretrained_10class.pt


-----------------------------------------------------------------------------------------------
Step 6: Transfer Learning to In-House Dataset

*The 10-class model is loaded.

*The final classification layer (fc) is replaced:
   model.fc = nn.Linear(in_features, num_inhouse_classes)


*Two-phase training:

  1. Phase 1 — Head-Only Training

      *Backbone frozen

      *Only classification head trained

  2. Phase 2 — Full Fine-Tuning

      *Entire network unfrozen

      *Fine-tuned on in-house data
Best model saved as:
    best_finetuned_on_newtest.pt



-----------------------------------------------------------------------------------------------
Step 7: Evaluation

Quantitative Evaluation

*Confusion Matrices (raw + normalized)

*Classification Reports (exported to Excel)

*Separate evaluation for:

  *10-class public model

  *4-class in-house fine-tuned model

Qualitative Visualization

*Per-image visualization:

  *Blue boxes → training patches

  *Green boxes → correctly classified test patches (TP)

  *Yellow boxes → misclassified test patches (FN)
  
This matches the professor’s requirement to visually show detections and misclassifications.
