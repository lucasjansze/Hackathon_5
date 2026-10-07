# Prompt for the Hackathon 5 presentation

Paste everything below the line into the assistant, in a session that can read this repository.

---

Make the presentation for our Hackathon 5 project as a PowerPoint file (.pptx, 16:9), saved in the repository root as `Who_needs_early_help_Hackathon5_presentation.pptx`.

## Read these first

- `Week5_Hackathon.html`: the assignment. Look at the section "Presentation" and the table that explains each rubric criterion for this hackathon.
- `Hackaton Rubric.html`: the rubric the teachers score with.
- `README.md` and `ETHICS.md`: every number and source on the slides must come from these two files. Do not invent or round differently.
- `uwv_long_term_unemployment.ipynb`: the notebook. Use its saved outputs if you need a number that is not in the README.
- `docs/figures/`: `confusion_matrices.png`, `share_long_term_by_group.png`, `coefficients.png`.

## What the presentation must contain

The assignment asks for five things. All five must be easy to find:

1. The problem and the user.
2. The comparison table.
3. The confusion matrix of the model we recommend.
4. One live prediction.
5. The one decision in our pipeline we would defend hardest.

The teachers ask questions straight from the rubric, so every slide shows in a small label which criterion it answers (for example "RUBRIC 2 · USER GROUP" or "K2 · SDG RELEVANCE"), the same way our earlier decks did.

## Time and length

At most 8 slides. No backup slides and no separate closing slide. The whole presentation, including the live prediction, is at most 10 minutes. Aim for 9:20 so there is room for a slow start. Two speakers: Lucas Jansze and Minhyeok Sung. Put the speaker and the target time in the notes of every slide.

| # | Slide | Rubric | Time | Speaker |
|---|---|---|---|---|
| 1 | Title | | 0:15 | Lucas |
| 2 | The problem and SDG 8.5 | 1, K2 | 1:20 | Lucas |
| 3 | The user group | 2 | 1:00 | Minhyeok |
| 4 | From raw data to a prediction | 3, K1 | 0:50 | Minhyeok |
| 5 | The comparison table | 3, 5 | 1:10 | Lucas |
| 6 | Confusion matrix, who the model misses, and the live prediction | 5, 6 | 2:30 | Lucas, then Minhyeok for the live part |
| 7 | Does it fit, and what we defend hardest | 4 | 1:00 | Minhyeok |
| 8 | Ethical reasoning and closing | 6 | 1:15 | Lucas |

Change the speaker split only if we ask.

## Slide by slide

**1. Title.** "Who needs early help?" with one line underneath: a scikit-learn model that predicts, at the moment someone loses their job, whether they are still without work a year later. Names, "AI for Good · Hackathon 5: Model Showdown · SDG 8 · scikit-learn".

**2. The problem and SDG 8.5.** Left, about two thirds of the slide: what goes wrong, for whom, where and when, in four short blocks. Then three large numbers with their source: 191,500 running WW benefits at the end of 2025 (UWV, 2026), about 70% predicted correctly by UWV's Werkverkenner so three in ten are not (UWV, 2018), and the seventh month as the first conversation for people with a good score (UWV, 2022). Right, one third: SDG 8 with target 8.5 quoted in one line, and two points on why we fit. It is about getting people back into work, and "for all" is why we check the model per group. Name who benefits: job seekers who would otherwise be seen too late. Our three questions from the README go in the notes, not on the slide.

**3. The user group.** Three columns.
- Who uses it: the UWV work coach who handles new WW claimants. Use the numbers from the README: 179,840 new claimants in the year UWV studied, 41,202 with a low score and 90,689 with a high score, more than 90% of low scorers and 52% of high scorers get a conversation, three in ten high scorers are invited earlier on the coach's own judgement.
- Who it is about: people aged 18 to 66 who just lost their job, with the share still without work after a year per age group (25%, 36%, 53%).
- Not for, and who is missing: employers, fraud and enforcement teams, municipalities; and people who do not read Dutch well are hardly in the data.

Add one sentence on why this user needs this kind of model: a coach has to be able to explain a score and to disagree with it.

**4. From raw data to a prediction.** A horizontal flow of six or seven steps: LISS monthly files, find job losses (2,019 rows), features from before the job loss (22), split by person (1,614 train, 405 test), pipeline (impute, scale, one-hot), GridSearchCV on the same five folds for KNN, logistic regression and random forest, test once and check per group. Mark where scikit-learn is used. One line on input and output. How we used scikit-learn and AI assistants while building goes in the notes, in case a teacher asks.

**5. The comparison table.** The full table from the README, baselines first: most frequent, age 50 or older, best age cutoff (40), KNN, logistic regression, random forest, with best hyperparameters, cross-validation score with spread, and test precision, recall and F1. Highlight the logistic regression row and the age-40 row. One line below the table carries the message: the best model is only a little better than inviting everyone aged 40 or older (0.620 against 0.602 on the test set).

**6. Confusion matrix, who the model misses, and the live prediction.** Left: the confusion matrix of the logistic regression only (153 correct back in work, 94 extra invites, 60 missed, 98 correct long-term). Crop it from `confusion_matrices.png` or redraw it. Middle: recall per age group on the test set and in cross-validation (18-29: 0.29 and 0.15, 30-49: 0.50 and 0.50, 50-66: 0.85 and 0.85), with one line that the gap between men and women on the test set mostly disappeared on more data. Right: a "Live prediction" panel with a small screenshot of the saved output of step 10 as backup, and what to watch for: the probability, the advice, and the three answers that raise and lower the score most. About one minute on the matrix and the groups, then we switch to the notebook and run step 10 with a made-up job seeker for about 1:30.

**7. Does it fit, and what we defend hardest.** Left: "why not something simpler?" with the honest answer. Against "50 or older" the model finds more people at the same workload (78 against 65). Against "40 or older" it does about as well (114 against 107, for 18 more invitations). What the model adds is a reason a coach can discuss and a threshold that follows capacity. Right: the decision we defend hardest, splitting by person. 352 people lost a job more than once, and a normal split would let the model recognise them.

**8. Ethical reasoning and closing.** A table with three risks, each with the consequence for the job seeker and what we did:
- The model misses young people who get stuck. They wait until month seven. A score may only add help, and a coach can always invite.
- Scoring people on where they were born. We left migration background out and check per origin group. By the same rule gender should come out, which we found at the end.
- Health is private. Answering is voluntary, and declining can never lower the chance of an invitation.

Below the table, one closing line that also answers "would we hand this to UWV?": not as it is, because a model on intake information is a weak tool, so it may only add help. Then "Questions?" and the repository link in small print.

## Style

- Match our earlier decks in `ai4g-portfolio-LucasJansze/Term 1/Week 2, 3 and 4/presentation/`: a rubric label at the top of each slide, a short headline that states the point, a few large numbers, sources in small print at the bottom of the slide, and a slide number.
- Little text. Slides 2, 6 and 8 each carry two or three things, so keep every block to a few words and move the rest to the notes. No full paragraphs on a slide.
- Plain English, short sentences. No em dashes anywhere, on the slides or in the notes. No buzzwords.
- Every number has its source on the same slide, written as in the README, for example "UWV (2022)".
- Use at most two colours next to black and white. The notebook charts use dark blue `#1f4e79` and light blue `#9dc3e6`, so use those.
- Do not show any data about individual people. Only group numbers, as in the notebook.

## Speaker notes

Write notes for every slide in the words we would actually say, about 130 words per minute of speaking time. Keep them simple enough that we can say them without reading. On slides 5, 6 and 7, add the one question a teacher is most likely to ask and a two-sentence answer.

## When you are done

Tell us the total speaking time from the notes, list any number you could not find in the README or the notebook, and list anything you left out to stay within 8 slides and 10 minutes.
