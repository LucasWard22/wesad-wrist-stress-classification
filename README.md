Comparing wrist sensor combinations for stress classification
I built this project to explore a practical question: how much information do different wrist sensors add when classifying stress in someone the model has not seen before?
Using the WESAD dataset, I compared electrodermal activity (EDA), skin temperature, blood volume pulse (BVP) and acceleration, both individually and in selected combinations. I also examined participant-level failures, repeated the comparison with non-overlapping windows, and tested whether a short baseline calibration improved performance.
The main finding was that EDA and temperature performed similarly to the larger sensor combinations on average. However, performance varied considerably between participants, and calibration did not show a clear overall benefit.
Data and methods
The analysis uses wrist recordings from 15 WESAD participants. Stress is treated as the positive class, with baseline and amusement combined into the non-stress class. Meditation, transitions and other labels are excluded.
I divided the recordings into 60-second windows with a 30-second step. Windows crossing condition boundaries were excluded, leaving 1,015 windows: 302 stress, 557 baseline and 156 amusement.
For each sensor, I extracted the mean, standard deviation, minimum and maximum. EDA and temperature also included a slope feature, giving 18 features in total. The three acceleration axes were combined into a magnitude signal. Window timestamps and participant identifiers were retained for checking results but excluded from the predictors.
Models were evaluated using leave-one-subject-out cross-validation: train on 14 participants and test on the remaining participant, repeated for all 15. Logistic regression scaling was fitted on the training data within each fold.
I compared a majority-class baseline, logistic regression and histogram gradient boosting. The sensor comparison used the same gradient boosting settings for every configuration, with balanced class weights and a fixed random seed. No hyperparameter search was performed.
Balanced accuracy is the average of stress sensitivity and non-stress specificity. Scores were calculated separately for each held-out participant and then averaged, giving each participant equal weight.
Results
Model comparison
Using all 18 features:
Model	Mean balanced accuracy	SD across participants	Mean stress F1
Majority-class baseline	0.500	0.000	0.000
Logistic regression	0.756	0.146	0.641
Gradient boosting	0.786	0.143	0.670


Sensor comparison
Sensor inputs	Mean balanced accuracy	SD across participants
EDA + temperature	0.801	0.119
All wrist sensors	0.786	0.143
EDA + temperature + acceleration	0.782	0.136
EDA only	0.716	0.120
Acceleration only	0.687	0.097
Temperature only	0.661	0.226
BVP only	0.558	0.100


 
Each box summarises scores from the 15 held-out participants. The dashed line shows the majority-class baseline at 0.5. The spread represents variation between participants, rather than uncertainty around the mean.
Adding temperature to EDA increased mean balanced accuracy by 8.5 percentage points and improved scores for 10 of 15 participants. The exploratory paired Wilcoxon test gave an unadjusted p-value of 0.016. Comparisons against the larger sensor combinations showed no detectable difference (p = 0.593 for all sensors; p = 0.551 for EDA + temperature + acceleration).
These tests are exploratory. They do not prove that configurations are equivalent, and their interpretation is limited by the small sample, multiple comparisons and shared training data across cross-validation folds.
Where the model struggled
For EDA + temperature, mean stress sensitivity was 0.742, specificity was 0.860, and stress F1 was 0.703.
The average score hid different failure patterns:
- S14: all 20 stress windows were missed.
- S8: 10 of 21 stress windows were detected, with no false positives.
- S17: 17 of 22 stress windows were detected, but 24 of 47 non-stress windows were incorrectly flagged as stress.
S14 was not difficult under every configuration: acceleration alone scored 0.865. This makes it important to distinguish a model's failure from a participant being unclassifiable.
The signal plots below show the much smaller EDA range in S14 compared with S13. They do not establish whether this reflects physiological differences, sensor contact or another cause.
  
EDA is shown in microsiemens and temperature in degrees Celsius. Condition labels 1, 2 and 3 correspond to baseline, stress and amusement. Other labels appear in these full-session plots but are excluded from classification.
Non-overlapping window check
I repeated the sensor comparison using a 60-second step, retaining 513 non-overlapping windows.
Sensor inputs	30-second step	60-second step
EDA only	0.716	0.713
EDA + temperature	0.801	0.823
EDA + temperature + acceleration	0.782	0.824
All wrist sensors	0.786	0.823


EDA + temperature remained competitive, although the exact ranking changed. This check changes both the training sample and the evaluated windows, so the score changes cannot be attributed solely to removing overlap. Non-overlapping windows can still be temporally correlated.
Baseline calibration
I also tested normalisation using each participant's first 19 baseline windows, covering approximately 10 minutes. The calibration windows and the next overlapping window were excluded from test scoring. Raw and normalised models were evaluated on the same remaining windows.
Mean balanced accuracy increased from 0.789 to 0.814, with improvement for 9 of 15 participants. The exploratory paired test gave p = 0.561. Some participants improved substantially while others became worse, so this analysis does not establish a consistent benefit from calibration.
What I take from this
EDA + temperature is a reasonable configuration to investigate further: it performed well relative to the larger combinations in both windowing analyses. The more important issue is whether the model works reliably for each person. A useful average score can still conceal missed stress or frequent false alarms.
For a future wearable study, I would assess signal quality and participant-level sensitivity and specificity before deciding which inputs to retain. These results alone would not justify removing a sensor or adding a calibration period.
Limitations
- The dataset contains only 15 participants recorded under laboratory conditions. Results do not establish performance in daily life or a clinical population.
- Labels identify experimental conditions; they are not continuous measurements of each participant's subjective stress.
- Movement, posture, condition order and signal drift may contribute to classification alongside stress-related changes.
- BVP features are basic waveform summaries. Pulse rate and pulse interval variability were not extracted, so the low BVP score does not demonstrate that cardiac information is unhelpful.
- Acceleration magnitude contains gravity and posture effects as well as movement. Signal artefacts were not systematically filtered or assessed.
- The sensor ranking and model choice were inspected on the same cross-validation results. There is no independent dataset confirming the selected configuration.
- Overlapping windows are correlated and do not represent independent stress events.
- Battery use, comfort, cost and adherence were not measured. Using fewer sensor inputs does not itself demonstrate reduced participant burden.
Reproducing the analysis
1. Open [wesad_sensing_burden.ipynb](wesad_sensing_burden.ipynb) in Google Colab.
2. Run the dataset download and extraction cells, or obtain WESAD from the authors and place its participant folders inside WESAD/ in the runtime working directory. Only load pickle files obtained from the trusted dataset source.
3. Run the notebook from top to bottom. It extracts features, evaluates models, compares sensors and runs the follow-up analyses.
4. The final cells save figures and CSVs to figures/ and results/, then download them as a ZIP.
The uploaded figures and CSVs are currently in the repository's top folder. Raw WESAD recordings are not included. [package_versions.csv](package_versions.csv) records the environment used for the saved results; dependency changes may affect reproduction.
Files
File	Contents
[wesad_sensing_burden.ipynb](wesad_sensing_burden.ipynb)	Analysis code, outputs and interpretation
[wesad_wrist_features.csv](wesad_wrist_features.csv)	Extracted features, labels and window timestamps
[model_comparison.csv](model_comparison.csv)	Per-participant scores for the three models
[sensor_set_results.csv](sensor_set_results.csv)	Per-participant scores for seven sensor configurations
[eda_temp_participant_metrics.csv](eda_temp_participant_metrics.csv)	Sensitivity, specificity, F1 and confusion counts
[eda_temp_window_predictions.csv](eda_temp_window_predictions.csv)	Held-out predictions linked to window timestamps
[nonoverlap_sensor_results.csv](nonoverlap_sensor_results.csv)	Scores using non-overlapping windows
[window_overlap_comparison.csv](window_overlap_comparison.csv)	Summary of the two windowing analyses
[calibration_comparison.csv](calibration_comparison.csv)	Raw and baseline-normalised scores
[package_versions.csv](package_versions.csv)	Recorded Python and package versions


Dataset acknowledgement
This project uses WESAD, introduced by Philip Schmidt, Attila Reiss, Robert Dürichen, Claus Marberger and Kristof Van Laerhoven (2018), Introducing WESAD, a Multimodal Dataset for Wearable Stress and Affect Detection, ICMI '18. Original paper.
The original study also compared sensor modalities and device locations. This project is an exploratory reanalysis using a simpler feature set, with a focus on participant-level errors and follow-up robustness checks. Its balanced accuracy scores are not directly interchangeable with the original paper's reported accuracy.
