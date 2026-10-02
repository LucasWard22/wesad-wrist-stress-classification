# Wrist Sensor Combinations for Participant-Independent Stress Classification in WESAD

## Summary

**Question.** Which wrist-sensor signals are needed to identify the stress condition in people the model has not seen?

**Method.** Recordings from 15 WESAD participants were divided into 60-second windows and classified as stress or non-stress using leave-one-subject-out cross-validation. Gradient boosting was applied to combinations of electrodermal activity (EDA), skin temperature, blood volume pulse (BVP) and movement.

**Main result.** EDA and temperature reached a mean balanced accuracy of 0.80, similar to all four wrist signals (0.79) and higher than EDA alone (0.72). With non-overlapping windows the corresponding scores were 0.82, 0.82 and 0.71.

**Main caveat.** Performance varied widely between participants, from chance level to near-perfect, and every stress window was missed for one participant. The analysis is exploratory, based on 15 people in a laboratory, and does not show that additional sensors are unnecessary.

**Relevance to digital health studies.** The results illustrate why sensor choices for remote monitoring should be evaluated on unseen participants, reported per participant, and tested against participant burden, which was not measured here (see Section 4.1).

## Abstract

Wearable stress classification relies on physiological signals that can vary substantially between individuals. This study examined whether combining wrist electrodermal activity (EDA) and skin temperature could achieve performance comparable to a larger set of sensor inputs. Recordings from 15 WESAD participants were divided into 60-second windows, and stress was classified against baseline and amusement using leave-one-subject-out cross-validation. EDA and temperature achieved a mean balanced accuracy of 0.801, compared with 0.716 for EDA alone and 0.786 for all wrist sensors. With non-overlapping windows, the corresponding scores were 0.823, 0.713 and 0.823. However, participant-level analysis revealed substantial differences in missed stress and false alarms. Baseline normalisation increased mean balanced accuracy from 0.789 to 0.814 on the remaining evaluation windows, without clear evidence of an overall improvement. EDA and temperature therefore provided a competitive combination under the methods examined, although the results do not establish sensor equivalence or reliable classification outside the laboratory.

## 1. Introduction

Wearable devices record several signals that may change during stress, including skin conductance, temperature, pulse and movement. Combining these signals could improve classification, but additional inputs may also introduce noise or capture features of an experimental task rather than stress itself. Their usefulness must therefore be evaluated in people who were not included in model training.

This project examined how selected wrist sensor combinations affected participant-independent classification in WESAD. The analysis focused on a simple feature set and a consistent evaluation scheme, allowing differences between sensor configurations to be assessed without changing the model settings. Participant-level errors, window overlap and baseline calibration were also examined to understand what the average results concealed.

## 2. Data and Methods

### 2.1 Recordings and feature extraction

The analysis used wrist recordings from 15 WESAD participants. EDA and skin temperature were sampled at 4 Hz, blood volume pulse (BVP) at 64 Hz, and three-axis acceleration at 32 Hz. Stress was treated as the positive class, with baseline and amusement combined into the non-stress class. Meditation, transitions and other labels were excluded.

Recordings were divided into 60-second windows with a 30-second step. Windows spanning more than one experimental condition were discarded, leaving 1,015 windows: 302 stress, 557 baseline and 156 amusement. Each sensor contributed its mean, standard deviation, minimum and maximum. EDA and temperature also contributed a slope calculated using the actual sample times in seconds. Acceleration was represented by the magnitude of its three axes, giving 18 features overall.

Window start and end times were retained so that predictions could be traced to the recordings. Checks confirmed that every window lasted 60 seconds and that timestamps, participant identifiers and labels were excluded from the predictors. The exported feature table contained no missing or non-finite values.

### 2.2 Participant-independent evaluation

Models were evaluated using leave-one-subject-out cross-validation. Each fold trained on 14 participants and tested on the remaining participant, repeated across all 15. Logistic regression scaling was fitted within the training pipeline, keeping the held-out participant separate from fitting.

A majority-class baseline, logistic regression and histogram gradient boosting were compared using all 18 features. The sensor comparison then used gradient boosting with balanced class weights and a fixed random seed. The same model settings were used for every sensor configuration, without a hyperparameter search.

Balanced accuracy was calculated as the average of stress sensitivity and non-stress specificity. Scores were first calculated for each participant and then averaged, giving each participant equal weight. Stress-class F1 and confusion counts were also examined. Paired Wilcoxon tests were exploratory, and no separate dataset was reserved to confirm the model and sensor choices.

### 2.3 Follow-up analyses

The sensor comparison was repeated using 60-second windows with a 60-second step, retaining 513 non-overlapping windows for both training and testing. This assessed whether the broad findings persisted under a different windowing scheme.

A separate calibration analysis normalised each participant’s EDA and temperature features using the mean and sample standard deviation of their first 19 baseline windows, covering approximately 10 minutes. These calibration windows and the next overlapping window were excluded from test scoring. Raw and normalised models were evaluated on the same remaining windows; calibration windows from training participants remained available for training.

## 3. Results

### 3.1 EDA and temperature were competitive with larger sensor combinations

Using all features, gradient boosting achieved a mean balanced accuracy of 0.786, compared with 0.756 for logistic regression and 0.500 for the majority-class baseline. Mean stress-class F1 was 0.670 for gradient boosting and 0.641 for logistic regression.

EDA and temperature achieved the highest observed mean balanced accuracy in the primary sensor comparison, at 0.801, with a standard deviation of 0.119 across participants. EDA alone scored 0.716. Adding acceleration to EDA and temperature gave 0.782, while all four sensor inputs gave 0.786. Acceleration, temperature and BVP alone scored 0.687, 0.661 and 0.558 respectively.

![WESAD wrist sensor comparison](https://raw.githubusercontent.com/LucasWard22/wesad-wrist-stress-classification/main/sensor_sets_boxplot.png)

**Figure 1. Participant-level balanced accuracy across wrist sensor configurations.** Each box summarises scores from the 15 held-out participants. The dashed line marks the majority-class baseline of 0.5. The overlapping distributions show that variation between participants was substantial relative to the differences between several configurations.

Adding temperature to EDA improved scores for 10 of 15 participants, with a mean increase of 8.5 percentage points and an unadjusted exploratory p-value of 0.016. Comparisons against all sensors and against EDA, temperature and acceleration showed no detectable difference, with p-values of 0.593 and 0.551 respectively. These results do not establish equivalence, and the tests require caution because multiple comparisons were examined and cross-validation folds shared training participants.

### 3.2 Participant-level errors revealed uneven reliability

For EDA and temperature, mean stress sensitivity was 0.742, specificity was 0.860, and stress-class F1 was 0.703. These averages concealed distinct failure patterns. All 20 stress windows were missed for S14. For S8, only 10 of 21 stress windows were detected, despite no false positives. For S17, 17 of 22 stress windows were detected, but 24 of 47 non-stress windows were incorrectly flagged as stress.

S14 was not difficult under every configuration: acceleration alone achieved a balanced accuracy of 0.865. The failure therefore concerned particular model inputs rather than showing that the participant’s experimental conditions were entirely indistinguishable.

![Wrist signals and experimental labels for S13](https://raw.githubusercontent.com/LucasWard22/wesad-wrist-stress-classification/main/signals_S13.png)

**Figure 2. Wrist signals and experimental conditions for S13.** EDA varied substantially across the recording, including a rise around the stress period, while temperature fell over part of the same interval. The EDA and temperature model achieved a balanced accuracy of 0.959. These patterns illustrate the information available to the model but do not establish that each signal change was caused by stress.

![Wrist signals and experimental labels for S14](https://raw.githubusercontent.com/LucasWard22/wesad-wrist-stress-classification/main/signals_S14.png)

**Figure 3. Wrist signals and experimental conditions for S14.** EDA occupied a much smaller range than in S13, while temperature changed gradually during the session. The EDA and temperature model predicted every retained window as non-stress. The plot alone cannot distinguish physiological differences from sensor contact or other explanations. In both signal figures, EDA is measured in microsiemens and temperature in degrees Celsius. Labels 1, 2 and 3 indicate baseline, stress and amusement; other labels are shown for context but excluded from classification.

### 3.3 The broad comparison persisted without overlapping windows

With a 60-second step, EDA and temperature achieved a mean balanced accuracy of 0.823, compared with 0.824 for EDA, temperature and acceleration and 0.823 for all sensors. EDA alone scored 0.713.

The exact ranking changed, but EDA and temperature remained competitive with the larger configurations. Because both the training sample and evaluated windows changed, the higher scores cannot be attributed solely to removing overlap. Non-overlapping windows may also remain temporally correlated.

### 3.4 Baseline calibration did not provide a consistent improvement

On the windows remaining after calibration exclusion, mean balanced accuracy increased from 0.789 with raw features to 0.814 with baseline-normalised features. Scores improved for 9 of 15 participants, but the exploratory paired test gave p = 0.561. Substantial improvements for some participants were accompanied by deterioration for others, providing no clear evidence of a consistent overall benefit.

## 4. Discussion

EDA and temperature provided a useful combination under the feature extraction and modelling approach tested here. Their observed advantage over EDA alone persisted across both windowing schemes, while the larger configurations achieved similar average performance. This supports further investigation of EDA and temperature, rather than a conclusion that additional sensors are unnecessary.

The participant-level results qualify the average performance. A model can achieve a reasonable balanced accuracy while missing many stress windows, and strong sensitivity can be accompanied by frequent false alarms. The complete failure to detect stress in S14 also shows why performance should be examined separately for each participant. S14 illustrates a further caution. Acceleration alone achieved a balanced accuracy of 0.865 for this participant, whereas the EDA and temperature model missed all 20 stress windows, and adding acceleration to EDA and temperature gave only 0.52. One possible explanation is that movement or posture differed between experimental conditions, so that acceleration reflected the task rather than a physiological stress response, while the combined model relied on patterns that did not generalise to this participant. The present analysis cannot test this explanation. A model that detects the task rather than the response would be of limited use outside a structured laboratory protocol, and movement-based features should therefore be interpreted with particular care.

The weak BVP-only result requires particular caution. BVP was represented by basic waveform summaries rather than pulse rate or pulse interval variability, so this analysis did not fully assess its physiological information. Acceleration magnitude likewise includes gravity and posture effects alongside movement. Classification may therefore reflect aspects of the experimental task as well as stress-related responses.

## 5. Limitations and Conclusion

The study includes only 15 participants recorded in a laboratory. Experimental labels identify conditions rather than continuous subjective stress, and movement, posture, condition order and signal drift may contribute to prediction. Signal artefacts were not systematically assessed, overlapping windows are correlated, and the model and sensor rankings were inspected within the same cross-validation results. The follow-up analyses should therefore be treated as exploratory.

Comfort, battery consumption, cost and adherence were not measured. Since all signals came from the same wrist device, selecting fewer inputs does not itself demonstrate reduced participant burden.

Within these limits, EDA and temperature achieved observed average performance comparable to the larger sensor combinations, while participant-level reliability remained uneven. More informative BVP features, explicit signal-quality assessment and evaluation on an independent dataset would be useful next steps. These extensions have not yet been carried out.

## 6. Reproduction and Data Acknowledgement

The analysis is available in [wesad_sensing_burden.ipynb](https://github.com/LucasWard22/wesad-wrist-stress-classification/blob/main/wesad_sensing_burden.ipynb). Open the notebook in Google Colab and run the cells in order. The opening cells download and extract WESAD; the final cells save the figures and result tables. Raw recordings are not included in this repository. Features, window timestamps, held-out predictions, participant metrics and full-precision comparison scores are included, with the recorded environment listed in [package_versions.csv](https://github.com/LucasWard22/wesad-wrist-stress-classification/blob/main/package_versions.csv).

WESAD was introduced by Philip Schmidt, Attila Reiss, Robert Dürichen, Claus Marberger and Kristof Van Laerhoven (2018), *Introducing WESAD, a Multimodal Dataset for Wearable Stress and Affect Detection*, Proceedings of the 2018 International Conference on Multimodal Interaction. [DOI: 10.1145/3242969.3242985](https://doi.org/10.1145/3242969.3242985).

The original study also compared sensor modalities. This project is an exploratory reanalysis using simpler features, with an emphasis on participant-level failures and robustness checks. Its balanced accuracy scores are not directly interchangeable with the original paper’s reported accuracy. Source data remain subject to the authors’ data-use conditions.
