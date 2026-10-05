# Ethical reflection

**Who needs early help? — AI for Good, Hackathon 5: Model Showdown, SDG 8**
*Lucas Jansze & Minhyeok Sung*

Our model scores people at a vulnerable moment, just after they lost their job, and decides who gets scarce personal help first. Below are seven risks specific to this model and this data. For each one we describe the risk, the consequence for the people the model makes predictions about (job seekers aged 18–66 at the start of their unemployment), what we did about it, and what is not solved.

> Alongside this: [README.md](README.md) for the problem, the user and the results, [uwv_long_term_unemployment.ipynb](uwv_long_term_unemployment.ipynb) for every number below (step 8 for the group checks), and [data/README.md](data/README.md) for the data and its licence.

> **The data is not in this repository.** It comes from the LISS panel (Centerdata, Tilburg University) and may not be redistributed. The notebook shows no individual people, only results for groups of at least 10 people.

All numbers are from the test set (405 job losses) unless we say they come from cross-validation on the training set.

---

## 1. The biggest risk: the model misses the wrong people

**The risk.** A false negative is a person who will still be without work after a year, but whom the model predicts will find work quickly. At the default threshold the model misses 60 of the 158 long-term cases in the test set, and these mistakes are not spread evenly. It finds only **29% of the long-term cases aged 18–29**, against 85% for people aged 50–66. It finds **46% of the men** against 77% of the women, although men and women are almost equally often long-term unemployed (37% and 41%). The missed cases are on average eleven years younger than the ones it finds, more often had a flexible contract, and almost never reported poor health. The model has learned that age and health predict long-term unemployment, so a young, healthy person who does get stuck "looks like" someone who will be fine.

**Consequence for the job seeker.** A missed person starts with online services and sees a work coach only after about three months, while their WW runs down. UWV's own evaluation found that personal help raises the chance of work by about 2 percentage points. For a young person a year without work early in their career costs experience and confidence that is hard to make up later. A false positive costs much less: one conversation that turns out not to be needed, as long as it is help and not pressure.

**What we did.** We chose balanced accuracy as the main metric, so "invite nobody" (0.50) cannot look good, and we checked recall for every age band, gender, education level, origin group and for people with and without a long-standing disease. We lowered the advised threshold from 0.5 to 0.35, which raises the share of long-term cases found from 62% to 80%, and we show the cost: 72% of people are invited instead of 47%. And we made it a rule in the README and the notebook that a low score may never mean less help: a work coach can always invite someone, and the README names young people and people with an unknown job history as groups the coach should look at regardless of the score.

**Not solved.** We measured the group gaps at threshold 0.5, not at 0.35, and several groups are small (87 young people, 36 second-generation migrants in the test set), so the exact numbers are uncertain. We do not know *why* men are missed so much more often than women. **Next step:** repeat the group check at the chosen threshold and on UWV's own data, and test whether a separate threshold per age band reduces the gap without inviting far more people.

## 2. Migration background, and the features that reveal it

**The risk.** People who migrated to the Netherlands themselves are more often still without work after a year (57%, against 38% for people with a Dutch background). A model that uses origin could steer help to them – but scoring people on where they or their parents were born is exactly what the District Court of The Hague rejected in the SyRI case (2020), and what went wrong in the childcare benefits scandal.

**Consequence for the job seeker.** A person could get a higher or lower score *because of their origin*, something they cannot change. Even when that means more help, it puts them in a category: "invited because you are a migrant" is not an explanation a work coach should ever have to give.

**What we did.** We decided before seeing the result to leave origin out unless it improved the model far beyond the spread over the folds. It did not: balanced accuracy is 0.633 *with* origin against 0.640 *without*. It only moved help around (recall for first-generation migrants from 0.69 to 0.87, for people with a Dutch background from 0.63 to 0.60). So origin is not a feature; we use it only to check fairness.

**Not solved.** Leaving the column out does not make the model blind to origin: the other features predict migration background with a ROC-AUC of 0.68 (0.5 would mean no information), through things like education, household and sector. Second-generation migrants are the group the model finds least often (0.44 on the test set, 0.55 in cross-validation). That is why the recall per origin group is a standing check that has to be repeated with every new version, not a box we ticked once.

## 3. Health is special-category data

**The risk.** Self-rated health, a long-standing disease and how much health hinders daily life are *special categories of personal data* under the GDPR. A job seeker may not want to share them with UWV, and may fear that sharing them is used against them.

**Consequence for the job seeker.** If health raises the score, a person with a chronic illness is more likely to be invited early – which can be good. But if declining to answer were treated as "healthy", people who keep their health private would quietly get less help. And a person who learns their health was part of the score may stop trusting the conversation.

**What we did.** We tested the model with and without health instead of assuming it was needed. Overall it adds little (balanced accuracy 0.640 against 0.628, less than the spread over the folds), but it raises recall for people with a long-standing disease from 0.68 to 0.79: they are more often long-term unemployed (51% against 38%), and health helps the model find them. We kept health in under two conditions, stated in the notebook and the README: answering is voluntary, and when someone declines, UWV scores them both with and without health and uses the *higher* risk, so declining can never reduce help.

**Not solved.** In our data a missing health answer means "did not take part in the Health survey that year"; at UWV it would mean "did not want to say". The model has never seen the second situation, so we cannot say how it behaves there. Whether UWV may ask these questions at intake at all is a legal question we cannot answer; it would need a data protection impact assessment.

## 4. What we do not know about someone counts against them

**The risk.** For about 30% of the job losses there was no Work and Schooling or Health interview in the two years before, so the job and health features are missing. The model learns from the *fact* that information is missing, and in this data that partly reflects who takes part in surveys, not anything about the person's chances.

**Consequence for the job seeker.** 37% of the missed long-term cases had no job interview before, against 20% of the ones the model found. People we know less about are missed more often. At UWV the job history would be known, but a model trained on our data could still treat incomplete files unfairly.

**What we did.** Missing answers become their own category instead of a guessed value, so we can see their effect, and we removed the "missing" columns from the coefficient chart because they describe the survey, not the person. In step 8 we report the share of missed people without a job interview.

**Not solved.** We did not test the model separately on people with complete information. **Next step:** when retraining on UWV data, compare performance for complete and incomplete files.

## 5. The score turns into a decision, or a sanction

**The risk.** A score with two decimals looks objective. A work coach with a full caseload may follow it without thinking ("the model says 0.31, so online services"), and a score of long-term unemployment risk could be reused for things it was never meant for: checking job-search effort, sanctions or fraud detection.

**Consequence for the job seeker.** Under automation bias, the 60 missed people in our test set would never get the early conversation, even when the coach had a good reason to invite them. If the score were used for obligations or sanctions, people predicted to stay unemployed would get more pressure instead of more help – and Leiden University (2024) found that imposed job-search obligations for WW recipients backfire.

**What we did.** We positioned the model as support, not as the decision-maker, in the notebook and the README. We chose a model the coach can explain (the logistic regression, with its coefficients in step 9) over an equally good random forest that cannot be explained, and the README lists who must not use it: employers, fraud and enforcement teams, and municipalities for bijstand clients.

**Not solved.** A README cannot stop anyone from using a model differently. In a real deployment this needs to be in UWV's rules for the tool and in its public algorithm register, and coaches need to know that most of the model's mistakes fall on young people and men.

## 6. Privacy and consent of the people in the data

**The risk.** LISS panel members agreed to take part in scientific research. They did not agree to be shown, and every user of LISS data signs a statement not to pass the data on and not to publish information about individual persons or households.

**Consequence for the people in the panel.** A table with a group of three people, an example "real" case, or a saved model could make someone recognisable to people who know them – for example a household with an unusual combination of age, job and illness.

**What we did.** The data is not in this repository and `data/liss/` is in `.gitignore`. The notebook never shows rows, person numbers or real cases: only counts, percentages, averages and scores for groups, and any group smaller than 10 people is shown as "< 10" without numbers. The case in step 10 is made up. We do not save trained models, because a KNN model contains its training rows. We describe the missed cases in step 8 with an average profile instead of examples.

**Not solved.** Using panel data for a study prototype fits the consent; building a model that decides about real job seekers does not. A production version would need UWV's own data and a legal basis, not LISS.

## 7. The people in the data are not UWV's job seekers

**The risk.** Our population is LISS panel members who went from paid work to "job seeker following job loss" between 2007 and 2025. Not all of them claimed WW, the questionnaires are in Dutch, people who leave the panel during their unemployment are dropped (74 job losses), and the period includes the financial crisis and COVID.

**Consequence for the job seeker.** A model can be accurate for the panel and wrong for the people UWV sees: people who do not speak Dutch well are hardly in our data, and they are exactly a group UWV worries about. Their scores would be the least reliable, without anyone noticing.

**What we did.** We say in the README and the dataset card who is missing, and that the model must not be used for people outside the population it was trained on. We recommend retraining on recent UWV data of actual WW claimants before any real use.

**Not solved.** We cannot measure how well the model works for groups that are not in the data.

## Would we hand this model to UWV?

Not as it is. It beats the simple alternatives – it finds 62% of the long-term unemployed against 41% for "invite everyone over 50" – and it is honest about its limits, but roughly four in ten people in each group are classified wrongly, and the errors fall hardest on young people and men. We would hand UWV the *approach*: a model a coach can explain, a threshold chosen for capacity, health only with consent, origin left out, a per-group check on every new version, and a rule that a score can only add help.
