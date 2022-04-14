# MedicalTextNegation

The goal of the exercise is identifying ways of negation of anamnesis facts by a patient, in clinical summaries written in Russian.

The code is presented in the python notebook. The data is real-world data from a Russian clinic and is not available for confidentiality reasons.

## Task description
The task description was formulated by the research organization.

The task is to identify the structure of negation related to specific phenomena in the text (negation of illness, a specific state, etc.) with possibility to analyze negation automatically (identify negation forms, the entities which are negated, etc.). The texts are anamneses of the patients with acute coronary syndrome. The task is specific in that it deals with the anamneses collected by clinicians listening to the patients. Thus, a clinician cannot objectively claim that a phenomenon is ‘absent’ (e.g. the patient doesn’t smoke, they haven’t had a heart attack etc.) - only the negation of the phenomenon on the part of the patient can be claimed.

## Annotation of negation in the anamneses dataset
The texts have been annotated in IOB format (wiki description) with a few modifications.

Namely, the following words have been annotated:
- the word denoting the negation itself (e.g., not, without, denies) - neg;
- the words denoting the negated phenomena (e.g., (stroke, kidney disease, заболевания почек, myocardial infarction, etc.) – phen:
  - the first word in the group phen_b;
  - the rest of the words (if present) phen_i.

The annotation was performed with UAM CorpusTool. An example of the annotation procedure is presented below:
![Новый рисунок](https://user-images.githubusercontent.com/2161303/163346891-beaaf3cf-5c1e-4cca-83b7-53427189263d.jpg)

80 documents were annotated, containing 125 negation occurrences. Some documents don’t contain negation, however, are included in the annotated sample, because their text carries relevant information about the absence of negation. In future, a larger collection of documents should be annotated.

_*It would be beneficial to approve a part of the resulting annotation with clinicians, because there are ambiguous cases in the data. A difficult case is a distinction between the patient’s negation and objective data obtained by the clinician. For example, didn’t have anginal pain - appears to be the patient’s negation. However, didn’t have postoperative anginal pain, given that the patient was attended to in a hospital during the postoperative period, could rather mean objective anginal pain manifestations which were not identified by a clinician during a postoperative examination. Some more examples of the ambiguous cases: didn’t receive treatment, without regular antihypertensive therapy, didn’t see a doctor, no anamnesis present._

## Machine learning of the negation structure
Automatic classification is developed and evaluated in negation-crf.ipynb. It is developed in python 3, with the following libraries: BeautifulSoup, numpy, pandas, pymorphy2, scikit-learn and sklearn_crfsuite.

First, the xml-files with the annotation are processed; words are identified which received relevant annotation, and those which did not; the latter words are assigned the ‘O’ annotation tag. Morphological analysis is performed with pymorphy2. The following features are extracted for each word: the orthographic form, the first few letters, the normal form, some orthographic characteristics (case and digits), some morphological characteristics. The trained model is based on conditional random fields (in sklearn_crfsuite) and is aimed at predicting the annotation of each word (O, neg, phen_b, phen_i). 

The experiment is run using three-fold cross-validation by documents (not by words). I only output the results for the relevant classes (neg, phen_b, phen_i), because the ‘O’ class is extremely predominant in the dataset, and using it to average the results would result in overestimating the performance metrics. The evaluation metrics are presented on a word-by-word basis, not by documents.

I also output all the errors of the annotation prediction and the correct cases separately, allowing to analyze errors linguistically.

## Error analysis and further steps
Based on linguistic analysis, the following classes of errors were identified:
- including only parts of noun groups denoting negated clinical phenomena;
- erroneous (?) classification of objectively negated phenomena as those negated by the patient;
- new negation words and constructions are not identified as neg elements in some cases.

The following steps could be taken to improve the performance by addressing the errors:
- Including thesaurus information: add a feature showing if the word is included in a specific thesaurus category:
  - Russian linguistic thesaurus - would allow to identify negation elements and verbs commonly occurring with negation (mostly, verbs of speech);
  - Medical thesaurus - would allow to identify negated clinical phenomena more accurately.
- Including more distant words in the features - +-3, 4 words.
- As an option, include syntactic information: noun, prepositional and coordinate groups, based on rule-based syntactic analysis (for example, natasha lib) or machine learning (e.g., identify syntactic groups via another conditional random fields algorithm). Adding the syntactic analysis option is specifically useful, if more tasks are planned for the dataset, not restricted to negation identification.
- Specifying the distinction between the patient’s negation and the ‘objective’ negation in coordination with clinicians. 

## Further evaluation of the negation identification algorithm
In addition to cross-validation on the annotated dataset, it is worth expanding the annotated dataset iteratively. After the cross-validation and the algorithm improvement phase, a new heldout dataset should be annotated, and the model should be trained on the cross-validation data, and evaluated on the heldout data. When the heldout dataset performance reaches the cross-validated performance, it’s probable that we have included enough data in the training.
## Further iterative work
- Modify algorithm/features/hyper-parameters and repeat the Machine Learning step;
- Annotate an additional subset of documents and evaluate the model on it.

