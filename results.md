Final Experiment Results
Evaluation protocol
Training: 100 patients. Validation: 20 patients, with five fixed cases used for checkpoint monitoring. Final testing: 30 held-out patients. Selected checkpoint: corrected v2 model, epoch 30. Threshold: 0.5; sliding-window size: 64 x 64 x 64; overlap: 0.25.

Final test summary
Region	Mean Dice	SD	Median	Minimum	Maximum	Volume MAE (mL)
WT	0.5868	0.1896	0.5628	0.2099	0.8665	91.2326
TC	0.6393	0.1810	0.6603	0.2299	0.8702	46.0763
ET	0.4516	0.2861	0.5354	0.0147	0.8430	21.9789
Overall mean Dice: 0.5592. Each region was evaluated on 30 patients; no ground-truth regions were empty.

Validation summary
WT: 0.5584; TC: 0.6115; ET: 0.3676 mean Dice across 20 validation patients.

Main finding
The model exceeded the immediate project target of 0.50 mean WT Dice on held-out testing. Tumor core achieved the strongest overlap. Enhancing tumor remained challenging.

Important limitation
WT predictions systematically overestimated volume in the scatter plot. The 91.23 mL WT mean absolute error means that volumetric predictions are exploratory, not clinically reliable. Dice values are segmentation-overlap scores, not diagnostic accuracy.

Visual artifacts to upload
outputs/final_test_dashboard.png: Dice bars, patient-level distributions, volume MAE, and WT volume agreement.

outputs/test_prediction_example.png: MRI, ground-truth WT, and predicted WT.

outputs/test_per_patient.csv: exported individual patient results.

patient_split.json: reproducible patient assignments.

The test set has now been inspected. Further model development should be documented separately and assessed on new held-out data where possible. RL was not implemented.
