3D U-Net Brain Tumor Segmentation and Volume Estimation
A coursework deep-learning project for segmenting brain-tumor regions from multi-modal 3D MRI using PyTorch and MONAI. The pipeline loads MRI volumes, trains a compact 3D U-Net, predicts tumor masks, and estimates region volumes in milliliters.

This is a research/educational prototype, not a diagnostic or clinically validated measurement system.

Dataset
Medical Segmentation Decathlon Task01_BrainTumour, a BraTS-derived dataset with 484 labeled training cases. Each case contains four MRI modalities in one 4D NIfTI image and a corresponding 3D segmentation mask. MRI modality ordering and label definitions should be verified against the dataset metadata before adapting the code to another release.

Dataset information: http://medicaldecathlon.com/
MONAI tutorials: https://github.com/Project-MONAI/tutorials

The dataset and trained weights are not included in this repository. Follow the dataset provider's access and usage requirements.

Patient-level split
Split	Patients	Purpose
Training	100	Weight optimization
Validation	20	Development and validation evaluation
Test	30	Held-out final evaluation
Unused	334	Not used in this experiment
The sorted list of matched image-label cases was shuffled with Python random seed 42. Five fixed validation patients were used for checkpoint monitoring. The selected model was then evaluated on all 20 validation patients and finally on the 30 test patients. Patient IDs are saved in the split JSON.

Target regions
Whole Tumor (WT): all non-background tumor labels.

Tumor Core (TC): necrotic/non-enhancing core and enhancing tumor.

Enhancing Tumor (ET): enhancing tumor only.

The model uses three overlapping binary target channels, not three mutually exclusive classes.

Model and training
Framework: PyTorch and MONAI.

Hardware: Google Colab, NVIDIA Tesla T4.

Model: compact 3D U-Net; 1,204,658 trainable parameters.

Input/output channels: 4 / 3.

Feature channels: (8, 16, 32, 64, 128).

Downsampling strides: (2, 2, 2, 2).

Residual units per stage: 2; instance normalization.

Training patch size: 64 x 64 x 64 voxels.

Sampling: four random patches per patient, positive/negative sampling weights 3:1.

Source-patient batch size: 1; four patches collated per iteration.

Preprocessing: crop foreground, normalize each complete cropped MRI modality over nonzero voxels, then sample training patches.

Augmentation: flips, 90-degree rotations, intensity scaling and shifting.

Loss: DiceLoss(sigmoid=True, squared_pred=True) + 0.5 * BCEWithLogitsLoss.

Optimizer: AdamW; initial learning rate 1e-4; weight decay 1e-5.

Schedule: cosine annealing to 1e-6 over 30 epochs.

Mixed-precision training.

Validation monitoring every three epochs; checkpoint selection by mean WT Dice on five fixed validation patients.

Best checkpoint: epoch 30.

An earlier 12-patient prototype and an initial 100-patient experiment were development baselines. The reported final results below are from the corrected full-volume-normalization experiment.

Inference and metrics
Full-volume prediction uses overlapping 64-cubed sliding windows, overlap 0.25, and a fixed sigmoid threshold of 0.5 per region. Metrics are computed from these binary channels without a hierarchy-enforcement postprocessing step.

Dice = 2 * |prediction intersection ground truth| / (|prediction| + |ground truth|).

Volume (mL) = labeled voxel count * product of voxel spacings in mm / 1000.

Reported Dice values are patient-level means; standard deviations describe variation between patients, not confidence intervals. Both-empty masks receive Dice 1, and one-empty masks receive Dice 0. There were no empty ground-truth target regions in the final 30-patient evaluation.

Validation results: 20 patients
Region	Mean Dice	Standard deviation
WT	0.5584	0.1902
TC	0.6115	0.1497
ET	0.3676	0.2682
The other 15 validation patients, not used for checkpoint monitoring, achieved mean WT Dice 0.5590.

Final held-out test results: 30 patients
Region	Mean Dice	SD	Median Dice	Volume MAE (mL)
WT	0.5868	0.1896	0.5628	91.2326
TC	0.6393	0.1810	0.6603	46.0763
ET	0.4516	0.2861	0.5354	21.9789
Overall mean Dice across patients and regions: 0.5592.

The WT result exceeded the project's immediate 0.50 Dice target. This is a segmentation-overlap result, not classification accuracy or diagnostic accuracy.

Interpretation and limitations
Tumor core had the strongest mean test overlap. Enhancing tumor had the lowest and most variable overlap. Whole-tumor volume predictions showed systematic overestimation in the test scatter plot, with MAE 91.23 mL. Therefore the volume calculation pipeline works computationally, but its predicted volumes are not clinically reliable.

Other limitations include a small training subset, limited patch context, no external clinical validation, and independently thresholded channels that do not guarantee ET is contained within TC and TC within WT. The test set has now been inspected and should not be reused as an untouched benchmark for further tuning.

Running the project
Open the main training notebook from this repository in Google Colab.

Select a CUDA GPU runtime; the experiment used a Tesla T4.

Install the notebook dependencies and obtain Task01_BrainTumour.

Run setup, patient split, corrected transforms, loaders, model, and evaluation-function cells in order.

Run the 30-epoch training cell and back up the checkpoint and split to persistent storage.

Load the selected checkpoint, evaluate validation data, then run the fixed final test evaluation.

Export patient-level metrics, summary CSVs, dashboard, and MRI overlays.

Do not rerun the model-initialization cell immediately before evaluation unless the checkpoint is loaded afterward. Colab runtime files and Python variables are temporary.

Environment used
Python: record from the final notebook environment.

PyTorch: 2.11.0+cu128.

MONAI: 1.6.1.

Supporting libraries: NumPy, pandas, NiBabel, Matplotlib.

CUDA-enabled PyTorch installation depends on the runtime. The accompanying requirements list is a dependency list, not a complete environment lockfile.

Outputs
See RESULTS.md and test_summary.csv for the final results. Add the exported test_per_patient.csv, final_test_dashboard.png, and test_prediction_example.png to the repository's outputs folder. Preserve the patient_split.json alongside the experiment metadata.

Reinforcement learning status
No RL agent was implemented or evaluated. RL-guided patch sampling is proposed future work, not a demonstrated contribution.

Future work
Train with more patients and larger-context patches; investigate false positives and volume bias; explore hierarchy-consistent outputs; and evaluate improvements using a new held-out or external dataset.
