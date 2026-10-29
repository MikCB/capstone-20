# Technical notes and changes from Module 20

## Study lineage and source scope

The final project extends Module 11's draft question, Module 16's county-level proposal, and the published Module 20 initial report at commit `8ca462f0b2f3b852a06df3bd2c165066591aa0e0`. No additional capstone assignment completion or grade is inferred. This package prepares the Module 24 final deliverables; it does not publish or submit them.

The raw election snapshots match the published Module 20 Git blob hashes. The same locally preserved ACS JSON response is compressed deterministically in this package. No new finance or outreach data were acquired. Source manifests distinguish original publishers from the public mirrors actually used.

## Corrected geography

The election sources contain separate JACKSON and KANSAS CITY reporting rows in Missouri. Official jurisdiction descriptions establish that KCEB serves the Kansas City portion of Jackson County and JCEB serves its complement. The analytical harmonization sums the disjoint reporting totals and 2020 party votes before calculating county measures. It does not sum repeated totals across candidates or assign Kansas City across multiple counties.

| Year | Jackson jurisdiction | Kansas City jurisdiction added | Corrected county total |
|---|---:|---:|---:|
| 2020 | 196,418 | 136,645 | 333,063 |
| 2024 | 193,729 | 124,288 | 318,017 |

Official sources: [KCEB FAQ](https://kceb.org/board/faq/), [JCEB jurisdiction](https://jcebmo.org/election-information/jurisdiction/). The mapping is an analytical inference from those stated jurisdictions. Raw source rows remain unchanged. The initial draft's 2024 county participation of 35.08% becomes 57.59% after this correction.

The pre-harmonization noncounty audit still lists the recovered KANSAS CITY row, explicitly marked as recovered. For 2024, 42 source reporting units lack county FIPS; one is recovered in the Jackson County total, leaving 41 Alaska units excluded as noncounty geography. Eight Connecticut county rows fail the ACS geography join. Five other county rows exceed 100% on the proxy and are excluded, not clipped. Final N is 3,099.

## Final predictors and validation

Eight predictors are fixed: `log_population, pct_age_18_29, pct_age_65_plus, median_household_income, pct_bachelors_plus, pct_poverty, prior_dem_share_pct, prior_competitiveness`. Race/ethnicity shares are not final model inputs. The feature decision was informed by Module 20, so the entire study remains exploratory. FIPS, state, current vote counts, and the target denominator are not provided as direct predictors. Log population remains a declared size predictor; this does not eliminate all relationships with the population-based outcome.

Outer GroupKFold: 5 folds; inner GroupKFold: 3 folds, grouped by state/DC. There is zero state overlap in every outer and inner split. The combined search compares 21 candidates (5 Ridge, 8 random forest, 8 histogram gradient boosting), producing 315 inner candidate-fold fits plus refits. Final full-data five-fold search adds 105 candidate-fold fits. Searches use `n_jobs=1`, negative MAE scoring, `error_score='raise'`, and training-only median imputation/scaling. Gradient boosting uses `early_stopping=False`.

Selected families within the outer training folds: Gradient boosting: 5 of 5 outer folds. All family-specific comparisons use each family's inner winner, not the best outer test score. The main out-of-fold prediction is the family/hyperparameter winner chosen within each outer training set. Every county is predicted once by a model whose outer training set excludes its whole state.

## Evaluation results

| Model or selection procedure | Pooled outer MAE pp | Pooled outer RMSE pp | Pooled outer R² | Predictions outside 0–100 |
|---|---:|---:|---:|---:|
| Mean baseline | 8.048556 | 10.082092 | -0.024374 | 0 |
| Ridge regression | 5.536140 | 7.101227 | 0.491812 | 0 |
| Random forest | 5.215445 | 6.670202 | 0.551631 | 0 |
| Gradient boosting | 5.172283 | 6.590846 | 0.562236 | 0 |
| Nested selection procedure | 5.172283 | 6.590846 | 0.562236 | 0 |

The main performance result evaluates the combined selection procedure. The full-data fitted model may differ from the most common outer-fold family; its selection score is not an independent generalization estimate. Selecting a family post hoc using the family-specific outer results would add selection optimism, so that is not the reported primary procedure.

The Module 20 Ridge MAE of 5.58 points and the final workflow MAE of 5.17 points are not a like-for-like performance comparison: the final analysis changes the county preparation, feature set, and validation design. The 35.7% improvement claim compares only the final workflow and its baseline under the same final outer folds.

## Full-data fitted model

Family: **Gradient boosting**.

| Parameter | Selected value |
|---|---|
| model | HistGradientBoostingRegressor |
| l2_regularization | 0.0 |
| learning_rate | 0.05 |
| max_leaf_nodes | 7 |

The full-data model is refit for reproducibility and retrospective description. Its predictions on the fitting data are not used as validation evidence. The fitted model object is not serialized; rerunning Notebook 02 reconstructs it.

## Interpretation and statistical uncertainty

MAE weights counties equally and is measured in percentage points of the participation proxy. RMSE emphasizes larger misses. R² compares squared error with a test-set mean and is not individual-voter accuracy. Fold dispersion is geographic variation, not a confidence interval.

The state-resampling calculation uses 2,000 bootstrap draws of 49 whole state/DC groups, with replacement, from fixed out-of-fold residuals. Each draw keeps all counties in sampled states and uses county-weighted MAE. The paired baseline-minus-model statistic preserves the same sampled states for both methods. Percentile endpoints are at 2.5% and 97.5%. These are approximate geographic resampling intervals conditional on the saved predictions; they do not capture the full variability from refitting the searches or propagate ACS margins of error. Overlapping training folds create dependence. No causal test or individual-level inference is made.

Permutation importance is computed within outer excluded-state data using 8 permutations per predictor, then averaged using outer county counts. Correlated features may mask or share importance. The recorded outer-fold SD measures variation between folds, not a population confidence interval.

## Measurement and scope limitations

The target divides presidential votes by estimated adults, including ineligible adults, using a five-year rather than election-day estimate. Alaska, Connecticut, and five extreme-ratio counties are absent. Source candidate sums differ from reported totalvotes for 127 valid-FIPS units; these are audited and source totals retained. Reported subgroup errors use fixed predictions, without refitting or model selection. Uncertainty is greater than the displayed resampling calculation alone establishes.

No campaign-spending allocation, direct canvassing exposure, true prior turnout, urban/rural code, clustering solution, or future-election forecasting validation is claimed. The final question has been narrowed explicitly to the demographic and prior political-context information actually measured.
