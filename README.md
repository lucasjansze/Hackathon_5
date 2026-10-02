# Who needs early help? Predicting long-term unemployment for UWV

**Hackathon 5 – Model Showdown · SDG 8 Decent Work and Economic Growth · De Haagse Hogeschool**
Team: *[Name 1] & [Name 2]*

Notebook: [`uwv_long_term_unemployment.ipynb`](uwv_long_term_unemployment.ipynb) · Data: [`data/`](data/README.md) · Plan: [`docs/Plan_UWV_Hackathon5.docx`](docs/Plan_UWV_Hackathon5.docx)

---

## 1. The problem

When people in the Netherlands lose their job, most of them claim unemployment benefit (WW) from UWV. At the end of December 2025 there were **191,500 running WW benefits**, almost 10% more than a year earlier ([UWV, Sociale zekerheid in 2025](https://www.uwv.nl/nl/publicaties/jaar-en-tertaalverslagen/2025/sociale-zekerheid-in-2025-stijging-wia-instroom-vlakt-af)). CBS reported in September 2025 that unemployed people are searching for work slightly longer than before ([CBS, Werklozen iets langer op zoek naar werk](https://www.cbs.nl/nl-nl/nieuws/2025/38/werklozen-iets-langer-op-zoek-naar-werk)).

Long-term unemployment, meaning a year or more, matters because it becomes self-reinforcing: skills and networks fade, and employers become hesitant. It also has a hard financial edge. WW lasts at most 24 months, and after that comes *bijstand* (social assistance), where people first have to use up their savings and their partner's income counts.

UWV tries to prevent this with personal help, but counsellor time is limited. At the start of every WW benefit, UWV's algorithm **Werkverkenner** estimates the chance that the person is back at work within 12 months. People below 50% are invited early for a face-to-face conversation with a work coach. The others start with online services and are seen in person after about three months ([UWV algorithm register](https://www.uwv.nl/nl/over-uwv/algoritmeregister-uwv/werkverkenner)).

The help works, but modestly. UWV's own evaluation found that personal service raises the chance of work by about **2 percentage points** within a year, with about €201 million in societal benefits against €97 million in costs ([UWV 2021](https://www.uwv.nl/nl/publicaties/kennis/2021/effect-persoonlijke-dienstverlening-op-werkkans-en-ww-duur)). So it matters *who* gets that early conversation.

**Our question:** can open Dutch data, using only what is known when someone becomes unemployed (personal background and the previous job), predict who will be unemployed for **12 months or longer**? And does such a model make the same mistakes for everyone?

**Does our data population match the people we want to use the model for?** Only partly, and we say so openly. UWV's population is WW claimants at the start of their benefit. Our data are 1,163 Dutch survey respondents aged 18–66 who were unemployed and looking for work for at least three months in the last five years. Some of them never received WW (for example after a short temporary contract), and their characteristics were measured at the interview, not on their first day of unemployment. Section 8 lists what this means.

## 2. User group

**Who uses the prediction.** The users are **UWV work coaches** (*werkcoaches*) who run intake for new WW claimants, and the UWV team that decides how many people can be invited early (the threshold). Work coaches are professionals with a full caseload. They need a short, explainable signal at intake, *"this person has a high risk, invite early"*, and they need to be able to explain that signal to the job seeker. They use it at one specific moment: the first weeks of a WW benefit, before the regular three-month conversation.

**Who the prediction is about.** The people affected are **job seekers aged 18–66 who have just lost their job**. In our data this group has a median age of 42. 23% are 18–29, 45% are 30–49 and 32% are 50–66. 54% are women. 20% were born abroad and another 11% were born in the Netherlands to at least one parent born abroad. Their education is spread evenly over low, middle and high. Four in ten had a temporary contract in their last job. Half of them (51.5%) were eventually unemployed for 12 months or longer. Long-term unemployment is far more common for people over 50 (64%) than for people under 30 (32%), and more common for people with low education and people born abroad.

**Who is *not* the intended user.**
- **Employers and recruiters.** They must never use a risk score to screen applicants.
- **Fraud and enforcement teams** (inside or outside UWV). A risk score of long-term unemployment says nothing about fraud. Leiden University found that imposing job-search obligations on WW recipients backfires ([Universiteit Leiden, 2024](https://www.universiteitleiden.nl/nieuws/2024/01/werklozen-verplichten-breder-naar-werk-te-zoeken-pakt-vaak-averechts-uit)).
- **Municipalities, for bijstand clients.** That is a different population, and our model was not built or tested for it.

**Who is missing from the data.** People who do not speak Dutch well enough to do a survey interview are hardly represented, and they are exactly a group UWV worries about. People under 18 or over 66, people living in institutions and undocumented workers are not in the data at all.

## 3. Why this fits this user group

The work coach already makes this decision, with an algorithm and a threshold, so we don't add a new step to their work. We give them a model they can **check and explain**. The logistic regression we recommend shows *which* characteristics raise the risk (age, never having had a paid job, elementary occupations, region). A coach can then say *"you are invited early mainly because…"*, which a black-box score cannot do. That matters for the job seeker's trust, and it fits UWV's own choice to publish its algorithms in a public algorithm register.

The simple alternative a coach could use without any model is *"invite everyone over 50"*. It finds only 37% of the long-term unemployed on our test set. The model finds 70% while inviting about half of the job seekers, or 87% if UWV can invite about seven in ten early. Inviting everyone, the other simple alternative, is impossible with UWV's capacity. Machine learning beats both, but only modestly, and that is why we position the model as *support* for the coach, never as the decision-maker (section 6).

## 4. SDG 8

**Target 8.5** is *full and productive employment and decent work for all women and men, including young people and persons with disabilities*. **Indicator 8.5.2** is the unemployment rate. Our model is about one concrete part of that target: preventing short unemployment from becoming long-term unemployment, by getting scarce help to the right people in their first weeks of unemployment.

## 5. The solution, step by step

**Input:** characteristics of one job seeker that are known at intake. These are age, gender, education level, parents' education, household size, whether they live with a partner or children, type of area and province, and about their last job: contract type, employee or self-employed, workplace size, whether they supervised others, occupation group and sector.
**Output:** the probability that the unemployment lasts 12 months or longer, plus an advice: *invite early* or *online services first*.

The notebook does the following, in order. Each step has a markdown cell explaining what we did and why.

1. **Frame the problem.** Positive class = unemployed 12+ months. A false negative (missing someone who needs help) is worse than a false positive (one extra conversation). The main metric is **balanced accuracy**, because the long-term group is the majority (51.5%). A model that simply invites everyone gets recall 1.0 on that group, while balanced accuracy gives it the 0.5 it deserves. In step 9 we then move the threshold to raise recall.
2. **Explore the data.** We replaced ESS missing-value codes (7/8/9, 77/88/99, 66666 and so on) with real missing values. We filtered the population (13,890 → 1,163 rows), checked impossible values (none) and duplicates (none). We found and **dropped leaking columns**: `mnactic` (current activity) shows 74% long-term among people unemployed *now* against 35% among people in paid work, and income and health variables are consequences of long unemployment. Migration background is kept *out* of the model and used only for fairness checks.
3. **Split first.** A stratified 80/20 split (930 / 233 rows) with `random_state=42`, before any pre-processing.
4. **Pipeline.** A `ColumnTransformer` handles numeric columns (median imputer, then `StandardScaler`) and categorical columns ("missing" as its own category, then one-hot encoding with categories of fewer than 10 people grouped together). It is fitted on training data only.
5. **Baselines.** `DummyClassifier(most_frequent)` and the rule "invite everyone aged 50 or older".
6. **Tuning.** `GridSearchCV` with the same 5 stratified folds and balanced accuracy for all three models. KNN: `n_neighbors`, `weights`. Logistic regression: `C`, `class_weight`. Random forest: `max_depth`, `min_samples_leaf`, `class_weight`. The notebook plots the score against each main hyperparameter.
7. **Test once.** Confusion matrices, precision, recall, F1 and balanced accuracy, with train, CV and test scores side by side.
8. **Errors and groups.** Recall per gender, age band, origin and education. We also run a with/without migration-background experiment and a proxy check.
9. **Recommendation and threshold.** The threshold is chosen from cross-validated probabilities on the training set only.
10. **Use it.** A made-up job seeker gets a probability and an advice.

### Comparison table (test set: 233 people, scored once)

| Model | Best hyperparameters | CV balanced accuracy (mean ± std) | Test precision | Test recall | Test F1 | Test balanced accuracy |
|---|---|---|---|---|---|---|
| **Baseline: most frequent** | – | 0.500 ± 0.000 | 0.515 | 1.000 | 0.680 | 0.500 |
| Baseline: age ≥ 50 | threshold 50 | 0.590 ± 0.038 | 0.603 | 0.367 | 0.456 | 0.555 |
| KNN | n_neighbors=47, weights=uniform | 0.643 ± 0.052 | 0.652 | 0.767 | 0.705 | 0.667 |
| **Logistic regression** | C=10, class_weight=balanced | **0.674 ± 0.016** | 0.677 | 0.700 | 0.689 | 0.673 |
| Random forest | max_depth=3, min_samples_leaf=20, class_weight=balanced | 0.672 ± 0.035 | 0.667 | 0.700 | 0.683 | 0.664 |

Recommended model at threshold 0.35 (test set): recall 0.867, precision 0.615, 73% of people invited early.

## 6. Recommendation

**We recommend the logistic regression, as support for the work coach and not as the decision-maker.** It has the best and most stable cross-validation score (0.674 ± 0.016). The random forest and KNN are not meaningfully better or worse, because the differences are smaller than the spread over the folds. With equal performance, we choose the model whose reasoning a coach can explain.

At the default threshold of 0.5 it invites about half of the job seekers early and finds 70% of the long-term unemployed. Because missing someone is the worse mistake, we advise a threshold of **0.35** if UWV can see about seven in ten people early: that finds 87% of the long-term unemployed. If UWV can't, use 0.5. That choice is about capacity, and the notebook shows its cost.

**Is it good enough? Not to decide alone.** Roughly one in three predictions is wrong. The mistakes also fall unevenly: the model finds only 35% of the long-term unemployed aged 18–29 (against 89% of those aged 50–66) and half of the higher-educated ones. A young or highly educated person who gets stuck "looks like" someone who will find work quickly. Therefore:
- The score may only **add** people to the early-invitation list. A low score must never mean less help.
- It must never be used for obligations, sanctions or fraud detection.
- Before any real use it must be retrained on recent UWV data of actual WW claimants, with the subgroup check repeated each time.

Our results cannot be compared with UWV's "70% correct". That figure is accuracy, on a different population, with a questionnaire we don't have.

## 7. Ethical reflection

This model scores people at a vulnerable moment, just after losing their job, and decides who gets help first. The most concrete risk we found is **unequal errors**. On our test set the model misses most young long-term unemployed people (recall 0.35 for ages 18–29) and half of the higher-educated ones (0.51), and men somewhat more often than women (0.65 against 0.74), even though men and women are equally often long-term unemployed. For those people, a false negative means waiting three months or more for personal help while their WW runs down, and the evidence says that help raises their chance of work. A false positive costs a work coach one conversation, and costs the job seeker little, *as long as the conversation is support and not pressure*.

**Migration background** is the second risk. People born abroad are more often long-term unemployed in our data (62% against 49%). Giving the model this information could steer help to them, but scoring people on where they or their parents were born is exactly what the District Court of The Hague rejected in the SyRI case (2020) and what went wrong in the childcare benefits scandal. We tested it. Adding migration background did not improve the model (balanced accuracy 0.676 against 0.674), so we **left it out**. Removing the column is not enough, though. Our proxy check shows that the remaining features still partly reveal origin (ROC-AUC 0.67), so we **report recall per origin group** as a standing fairness check instead of assuming the model is blind.

**What we did about these risks:**
- We chose balanced accuracy as the main metric, so that "invite everyone" or "invite only older people" cannot look good.
- We lowered the threshold to 0.35 to reduce missed people, and showed the cost.
- We left migration background out and checked recall per group.
- We set the rule that the score may only add help, never remove it, and must never be used for sanctions.

**Consent and licence.** ESS respondents agreed to take part in research, not to be the basis of decisions about other people. The data are licensed CC BY-NC-SA 4.0 (non-commercial, with attribution), which fits a study prototype but not a production system.

**This model must not be used:** to decide about people outside the population it was trained on (non-Dutch speakers, people under 18 or over 66, bijstand clients); by employers; or for fraud detection or sanctions.

## 8. Dataset card

| | |
|---|---|
| **Source** | European Social Survey (ESS ERIC), [ESS Data Portal](https://europeansocialsurvey.org/data-portal); details and editions in [`data/README.md`](data/README.md) |
| **Collected by** | ESS ERIC and the Dutch national ESS team, with fieldwork by a survey agency |
| **How** | Interviews with a new random probability sample of residents of the Netherlands aged 15+ in private households every round, mostly face-to-face (see the ESS documentation of each round for fieldwork details) |
| **When** | Rounds 4–11: 2008, 2010, 2012, 2014, 2016, 2018, 2020–22, 2023–24 |
| **Licence** | CC BY-NC-SA 4.0 (ESS recommends linking to the portal; see `data/README.md`) |
| **Size** | Raw file: 13,890 respondents × 827 columns. After filters (unemployed 3+ months, in the last 5 years, aged 18–66, target answered): **1,163 rows**; 16 features used (5 numeric, 11 categorical) + migration background for fairness checks |
| **Target** | `uemp12m`: "Did any period of unemployment and work seeking last 12 months or more?" 1 = yes (**51.5%**), 0 = no (48.5%) |
| **Known limitations** | (1) `uemp12m` refers to *any* period, not necessarily the most recent one. (2) Not everyone in the population received WW. (3) Characteristics are measured at the interview, not at the start of unemployment; the 5-year filter limits but doesn't remove this. (4) Retrospective self-reports can be misremembered. (5) Non-Dutch speakers are under-represented. (6) Occupation and sector use different classifications before round 6 / round 5; we harmonised them to broad groups. (7) Region was not asked in round 4. (8) The analysis is unweighted, so it describes the sample, not exactly the Dutch population. |

## 9. How to run

**In Google Colab (no installs needed):** open `uwv_long_term_unemployment.ipynb` in Colab (File → Open notebook → GitHub → `lucasjansze/Hackathon_5`), then choose *Runtime → Run all*. The notebook downloads the CSV from this repository automatically. If the repository is private, first upload the CSV to a `data/` folder in the Colab file panel.

**Locally:** clone the repository and run the notebook from the repository folder. The CSV is read from `data/`.

Built and tested with Python 3.11, pandas 3.0, numpy 2.4, scikit-learn 1.9 and matplotlib. The notebook only uses packages that Colab has by default. `random_state=42` is used everywhere. Other package versions can shift numbers in the third decimal.

## 10. Sources

- UWV – [Werkverkenner (algorithm register)](https://www.uwv.nl/nl/over-uwv/algoritmeregister-uwv/werkverkenner)
- UWV – [Effect of personal services on job chances and WW duration (2021)](https://www.uwv.nl/nl/publicaties/kennis/2021/effect-persoonlijke-dienstverlening-op-werkkans-en-ww-duur)
- UWV – [Sociale zekerheid in 2025](https://www.uwv.nl/nl/publicaties/jaar-en-tertaalverslagen/2025/sociale-zekerheid-in-2025-stijging-wia-instroom-vlakt-af)
- UWV – [Kennisverslag 2015-3 (work-orientation conversation)](https://www.uwv.nl/assets-kai/files/6fc77bdf-5d12-4389-af70-84c93e1e95c0/uwv-kennisverslag-ukv-2015-3.pdf)
- CBS – [Werklozen iets langer op zoek naar werk (2025)](https://www.cbs.nl/nl-nl/nieuws/2025/38/werklozen-iets-langer-op-zoek-naar-werk)
- Universiteit Leiden – [Werklozen verplichten breder naar werk te zoeken pakt vaak averechts uit (2024)](https://www.universiteitleiden.nl/nieuws/2024/01/werklozen-verplichten-breder-naar-werk-te-zoeken-pakt-vaak-averechts-uit)
- District Court of The Hague – SyRI judgment, 5 February 2020 (ECLI:NL:RBDHA:2020:865)
- European Social Survey – [Data Portal](https://europeansocialsurvey.org/data-portal) and [conditions of use](https://europeansocialsurvey.org/node/58)
- scikit-learn – [documentation](https://scikit-learn.org/stable/)
