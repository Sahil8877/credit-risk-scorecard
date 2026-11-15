# Credit Risk Scorecard

A credit scorecard built end to end in Python on the Kaggle "Give Me Some Credit" dataset. It predicts which borrowers will become seriously delinquent (90+ days past due) within two years, and converts the model into a points based scorecard like the ones lenders use.

The full work is in one notebook: `notebooks/credit_risk_scorecard.ipynb`. Every step has a heading and a plain English explanation.

## Results

Tested on 45,000 borrowers the model never saw during training.

| Measure | Result |
|---|---|
| AUC (test) | 0.858 (train 0.856) |
| Gini (test) | 0.716 |
| KS (test) | 0.554 |
| PSI (train vs test scores) | 0.0003 |
| Default rate, lowest score band | about 36% |
| Default rate, highest score band | about 0.5% |
| Share of all defaulters in the lowest 10% of scores | about 54% |
| Share of all defaulters in the lowest 20% of scores | about 71% |

The close match between train and test AUC and the very low PSI suggest the model is stable and not overfitted to the training data.

## Data

- Source: [Give Me Some Credit](https://www.kaggle.com/c/GiveMeSomeCredit) (Kaggle). 150,000 borrowers, 10 input columns, target `SeriousDlqin2yrs`.
- Overall default rate: 6.68%.
- The data file is not included in this repository. Download `cs-training.csv` from Kaggle and place it at `data/cs-training.csv` before running the notebook.

## Method

1. **Profile the data.** Check missing values, ranges and unusual codes in every column.
2. **Clean on a copy.** The raw data stays untouched. One row with age 0 was dropped. Debt ratio values above 100 were blanked. Past due codes 96 and 98 were blanked and flagged (see below).
3. **Split.** 70% train, 30% test, stratified on the target so both keep the same default rate (`random_state=0`).
4. **Bin and score each variable.** Continuous variables were grouped into bins using equal sized groups or hand chosen cut points, with missing values kept as their own bin. Each bin gets a Weight of Evidence (WoE), and each variable gets an Information Value (IV) to rank how useful it is.
5. **Fit logistic regression** on the WoE values. All coefficients came out negative, as expected for the WoE sign used.
6. **Scale to points.** 600 points at 50:1 good to bad odds, and 20 points to double the odds.
7. **Evaluate.** AUC, Gini, KS, default rate by score band, and PSI between train and test.

Ten variables were binned and nine are in the final model. In total the scorecard has 68 bins.

### Variables ranked by Information Value

| Variable | IV | Note |
|---|---|---|
| Revolving utilisation of unsecured lines | 1.1105 | Very strong |
| Times 90+ days late | 0.8742 | Very strong |
| Times 30 to 59 days late | 0.7584 | Very strong |
| Times 60 to 89 days late | 0.5914 | Very strong |
| Age | 0.2528 | Medium |
| Monthly income | 0.0734 | Weak |
| Real estate loans | 0.0649 | Weak |
| Open credit lines and loans | 0.0643 | Dropped, positive coefficient (wrong sign) |
| Debt ratio | 0.0518 | Weak |
| Number of dependents | 0.0328 | Weak |

## Decisions worth knowing about

- **Placeholder codes 96 and 98.** The past due columns contain these two codes, which are not documented. 269 borrowers have them, and they defaulted at 54.65% against 6.68% overall. They were not deleted. The codes were blanked and a single flag marks these borrowers, so the model can treat them as their own group.
- **Debt ratio above 100.** 24,380 rows have a debt ratio above 100, and 22,693 of them have no recorded income. Values above 100 were treated as invalid and blanked.
- **Utilisation above 10.** 241 rows have a revolving utilisation above 10, which is not a plausible ratio. These were left unchanged, and the dilution this causes is noted in the notebook.
- **One variable removed.** Open credit lines had a positive coefficient, which contradicts the direction of its WoE and usually means it overlaps with another variable. Removing it and refitting gave the same performance.

## Limitations

- **Possible leakage.** Codes 96 and 98 may be assigned after a borrower becomes delinquent. If so, the flag would not be available at the time of a real lending decision and the results would look better than they should. A useful next check is to refit without the 269 flagged borrowers.
- **One dataset, one split.** Results come from a single train and test split of one public dataset. No cross validation or out of time test was done.
- **No cut off or business analysis.** The project stops at scoring and ranking. It does not choose an approval cut off or estimate profit and loss.
- **Not a production model.** There is no monitoring, no fairness review and no documentation of the kind a regulator would expect.

## How to run

```
git clone https://github.com/Sahil8877/credit-risk-scorecard.git
cd credit-risk-scorecard
pip install pandas numpy scikit-learn matplotlib jupyter
```

Place `cs-training.csv` in the `data` folder, then open `notebooks/credit_risk_scorecard.ipynb` and choose Restart and Run All.

## Tools

Python, pandas, NumPy, scikit-learn, matplotlib, Jupyter.
