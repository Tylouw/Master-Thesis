# All 6 results highlighted with their most important findings
## 1. Best Channel Study
The best channel study leads to three main conclusions. First, F/T is the best single channel group, but the best overall results are achieved when it is combined with additional modalities. Successful insertion classification is therefore not encoded in one signal group alone. 

Second, current and TCP pose provide substantial additional information, even though they are more indirect than F/T. 

**Third, the best-performing configuration combines complementary information: contact interaction through F/T, actuator load through current consumption, robot state through TCP pose, and motion dynamics through joint velocity.**

Overall, the results show that **robust insertion-success classification benefits from multi-modal input.** Therefore, the combination of F/T, TCP pose, joint velocity, and current is used in the remainder of the thesis.

## 2. BM1
For Benchmark 1, the main conclusion is therefore that the **Big config provides the stronger in-distribution classifier, but not the better calibrated one.** Since this benchmark uses a mixed split, the result should be interpreted as a baseline/control group performance under favorable conditions, not as evidence for generalization to unseen domains. Its main purpose is to define the reference level for the more difficult domain-shift experiments in Benchmark 2 and Benchmark 3.

## 3. BM2
Overall, Benchmark 2 shows that the model can generalize to most individual unseen hole domains with good test balanced accuracy. The Big config is preferable when classification performance is evaluated by balanced accuracy, because it improves the mean predictions in every domain. However, the results also show that generalization is very much domain-dependent. **Tighter tolerances generally reduce balanced accuracy, but this effect is much more visible for rectangular holes than for round holes.** The tightest rectangular domain, \texttt{BM2\_rect\_1}, therefore reveals a clear limitation of generalization and indicates that some combinations of geometry and tolerance introduce much stronger domain shifts than others. Benchmark 2 therefore provides a more realistic reference than Benchmark 1 and motivates the group-level generalization analysis in Benchmark 3.

## 4. BM3
Overall, Benchmark 3 shows that group-level generalization is substantially more difficult than the mixed split in Benchmark 1 and the individual-hole split in Benchmark 2. The Big config is consistently preferable in terms of balanced accuracy, but the improvement depends on the held-out group. **The model generalizes best across tolerance levels, although the tightest tolerance remains more difficult. Generalization to the big and small round rod types is also comparatively strong, while rectangular rod and rectangular gripper domains are clearly harder. The weakest results occur for batch-level generalization and for rectangular contact conditions in general.** Benchmark 3 therefore identifies the main limitations when generalizing to entire unseen deviation groups (batch 1 vs. batch 2 and 3) and to big geometric shifts (round vs. rectangular). It can however transfer well across tolerance changes and minor geometric shifts (big round rod vs. small round rod).

## DA1 Physical Success
Deviation Analysis I shows that the physical insertion outcome is strongly dependent on the recorded deviations. The clearest global effect is caused by tilt magnitude. **While position offset alone varies on a small scale, the success rate for tilt decreases consistently from no tilt to moderate tilt and large tilt.** This trend is also visible in the combined position over tilt groups, where increasing tilt reduces the success rate for each position-offset level. Angular misalignment is therefore more informative for physical insertion difficulty than radial position offset alone. 

The second major effect is the **interaction between force level and force limit.** They cannot be interpreted independently, because the relevant factor is their difference. Settings where the force difference $L-F$ is close to zero or negative show very low success rates, most clearly for \texttt{F40/L30}. In contrast, under favorable geometric conditions without relevant offset or tilt, several force interactions with \texttt{L40} reach complete success. This indicates that unfavorable force settings become especially critical when combined with difficult contact/geometric deviations. 

The domain-specific analysis confirms the global tilt trend across most hole domains, but also reveals local effects that are hidden in the global view. In \texttt{small\_rod\_t0.3}, negative $x$ and negative $y$ offsets are significantly less successful than positive offsets, suggesting a **directional bias of the start insertion pose** without deviations applied. This is practically relevant because such a **bias could potentially be corrected** by shifting the starting pose in the opposite direction, towards the true hole center. 

The investigation of \texttt{rect\_rod\_t0.1} shows that failed insertions come from different physical mechanisms. **Low or negative force differences mainly lead to force-limit failures, larger force limits can allow more stall/jam failures, and low force levels can produce timeouts.** Therefore, equal success rates do not necessarily represent the same physical behavior. The failure-reason analysis is essential for distinguishing these mechanisms. 

Overall, Deviation Analysis I concludes that **tilt magnitude and force interaction are the strongest global factors affecting physical success rate, while offset direction can reveal a robotic setup-specific positional bias** for selected domains. These results show that the metadata is not only useful for grouping the dataset, but also to explain physically difficult deviations affecting the insertion process.

## DA2 Model Confusion
The model-confusion analysis shows that physical difficulty and classification difficulty are related, but not identical. In Deviation Analysis I, large tilt clearly reduced the physical success rate. In Deviation Analysis II, however, the **large-tilt groups still reach comparatively high balanced accuracy.** This indicates that many physically failed insertions caused by large tilt produce signal patterns that are recognizable for the model. Therefore, **a deviation can be physically unfavorable without necessarily being model-confusing.**

The strongest model-confusion effects are found in the force-interaction groups. In particular, **force settings with a small or negative difference between force limit and force level lead to reduced balanced accuracy.** The most extreme example is \texttt{F40/L30}. This group is physically almost always unsuccessful, and the model therefore predicts the dominant failure class reliably. However, the few successful exceptions are missed, which results in a false-negative rate of 1.0. Other groups, such as **\texttt{F30/L30} and \texttt{F40/L40}, show both low balanced accuracy and high loss.** A high loss comes from penalizing high confidence to the wrong classes, or low probability to the true class. These groups are not only wrong classified after thresholding, but are confidentially wrong, **indicating genuinely ambiguous or inconsistent signal patterns.**

A central observation is that the **physical success rate of a deviation group inherently determines the local class balance available for learning and evaluation. A success rate close to 50\% is physically not desirable, but it provides a comparatively balanced distribution** of successful and failed insertions. This can be favorable for the classifier, because both classes are sufficiently represented within the group. In contrast, **the physically desirable case of almost 100\% success and the physically undesirable case of almost 0\% success both lead to highly imbalanced classes.** In these extreme cases, the model tends to learn the dominant outcome and struggles with the rare exceptions. This explains why physically easy groups such as \texttt{N.O./N.T.} can still show increased false-positive rates, while physically difficult groups such as \texttt{F40/L30} or the negative-$x$ large-offset/large-tilt group of \texttt{small\_rod\_t0.3} show increased false-negative behavior. 

The domain-specific analysis supports this interpretation. The detailed inspection of **\texttt{F30/L30}** shows that its low performance is not caused by one isolated position--tilt group. Instead, **the group remains ambiguous across several available position--tilt combinations and is dominated by force-limit-related failures.** The \texttt{rect\_rod\_t0.2} no-offset/no-tilt case demonstrates the importance of sample count: a very low balanced accuracy in a single heatmap cell can result from only a small number of prediction records and should not be interpreted as a global domain effect. In contrast, the \texttt{small\_rod\_t0.3} negative-$x$ case connects directly to the physical directional bias found in Deviation Analysis I. The model correctly classifies most failures in the large-offset/large-tilt subgroup, but misses most of the rare successful insertions. This shows that the model captures the dominant failures associated with this subgroup, but does not reliably identify successful exceptions. 

The record-offset groups show no severe global performance collapse. This indicates that the final model is reasonably robust to the tested temporal shifts of the recorded sequence window. Likewise, offset direction is not a major global source of model confusion, even though it revealed physical setup asymmetries in the success-rate analysis. 

Overall, Deviation Analysis II shows that the final PatchTST model using the best channels is robust across most deviation groups and domains. The main **classification weaknesses occur in locally imbalanced deviation- or force-limit-related groups**, where the signal patterns of rare exceptions are not sufficiently distinct from the dominant outcome.

# Main Findings and unorganized thoughts
## Finding 1: Multimodal robotic data in combination with higher model capacity provides a strong classifier, but not the better calibrated one. 

Third, the best-performing configuration combines complementary information: contact interaction through F/T, actuator load through current consumption, robot state through TCP pose, and motion dynamics through joint velocity. 
Overall, the results show that robust insertion-success classification benefits from multi-modal input. 
Big config provides the stronger in-distribution classifier, but not the better calibrated one. 
From Best Channel Study and BM1 

## Finding 2: 
The model generalizes best across tolerance levels, although the tightest tolerance remains more difficult. Tighter tolerances generally reduce balanced accuracy, but this effect is much more visible for rectangular holes than for round holes. Generalization to the big and small round rod types is also comparatively strong, while rectangular rod and rectangular gripper domains are clearly harder. The weakest results occur for batch-level generalization and for rectangular contact conditions in general.  
From BM2 and BM3

## Tilt magnitude
While position offset alone varies on a small scale, the success rate for tilt decreases consistently from no tilt to moderate tilt and large tilt.
large-tilt groups still reach comparatively high balanced accuracy

## force interation 
Settings where the force difference $L-F$ is close to zero or negative show very low success rates 
Low or negative force differences mainly lead to force-limit failures, larger force limits can allow more stall/jam failures, and low force levels can produce timeouts. 
force settings with a small or negative difference between force limit and force level lead to reduced balanced accuracy.

## Direction Bias investigation practical result 
directional bias of the start insertion pose
bias could potentially be corrected

## Model Confusion
a deviation can be physically unfavorable without necessarily being model-confusing.
\texttt{F30/L30} and \texttt{F40/L40}, show both low balanced accuracy and high loss. indicating genuinely ambiguous or inconsistent signal patterns.
\texttt{F30/L30} remains ambiguous across several available position--tilt combinations and is dominated by force-limit-related failures.
classification weaknesses occur in locally imbalanced deviation- or force-limit-related groups

## Physical Success against Classification/Model Confusion
physical success rate of a deviation group inherently determines the local class balance available for learning and evaluation. A success rate close to 50\% is physically not desirable, but it provides a comparatively balanced
physically desirable case of almost 100\% success and the physically undesirable case of almost 0\% success both lead to highly imbalanced classes.