# Who needs early help?

**AI for Good — Hackathon 5: Model Showdown**
*Lucas Jansze & Minhyeok Sung*

A scikit-learn model that predicts, at the moment someone loses their job, whether they will still be without paid work a year later – so that UWV's work coaches can invite the people who need it most for an early face-to-face conversation.

| | |
|---|---|
| Notebook | [`uwv_long_term_unemployment.ipynb`](uwv_long_term_unemployment.ipynb): raw LISS files in, comparison table and a prediction for a new job seeker out (run with outputs saved) |
| Ethical reflection | [`ETHICS.md`](ETHICS.md) |
| Data | LISS panel, not included – how to get it: [`data/README.md`](data/README.md) |
| All variables, question texts and answer codes | [`docs/liss_overview.md`](docs/liss_overview.md) (made by [`scripts/liss_inventory.py`](scripts/liss_inventory.py); no respondent data) |
| Project plan | [`docs/Plan_UWV_Hackathon5.docx`](docs/Plan_UWV_Hackathon5.docx) (written when we still used ESS data) |
| Slides | *TODO: add our slides* |
| Tool | scikit-learn 1.9: KNN, logistic regression and random forest |
| SDG | SDG 8 Decent Work and Economic Growth, target 8.5 |

> **Privacy.** The LISS data may not be redistributed, so it is not in this repository (`data/liss/` is in `.gitignore`). The notebook shows no individual people: only counts, percentages, averages and model scores for groups, and groups smaller than 10 people are hidden. People in LISS have no names, only an encrypted number, which we never display.

---

## Problem definition

When people in the Netherlands lose their job, most of them claim unemployment benefit (WW) from UWV. At the end of December 2025 there were **191,500 running WW benefits**, almost 10% more than a year earlier (UWV, 2026), and CBS reported in September 2025 that unemployed people are searching for work slightly longer than before (CBS, 2025).

Long-term unemployment, meaning a year or more, becomes self-reinforcing: skills and networks fade, and employers become hesitant. It also has a hard financial edge. WW lasts at most 24 months, and after that comes *bijstand* (social assistance), where people first have to use up their savings and their partner's income counts.

UWV tries to prevent this with personal help, but counsellor time is limited. At the start of every WW benefit, UWV's algorithm **Werkverkenner** estimates the chance that the person is back at work within 12 months. People below 50% are invited early for a face-to-face conversation with a work coach; the others start with online services and are seen in person after about three months (UWV, algorithm register). The help works, but modestly: personal service raises the chance of work by about **2 percentage points** within a year, with about €201 million in societal benefits against €97 million in costs (UWV, 2021). So it matters *who* gets that early conversation.

**Our question:** at the moment someone loses their job, can Dutch panel data – using only what is known at that moment – predict who will **still not be back in paid work twelve months later**? And does such a model make the same mistakes for everyone?

**Does our data match the people we want to use the model for?** Better than most survey data, but not exactly. UWV's population is WW claimants at the start of their benefit. Ours is **2,019 job losses** of LISS panel members aged 18–66 who, in the monthly data between November 2007 and 2025, went from paid work in one month to *job seeker following job loss* in the next. Because LISS follows the same people every month, we see the job loss when it happens, measure everything *before* it – UWV's intake moment – and see what happens in the twelve months after. What does not match: not everyone who loses a job claims WW, and LISS records someone's main activity as reported by the household, not their benefit (see the dataset card).

## SDG 8 — Decent Work and Economic Growth

**Target 8.5** is *full and productive employment and decent work for all women and men, including young people and persons with disabilities*; **indicator 8.5.2** is the unemployment rate. Our model is about one concrete part of that target: preventing a short spell without work from turning into a long one, by getting scarce help to the right people in their first weeks of unemployment.

## User group

**Who uses the prediction.** **UWV work coaches** (*werkcoaches*) who run intake for new WW claimants, and the UWV team that decides how many people can be invited early (the threshold). Work coaches have a full caseload. They need a short, explainable signal at intake – *"this person has a high risk, invite early"* – that they can explain to the job seeker. They use it at one moment: the first weeks of a WW benefit, before the regular three-month conversation.

**Who the prediction is about.** **Job seekers aged 18–66 who have just lost their job.** In our data 23% of the job losses are of people aged 18–29, 43% of people aged 30–49 and 34% of people aged 50–66. **39%** were still not back in paid work twelve months later, and that share rises steeply with age: 25% for 18–29, 36% for 30–49 and 53% for 50–66. People who migrated to the Netherlands themselves (57%), people with a long-standing disease (51%) and lower-educated people (44%) are more often long-term; women (41%) slightly more often than men (37%).

### Not for

- **Employers and recruiters.** They must never use a risk score to screen applicants.
- **Fraud and enforcement teams** (inside or outside UWV). A risk of long-term unemployment says nothing about fraud, and imposing job-search obligations on WW recipients backfires (Universiteit Leiden, 2024).
- **Municipalities, for bijstand clients.** That is a different population, and the model was not built or tested for it.

### Who is missing from the data

LISS questionnaires are in Dutch, so people who do not read Dutch well are hardly represented – exactly a group UWV worries about. LISS gives households without internet a computer, but people who do not want to join a monthly panel are not in it, and people who left the panel during their unemployment were dropped (74 job losses). People under 18 or over 66, people living in institutions and undocumented workers are not in the data at all.

## Solution: from raw data to a recommendation

**Input:** what is known about one job seeker at intake – age, gender, education, household, urbanity, net income in the last job, how many months they were looking for work in the three years before, the job they lost (contract type, public or private, hours, years with the employer, sector, occupation, supervising others, firm size), and health before the job loss (self-rated health, a long-standing disease, how much health hinders daily life).
**Output:** the probability that the person is still not back in paid work after 12 months, and an advice: *invite early* or *start with online services*.

The notebook runs from the raw files to that output with *Restart & Run All*. Each step has a markdown cell explaining what we did and why:

1. **Read the raw LISS files** (zipped or unzipped): 220 monthly Background Variables files and every wave of the core studies *Work and Schooling* and *Health*.
2. **Frame the problem.** Positive class = not back in paid work 12 months after the job loss. A missed person (false negative) is worse than an extra conversation (false positive). Main metric: **balanced accuracy** – "invite nobody" and "invite everyone" both score 0.5 on it, while accuracy rewards inviting nobody and recall rewards inviting everyone.
3. **Explore the data.** A **job loss** is the first month as *job seeker following job loss* directly after a month in paid work; the eleven months after it decide the target, and job losses whose outcome is unknown are dropped. Every feature is taken **from before** the job loss: the last month in work, the 36 months before it, and the latest *Work and Schooling* and *Health* interview in the 24 months before. *Don't know* codes and impossible values (a working week of 0 or 90 hours, a start year in the future) become missing. Everything measured after the job loss is left out on purpose, and migration background is used only for fairness checks.
4. **Split first.** 80/20, stratified and **grouped by person** (`StratifiedGroupKFold`, `random_state=42`): 352 people lost a job more than once, and all their job losses stay on one side, so the model cannot score well by recognising people.
5. **Pipeline.** A `ColumnTransformer` imputes the median plus a *was missing* column for numbers and a "missing" category for answers, scales numbers and one-hot encodes answers (categories under 10 people grouped). It is fitted on training data only.
6. **Baselines.** `DummyClassifier(most_frequent)` and the rule "invite everyone aged 50 or older".
7. **Tuning.** `GridSearchCV` with the same 5 person-grouped folds and balanced accuracy for all three models, with a plot of the score against `n_neighbors`, `C` and `max_depth`.
8. **Test once.** Confusion matrices, precision, recall, F1 and balanced accuracy, with train, CV and test scores side by side.
9. **Errors and groups.** The average profile of the missed long-term cases, recall per age band, gender, education, origin and long-standing disease, and two experiments: with/without migration background (plus a proxy check) and with/without health.
10. **Recommend and use it.** A threshold chosen on cross-validated probabilities of the training set, the coefficients of the logistic regression, and a made-up job seeker who gets a probability and an advice.

### Comparison table (test set, scored once)

Training set 1,614 job losses, test set 405 (39% long-term in both), no person in both.

| Model | Best hyperparameters | CV balanced accuracy (mean ± std) | Test precision | Test recall | Test F1 | Test balanced accuracy |
|---|---|---|---|---|---|---|
| **Baseline: most frequent** | – | 0.500 ± 0.000 | 0.000 | 0.000 | 0.000 | 0.500 |
| Baseline: age ≥ 50 | threshold 50 | 0.603 ± 0.020 | 0.492 | 0.411 | 0.448 | 0.570 |
| KNN | n_neighbors=7, weights=distance | 0.590 ± 0.019 | 0.467 | 0.310 | 0.373 | 0.542 |
| **Logistic regression** | C=10, class_weight=balanced | 0.640 ± 0.018 | 0.510 | 0.620 | 0.560 | **0.620** |
| Random forest | max_depth=12, min_samples_leaf=5, class_weight=balanced | **0.642 ± 0.022** | 0.494 | 0.538 | 0.515 | 0.593 |

Train / CV / test balanced accuracy: logistic regression 0.69 / 0.64 / 0.62 (mild overfitting), random forest 0.85 / 0.64 / 0.59 (clear overfitting), KNN 1.00 / 0.59 / 0.54 (memorises the training data).

## Recommendation

**We recommend the logistic regression, as support for the work coach and not as the decision-maker.**

- **Why this model.** In cross-validation it is as good as the random forest: a difference of 0.002, far below the spread over the folds (± 0.02). It overfits much less, holds up better on the test set (0.620 against 0.593), and a work coach can explain its prediction: older age, a long-standing disease and some sectors (public administration, catering, transport) raise the risk; a higher income, higher occupations and a university degree lower it. We decided beforehand that when models are equally good, we take the one that can be explained. KNN is clearly weaker – on the test set even below the age rule.
- **Which threshold.** At 0.5 the model invites 47% of job seekers and finds 62% of the long-term cases. At **0.35** it finds 80% but invites 72%. Because a missed person is the worse mistake, we advise **0.35 if UWV can see about seven in ten people early**, and 0.5 if it cannot. That is a capacity decision for UWV.
- **Is it good enough? Not to decide alone.** Roughly four in ten people in each group are classified wrongly, and the mistakes fall unevenly: the model finds only 29% of the long-term unemployed aged 18–29 and 46% of the men, against 77% of the women. So the score may only **add** people to the early-invitation list – a coach can always invite someone, and should look in particular at young people and people whose job history is unknown – and it must never be used for obligations, sanctions or fraud detection. Before any real use it must be retrained on recent UWV data of WW claimants, with the group check repeated every time.

Our numbers cannot be compared with UWV's "70% correct": that figure is accuracy, on a different population, with a questionnaire we don't have.

## Problem–solution fit

The work coach already makes this decision with an algorithm and a threshold, so we don't add a new step to their work – we give them a model they can **check and explain**, which fits UWV's own choice to publish its algorithms in a public register.

**Why the metric fits.** For a work coach the costly mistake is the person who is not invited and stays stuck; an extra conversation costs little. Balanced accuracy stops a model from looking good by inviting nobody or everybody, and the threshold in step 9 then trades extra conversations for fewer missed people, with the cost shown.

**Why machine learning beats the simple alternatives.** A coach without a model could invite everyone over 50. On the test set that rule finds only **41%** of the long-term cases; the logistic regression finds **62%** at the same precision (about one in two invitations is needed), and 80% at threshold 0.35. Inviting everyone is impossible with UWV's capacity, and inviting nobody is what happens without early help. The gain is real but modest, which is why we position the model as support for the coach.

## Ethical reflection

The full reflection is in [`ETHICS.md`](ETHICS.md): seven risks specific to this model and this data, each with its consequence for job seekers, what we did, and what is not solved. In short:

- **The model misses the wrong people:** young people (recall 0.29) and men (0.46 against 0.77 for women). We chose balanced accuracy, lowered the threshold to 0.35, check recall per group, and made it a rule that a score can only add help.
- **Migration background** is left out: it did not improve the model (0.633 with, 0.640 without). The other features still reveal origin (ROC-AUC 0.68), so recall per origin group is a standing check.
- **Health** is special-category data. It adds little overall but helps find people with a long-standing disease (recall 0.68 → 0.79), so we keep it only if answering is voluntary and declining can never lower someone's chance of an invite.
- **Privacy:** the LISS data is not in this repository, the notebook shows no individual people, groups under 10 are hidden and no model is saved.

## Dataset card

| | |
|---|---|
| **Source** | LISS panel (Longitudinal Internet studies for the Social Sciences), [LISS Data Archive](https://www.dataarchive.lissdata.nl/); download instructions in [`data/README.md`](data/README.md) |
| **Collected by** | Centerdata (Tilburg University, The Netherlands) |
| **How** | Online questionnaires completed by a panel of about 5,000 households drawn as a probability sample from the population register by CBS; households without a computer or internet get one. The household's contact person updates the *Background Variables* (including everyone's main activity) every month; the core studies *Work and Schooling* and *Health* are asked once a year |
| **When** | Background Variables: November 2007 to March 2026 (November 2022 is missing). Work and Schooling: 18 waves, 2008–2025. Health: 18 waves, 2007–2025 |
| **Licence** | Free for scientific research after registering and signing the *statement on the use of LISS data*; the data may not be passed on. Publications acknowledge the LISS panel (see `data/README.md`) |
| **Size** | Raw: 2,424,987 person-months of 34,301 people; 101,691 Health and 104,288 Work and Schooling interviews. After building job losses (paid work → job seeking, outcome known after 12 months, aged 18–66): **2,019 rows** (one per job loss) of 1,455 people; 22 features (10 numeric, 12 categorical) + migration background for fairness checks only |
| **Target** | `long_term`: not back in paid work (LISS `belbezig` 1–3) in the eleven months after the first month of job seeking. 1 = yes (**39.1%**), 0 = no (60.9%) |
| **Known limitations** | (1) *Job seeker following job loss* is the main activity reported by the household, not WW receipt. (2) Job and health features come from the latest interview in the 24 months before the job loss: missing for about 30% of job losses, and they may describe an earlier job. (3) 74 job losses were dropped because the person left the panel before the outcome was known, which may not be random. (4) Because November 2022 is missing, job losses in November and December 2022 are partly not detected. (5) 352 people have more than one job loss; we split by person. (6) Only people who answer Dutch questionnaires. (7) 2008–2025 includes the financial crisis and COVID; chances in a new period may differ. (8) The analysis is unweighted, so it describes the panel, not exactly the Dutch population |

## How to run

The LISS data cannot be downloaded by a script, so the notebook only runs with your own copy:

1. Request access at the [LISS Data Archive](https://www.dataarchive.lissdata.nl/) and sign the statement on the use of the data.
2. Download in **Stata (.dta) format, English version**: *Background Variables* (all months), *Work and Schooling* (all waves) and *Health* (all waves), and put them – zipped or unzipped – in `data/liss/background/`, `data/liss/work_schooling/` and `data/liss/health/` ([details](data/README.md)). The notebook unzips them itself.
3. **Locally:** `pip install pandas numpy scikit-learn matplotlib notebook` (or `ipykernel` for VS Code), open `uwv_long_term_unemployment.ipynb` from the repository folder and choose *Restart & Run All*. Reading the 220 monthly files takes a few minutes.
4. **In Google Colab:** put the same folders in your own Google Drive under `MyDrive/liss/` (do not share that folder), open the notebook in Colab and choose *Runtime → Run all*; the notebook asks to mount your Drive. No installs needed.

Built and run with Python 3.12, pandas 3.0, numpy 2.5, scikit-learn 1.9 and matplotlib. `random_state=42` is used everywhere; other package versions can shift numbers in the third decimal.

## How we used the tool

scikit-learn does every modelling step: `StratifiedGroupKFold` for the person-grouped test split and the cross-validation folds, `ColumnTransformer` with `SimpleImputer`, `StandardScaler` and `OneHotEncoder` inside a `Pipeline` so nothing is fitted on the test data, `DummyClassifier` and a home-made age rule as baselines, `GridSearchCV` to tune `KNeighborsClassifier`, `LogisticRegression` and `RandomForestClassifier` on the same folds and metric, `cross_val_predict` for the threshold and the fairness experiments, and `sklearn.metrics` for the confusion matrices and scores. Without it there is no model to compare and no prediction for the work coach.

## What we learned

### [Lucas Jansze]

*TODO: write in your own words.*

### [Minhyeok Sung]

*TODO: write in your own words.*

## Sources

- LISS panel – [LISS Data Archive](https://www.dataarchive.lissdata.nl/), Centerdata (Tilburg University, The Netherlands)
- UWV (2026) – [Sociale zekerheid in 2025](https://www.uwv.nl/nl/publicaties/jaar-en-tertaalverslagen/2025/sociale-zekerheid-in-2025-stijging-wia-instroom-vlakt-af)
- UWV – [Werkverkenner (algorithm register)](https://www.uwv.nl/nl/over-uwv/algoritmeregister-uwv/werkverkenner)
- UWV (2021) – [Effect of personal services on job chances and WW duration](https://www.uwv.nl/nl/publicaties/kennis/2021/effect-persoonlijke-dienstverlening-op-werkkans-en-ww-duur)
- UWV – [Kennisverslag 2015-3 (work-orientation conversation)](https://www.uwv.nl/assets-kai/files/6fc77bdf-5d12-4389-af70-84c93e1e95c0/uwv-kennisverslag-ukv-2015-3.pdf)
- CBS (2025) – [Werklozen iets langer op zoek naar werk](https://www.cbs.nl/nl-nl/nieuws/2025/38/werklozen-iets-langer-op-zoek-naar-werk)
- Universiteit Leiden (2024) – [Werklozen verplichten breder naar werk te zoeken pakt vaak averechts uit](https://www.universiteitleiden.nl/nieuws/2024/01/werklozen-verplichten-breder-naar-werk-te-zoeken-pakt-vaak-averechts-uit)
- District Court of The Hague – SyRI judgment, 5 February 2020 (ECLI:NL:RBDHA:2020:865)
- scikit-learn – [documentation](https://scikit-learn.org/stable/)
