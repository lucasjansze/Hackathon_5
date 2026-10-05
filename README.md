# Who needs early help? Predicting long-term unemployment for UWV

**Hackathon 5 – Model Showdown · SDG 8 Decent Work and Economic Growth · De Haagse Hogeschool**
Team: *[Name 1] & [Name 2]*

Notebook: [`uwv_long_term_unemployment.ipynb`](uwv_long_term_unemployment.ipynb) · Data: [`data/`](data/README.md) · Variables: [`docs/liss_overview.md`](docs/liss_overview.md) · Plan: [`docs/Plan_UWV_Hackathon5.docx`](docs/Plan_UWV_Hackathon5.docx)

> **Privacy.** This project uses LISS panel data, which may not be redistributed. The data is **not** in this repository, and the notebook shows no individual people: only counts, percentages, averages and model scores for groups, with groups smaller than 10 people hidden. See [section 7](#7-ethical-reflection) and the first cells of the notebook.

> **Status.** Sections marked *✏️ after the final run* are filled in with the numbers of the notebook once it has been run on the LISS data and pushed with its outputs.

---

## 1. The problem

When people in the Netherlands lose their job, most of them claim unemployment benefit (WW) from UWV. At the end of December 2025 there were **191,500 running WW benefits**, almost 10% more than a year earlier ([UWV, Sociale zekerheid in 2025](https://www.uwv.nl/nl/publicaties/jaar-en-tertaalverslagen/2025/sociale-zekerheid-in-2025-stijging-wia-instroom-vlakt-af)). CBS reported in September 2025 that unemployed people are searching for work slightly longer than before ([CBS, Werklozen iets langer op zoek naar werk](https://www.cbs.nl/nl-nl/nieuws/2025/38/werklozen-iets-langer-op-zoek-naar-werk)).

Long-term unemployment, meaning a year or more, matters because it becomes self-reinforcing: skills and networks fade, and employers become hesitant. It also has a hard financial edge. WW lasts at most 24 months, and after that comes *bijstand* (social assistance), where people first have to use up their savings and their partner's income counts.

UWV tries to prevent this with personal help, but counsellor time is limited. At the start of every WW benefit, UWV's algorithm **Werkverkenner** estimates the chance that the person is back at work within 12 months. People below 50% are invited early for a face-to-face conversation with a work coach. The others start with online services and are seen in person after about three months ([UWV algorithm register](https://www.uwv.nl/nl/over-uwv/algoritmeregister-uwv/werkverkenner)).

The help works, but modestly. UWV's own evaluation found that personal service raises the chance of work by about **2 percentage points** within a year, with about €201 million in societal benefits against €97 million in costs ([UWV 2021](https://www.uwv.nl/nl/publicaties/kennis/2021/effect-persoonlijke-dienstverlening-op-werkkans-en-ww-duur)). So it matters *who* gets that early conversation.

**Our question:** at the moment someone loses their job, can Dutch panel data – using only what is known at that moment – predict who will **still not be back in paid work twelve months later**? And does such a model make the same mistakes for everyone?

**Does our data population match the people we want to use the model for?** Better than most survey data, but not exactly, and we say so openly. UWV's population is WW claimants at the start of their benefit. Our data are members of the LISS panel who, in the monthly data between November 2007 and 2025, went from paid work in one month to *job seeker following job loss* in the next: **2,019 job losses** of people aged 18–66. Because LISS follows the same people every month, we see the job loss *when it happens* and measure all characteristics *before* it – exactly UWV's intake moment – and we see what happens in the twelve months after. What does not match: not everyone who loses a job claims WW (for example after a very short job), and LISS records someone's main activity as reported by the household, not their benefit. Section 8 lists what this means.

## 2. User group

**Who uses the prediction.** The users are **UWV work coaches** (*werkcoaches*) who run intake for new WW claimants, and the UWV team that decides how many people can be invited early (the threshold). Work coaches are professionals with a full caseload. They need a short, explainable signal at intake, *"this person has a high risk, invite early"*, and they need to be able to explain that signal to the job seeker. They use it at one specific moment: the first weeks of a WW benefit, before the regular three-month conversation.

**Who the prediction is about.** The people affected are **job seekers aged 18–66 who have just lost their job**. In our data, 23% of the job losses are of people aged 18–29, 43% of people aged 30–49 and 34% of people aged 50–66. **39%** were still not back in paid work twelve months later. That share rises steeply with age: 25% for 18–29, 36% for 30–49 and 53% for 50–66. Women are slightly more often long-term (41%) than men (37%), lower-educated people (44%) more often than higher-educated people (33%), people who migrated to the Netherlands themselves (57%) much more often than people with a Dutch background (38%), and people with a long-standing disease (51%) more often than people without (38%).

**Who is *not* the intended user.**
- **Employers and recruiters.** They must never use a risk score to screen applicants.
- **Fraud and enforcement teams** (inside or outside UWV). A risk score of long-term unemployment says nothing about fraud. Leiden University found that imposing job-search obligations on WW recipients backfires ([Universiteit Leiden, 2024](https://www.universiteitleiden.nl/nieuws/2024/01/werklozen-verplichten-breder-naar-werk-te-zoeken-pakt-vaak-averechts-uit)).
- **Municipalities, for bijstand clients.** That is a different population (people who are already long out of work), and our model was not built or tested for it.

**Who is missing from the data.** LISS questionnaires are in Dutch, so people who do not read Dutch well are hardly represented, and they are exactly a group UWV worries about. LISS gives a computer and internet connection to households without one, so people without internet are included, but people who do not want to take part in a monthly panel are not – and people who leave the panel during their unemployment are dropped (74 job losses), which may not be random. People under 18 or over 66, people living in institutions and undocumented workers are not in the data at all.

## 3. Why this fits this user group

The work coach already makes this decision, with an algorithm and a threshold, so we don't add a new step to their work. We give them a model they can **check and explain**. The logistic regression shows *which* characteristics raise the risk, so a coach can say *"you are invited early mainly because…"*, which a black-box score cannot do. That matters for the job seeker's trust, and it fits UWV's own choice to publish its algorithms in a public algorithm register.

The simple alternative a coach could use without any model is *"invite everyone over 50"*; the notebook tests it as a baseline next to "invite nobody" (`DummyClassifier`). Inviting everyone is impossible with UWV's capacity. On the test set the age rule finds only 41% of the long-term cases; the logistic regression finds 62% at the same precision (about one in two invites is needed). *(✏️ after the final run: what the recommended threshold adds.)* Even if machine learning beats these rules, we position the model as *support* for the coach, never as the decision-maker (section 6).

## 4. SDG 8

**Target 8.5** is *full and productive employment and decent work for all women and men, including young people and persons with disabilities*. **Indicator 8.5.2** is the unemployment rate. Our model is about one concrete part of that target: preventing a short spell without work from becoming a long one, by getting scarce help to the right people in their first weeks of unemployment.

## 5. The solution, step by step

**Input:** characteristics of one job seeker that are known at intake. These are age, gender, education, household (size, children, partner, domestic situation), urbanity, net income in the last job, how many months they were looking for work in the three years before, and about the job they lost: contract type, public or private organisation, contract hours, years with the employer, sector, occupation, supervising others and firm size. Health before the job loss (self-rated health, a long-standing disease, how much health hinders daily activities) is included and tested separately (step 8c).
**Output:** the probability that the person is still not back in paid work after 12 months, plus an advice: *invite early* or *online services first*.

The notebook does the following, in order. Each step has a markdown cell explaining what we did and why.

0. **Read the raw LISS files** (zipped or unzipped): the 220 monthly Background Variables files and every wave of the core studies *Work and Schooling* and *Health*. Only the columns we use are read.
1. **Frame the problem.** Positive class = not back in paid work 12 months after the job loss. A false negative (missing someone who needs help) is worse than a false positive (one extra conversation). The main metric is **balanced accuracy**: "invite nobody" and "invite everyone" both score 0.5 on it, while accuracy would reward "invite nobody" (most people do find work within a year) and recall would reward "invite everyone". In step 9 we then move the threshold to raise recall.
2. **Explore the data.** A **job loss** is the first month with *job seeker following job loss* directly after a month in paid work. We look at the eleven months after it to set the target, and drop job losses whose outcome is unknown because the person left the panel. Features are taken **only from before** the job loss: the last month in work, the 36 months before it, and the most recent *Work and Schooling* and *Health* interview in the 24 months before. Codes for *don't know* and impossible values (a 0- or 90-hour working week, a start year in the future) become missing values. We check duplicates and find that **people can lose a job more than once**, which decides the split. Everything measured after the job loss is left out on purpose (it describes the outcome), and migration background is kept *out* of the model and used only for fairness checks.
3. **Split first.** A stratified 80/20 split, **grouped by person** (`StratifiedGroupKFold`), with `random_state=42`, before any pre-processing. No person appears in both training and test data.
4. **Pipeline.** A `ColumnTransformer` handles numeric columns (median imputer with a *was missing* indicator, then `StandardScaler`) and categorical columns ("missing" as its own category, then one-hot encoding with categories of fewer than 10 people grouped together). It is fitted on training data only.
5. **Baselines.** `DummyClassifier(most_frequent)` and the rule "invite everyone aged 50 or older".
6. **Tuning.** `GridSearchCV` with the same 5 person-grouped stratified folds and balanced accuracy for all three models. KNN: `n_neighbors`, `weights`. Logistic regression: `C`, `class_weight`. Random forest: `max_depth`, `min_samples_leaf`, `class_weight`. The notebook plots the score against each main hyperparameter.
7. **Test once.** Confusion matrices, precision, recall, F1 and balanced accuracy, with train, CV and test scores side by side.
8. **Errors and groups.** The average profile of the missed long-term cases against the found ones, recall per age band, gender, education, migration background and long-standing disease, a with/without migration-background experiment with a proxy check, and a with/without health experiment.
9. **Recommendation and threshold.** The threshold is chosen from cross-validated probabilities on the training set only.
10. **Use it.** A made-up job seeker gets a probability and an advice.

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

## 6. Recommendation

**We recommend the logistic regression, as support for the work coach and not as the decision-maker.** In cross-validation it is as good as the random forest (0.640 ± 0.018 against 0.642 ± 0.022: a difference of 0.002, far smaller than the spread over the folds). It overfits much less (the forest scores 0.85 on its own training data), it holds up better on the test set (0.620 against 0.593), and a work coach can explain its prediction to the job seeker. KNN is clearly weaker and on the test set even below the age rule. We decided this rule – *equal in cross-validation, then the explainable model* – before looking at the test set.

**Is it good enough? Not to decide alone.** A balanced accuracy of 0.62 means that in both groups roughly four in ten people are classified wrongly. *(✏️ after the final run: the threshold advice from step 9a and which groups the model misses, from step 8.)*

**What holds whatever the numbers turn out to be:**
- The score may only **add** people to the early-invitation list. A low score must never mean less help.
- It must never be used for obligations, sanctions or fraud detection.
- Before any real use it must be retrained on recent UWV data of actual WW claimants, with the subgroup check repeated each time.

Our results cannot be compared with UWV's "70% correct". That figure is accuracy, on a different population, with a questionnaire we don't have.

## 7. Ethical reflection

This model scores people at a vulnerable moment, just after losing their job, and decides who gets help first. A false negative means waiting three months or more for personal help while their WW runs down, and the evidence says that help raises their chance of work. A false positive costs a work coach one conversation, and costs the job seeker little, *as long as the conversation is support and not pressure*.

**Unequal errors.** A model that learns that age predicts long-term unemployment will tend to miss young people who *do* get stuck: they "look like" someone who will find work quickly. Step 8 measures recall per age band, gender, education, migration background and long-standing disease. *(✏️ after the final run: which groups the model misses most, with the numbers.)*

**Migration background.** Scoring people on where they or their parents were born is exactly what the District Court of The Hague rejected in the SyRI case (2020) and what went wrong in the childcare benefits scandal. We decided *before* seeing the result to leave it out unless it would improve the model far beyond the spread over the folds, and we test it (step 8b). Removing the column is not enough, though: the proxy check measures how well the remaining features still reveal origin, so we **report recall per origin group** as a standing fairness check instead of assuming the model is blind. *(✏️ after the final run: the with/without result and the proxy ROC-AUC.)*

**Health.** Health is a special category of personal data under the GDPR. Health problems can make finding work harder, so using it could steer early help to people who need it – but a work coach may only use it with a good reason, and a job seeker may not want to share it. We test the model with and without health (step 8c) and report recall for people with a long-standing disease. *(✏️ after the final run: what health adds, and the decision.)*

**Privacy and consent.** LISS panel members agreed to take part in scientific research and are identified only by an encrypted number; users of the data sign a statement not to pass it on and not to publish information about individual people. So: the data is not in this repository (`data/liss/` is in `.gitignore`); the notebook never displays rows, person numbers or real cases; groups smaller than 10 people are hidden in every table; the example in step 10 is made up; and we do not save trained models, because a KNN model contains its training rows. Using panel data to build a *study prototype* fits the consent given; a production model at UWV would need UWV's own data and legal basis.

**What we did about these risks:**
- We chose balanced accuracy as the main metric, so that "invite nobody" or "invite only older people" cannot look good.
- We lower the threshold to reduce missed people, and show the cost (step 9a).
- We split by person, so the model cannot score well by recognising people it has already seen.
- We left migration background out and check recall per group, including a proxy check.
- We tested whether health is needed instead of assuming it.
- We set the rule that the score may only add help, never remove it, and must never be used for sanctions.

**This model must not be used:** to decide about people outside the population it was trained on (people under 18 or over 66, bijstand clients, people who never had a job); by employers; or for fraud detection or sanctions.

## 8. Dataset card

| | |
|---|---|
| **Source** | LISS panel (Longitudinal Internet studies for the Social Sciences), [LISS Data Archive](https://www.dataarchive.lissdata.nl/); download instructions in [`data/README.md`](data/README.md) |
| **Collected by** | Centerdata (Tilburg University, The Netherlands) |
| **How** | Online questionnaires completed by a panel of about 5,000 households drawn as a true probability sample from the population register by CBS; households without a computer or internet get one. The household's contact person updates the *Background Variables* (including everyone's main activity) every month; the core studies *Work and Schooling* and *Health* are asked once a year |
| **When** | Background Variables: November 2007 to March 2026 (one month, November 2022, is missing). Work and Schooling: 18 waves, 2008–2025. Health: 18 waves, 2007–2025 |
| **Licence** | Free for scientific research after registering and signing the *statement on the use of LISS data*; the data may not be passed on to others. Publications must acknowledge the LISS panel (see `data/README.md`) |
| **Size** | Raw: 2,425,135 person-months of 34,301 people; about 5,000–7,000 respondents per core-study wave. After building job losses (from paid work to job seeking, outcome known after 12 months, aged 18–66): **2,019 rows** (one row per job loss); 22 features (10 numeric, 12 categorical) + migration background for fairness checks only |
| **Target** | `long_term`: not back in paid work (LISS `belbezig` 1–3) in the eleven months after the first month of job seeking. 1 = yes (**39%**), 0 = no (61%) |
| **Known limitations** | (1) *Job seeker following job loss* is the main activity reported by the household, not WW receipt. (2) Job and health features come from the most recent interview in the 24 months before the job loss: they are missing for about 30% of job losses, and may describe an earlier job if someone changed jobs. (3) 74 job losses were dropped because the person left the panel before the outcome was known, which may not be random. (4) Because November 2022 is missing, job losses in November and December 2022 are partly not detected. (5) One person can have several job losses; we split by person. (6) Only people who answer Dutch questionnaires. (7) 2008–2025 includes the financial crisis and the COVID period; chances in a new period may differ. (8) The analysis is unweighted, so it describes the panel, not exactly the Dutch population. |

## 9. How to run

The LISS data cannot be downloaded by a script, so the notebook does not run until you have your own copy:

1. Request access at the [LISS Data Archive](https://www.dataarchive.lissdata.nl/) and sign the statement on the use of the data.
2. Download, in **Stata (.dta) format, English version**: *Background Variables* (all months), *Work and Schooling* (all waves) and *Health* (all waves). Put them, zipped or unzipped, in `data/liss/background/`, `data/liss/work_schooling/` and `data/liss/health/` (details in [`data/README.md`](data/README.md)). The notebook unzips them itself.
3. Optional: `python scripts/liss_inventory.py` writes an overview of all variables and a count of job losses (already done: [`docs/liss_overview.md`](docs/liss_overview.md)).
4. **Locally:** `pip install pandas numpy scikit-learn matplotlib notebook`, then open `uwv_long_term_unemployment.ipynb` from the repository folder and choose *Restart & Run All*. Reading the 220 monthly files takes a few minutes.
5. **In Google Colab:** put the same folders in your own Google Drive under `MyDrive/liss/` (do not share that folder), open the notebook in Colab and choose *Runtime → Run all*. The notebook asks for permission to mount your Drive. No installs needed.

Built and tested with Python 3.11, pandas 3.0, numpy 2.4, scikit-learn 1.9 and matplotlib. The notebook only uses packages that Colab has by default. `random_state=42` is used everywhere. Other package versions can shift numbers in the third decimal.

## 10. Sources

- LISS panel – [LISS Data Archive](https://www.dataarchive.lissdata.nl/), Centerdata (Tilburg University, The Netherlands)
- UWV – [Werkverkenner (algorithm register)](https://www.uwv.nl/nl/over-uwv/algoritmeregister-uwv/werkverkenner)
- UWV – [Effect of personal services on job chances and WW duration (2021)](https://www.uwv.nl/nl/publicaties/kennis/2021/effect-persoonlijke-dienstverlening-op-werkkans-en-ww-duur)
- UWV – [Sociale zekerheid in 2025](https://www.uwv.nl/nl/publicaties/jaar-en-tertaalverslagen/2025/sociale-zekerheid-in-2025-stijging-wia-instroom-vlakt-af)
- UWV – [Kennisverslag 2015-3 (work-orientation conversation)](https://www.uwv.nl/assets-kai/files/6fc77bdf-5d12-4389-af70-84c93e1e95c0/uwv-kennisverslag-ukv-2015-3.pdf)
- CBS – [Werklozen iets langer op zoek naar werk (2025)](https://www.cbs.nl/nl-nl/nieuws/2025/38/werklozen-iets-langer-op-zoek-naar-werk)
- Universiteit Leiden – [Werklozen verplichten breder naar werk te zoeken pakt vaak averechts uit (2024)](https://www.universiteitleiden.nl/nieuws/2024/01/werklozen-verplichten-breder-naar-werk-te-zoeken-pakt-vaak-averechts-uit)
- District Court of The Hague – SyRI judgment, 5 February 2020 (ECLI:NL:RBDHA:2020:865)
- scikit-learn – [documentation](https://scikit-learn.org/stable/)
