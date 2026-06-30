TakeMeter: Restaurant Review Discourse Classifier

Project Overview

TakeMeter is a text classification project that evaluates the style of restaurant reviews rather than simply their sentiment. Instead of predicting whether a review is positive or negative, the model categorizes each review based on the type of discourse it represents. The objective is to determine whether a review primarily provides actionable feedback, expresses an overall opinion, or conveys an emotional reaction.

The project fine-tunes DistilBERT on a manually reviewed dataset of restaurant reviews and compares its performance against a zero-shot Large Language Model baseline.

⸻

Community Choice

I chose online restaurant review communities such as Yelp, Google Reviews, and TripAdvisor because they contain large amounts of short-form user-generated text with highly varied writing styles. Some reviews provide concrete and actionable feedback, while others simply express satisfaction or frustration. These differences make restaurant reviews a suitable domain for evaluating discourse quality rather than sentiment.

Understanding the style of a review could help businesses prioritize which reviews provide useful operational feedback and which primarily reflect customer emotions or general impressions.

⸻

Label Taxonomy

The classifier predicts one of three mutually exclusive labels.

1. specific_feedback

The review contains concrete, actionable information about a particular aspect of the restaurant such as food quality, service, cleanliness, atmosphere, pricing, menu items, or wait time.

Examples

* “The fries were crispy and perfectly seasoned.”
* “Service was very prompt.”

⸻

2. general_opinion

The review expresses an overall judgment about the restaurant but provides little or no supporting detail.

Examples

* “Would definitely come back.”
* “Overall, I enjoyed this restaurant.”

⸻

3. emotional_reaction

The review primarily communicates strong emotion, excitement, disappointment, anger, or exaggeration instead of describing the dining experience in detail.

Examples

* “Worst restaurant ever!”
* “Hands down my favorite Italian restaurant!”

⸻

Data Collection

The dataset was created using a public restaurant review corpus containing real customer reviews.

A balanced subset of 200 reviews was selected and labeled using the taxonomy above.

The final dataset contains:

Label	Count
specific_feedback	70
general_opinion	65
emotional_reaction	65

Each review contains the following fields:

* text
* label
* notes

The notebook automatically split the dataset into:

* 70% Training
* 15% Validation
* 15% Test

⸻

Difficult Labeling Decisions

Some reviews naturally fall near the boundary between two categories.

Example 1

Review

The fries were great too.

Possible labels:

* specific_feedback
* general_opinion

Final decision:

specific_feedback

Although the review is short, it discusses a specific menu item, making it more informative than a general opinion.

⸻

Example 2

Review

All I have to say is the food was amazing!!!

Possible labels:

* specific_feedback
* emotional_reaction

Final decision:

emotional_reaction

The review expresses excitement but provides no concrete explanation about why the food was good.

⸻

Example 3

Review

Service was a little slow.

Possible labels:

* specific_feedback
* general_opinion

Final decision:

specific_feedback

The review identifies a specific aspect of the dining experience that could be acted upon by the restaurant.

⸻
### Data Source

The original reviews came from the public `Restaurant_Reviews.tsv` dataset in this GitHub repository:

https://github.com/sharmaroshan/Restaurant-Reviews-Analysis/blob/master/Restaurant_Reviews.tsv

I used the first 200 reviews from the dataset as my starting point. The original dataset includes sentiment labels (`Liked`), but I did not use those labels for training. Instead, I relabeled the reviews according to my own TakeMeter taxonomy:

- `specific_feedback`
- `general_opinion`
- `emotional_reaction`

_____

Fine-Tuning Setup

Base Model

DistilBERT (distilbert-base-uncased)

Training Configuration

Hyperparameter	Value
Epochs	3
Learning Rate	2e-5
Batch Size	16
I kept the default training setup of 3 epochs, learning rate 2e-5, and batch size 16 because the dataset was small. Training for too many epochs could easily overfit the 140-example training split, while 2e-5 is a standard conservative learning rate for fine-tuning BERT-style models. The model was fine-tuned using the Hugging Face Transformers library on Google Colab with a T4 GPU.

⸻

Zero-Shot Baseline

Before fine-tuning, a zero-shot baseline was evaluated using a large language model through the Groq API.

The prompt defined the three labels, included examples for each label, and instructed the model to output only one valid label for each review.

This baseline provides a reference point for determining whether task-specific fine-tuning improves classification performance.

⸻

# Evaluation Results
## Sample Classifications

| Review | Predicted Label | Confidence | Analysis |
|--------|-----------------|-----------:|----------|
| "Service was very prompt." | `specific_feedback` | 0.91 | **Correct.** This review gives concrete information about the quality of the service, which matches the definition of `specific_feedback`. |
| "Hands down my favorite Italian restaurant!" | `specific_feedback` | 0.35 | **Incorrect.** The model focused on the mention of a restaurant rather than recognizing that the review is primarily expressing enthusiasm. A human would likely classify this as `emotional_reaction`. |
| "Will never, ever go back." | `specific_feedback` | 0.34 | **Incorrect.** This review contains no actionable details and mainly expresses frustration. The model confused strong emotional language with specific feedback. |
| "The fries were crispy and delicious." | `specific_feedback` | 0.88 | **Correct.** The review provides concrete information about a specific menu item, making it an appropriate example of `specific_feedback`. |
| "Worst restaurant ever!" | `emotional_reaction` | 0.82 | **Correct.** The review communicates a strong emotional response without supporting details, matching the intended definition of `emotional_reaction`. |

---

## Reflection

The goal of this project was to distinguish between three different styles of restaurant reviews: reviews that provide actionable feedback, reviews that express a general opinion, and reviews that primarily communicate emotion. Although the fine-tuned model achieved the same overall accuracy as the zero-shot baseline, it learned a noticeably different decision boundary.

The model became much better at identifying **emotional_reaction** reviews, suggesting that strong emotional language is a relatively consistent linguistic pattern that can be learned from a relatively small dataset. However, it struggled to distinguish **general_opinion** from **specific_feedback**. In many cases, the model treated any mention of food, service, or another restaurant-related topic as `specific_feedback`, even when the review functioned mainly as an overall opinion.

This indicates that the model overfit to topic-related keywords instead of learning the higher-level distinction I intended. Rather than identifying whether a review was actionable, it often relied on the presence of restaurant-specific vocabulary. This behavior is also reflected in the confusion matrix, where 9 out of 10 `general_opinion` reviews were misclassified as `specific_feedback`.

Another important observation is that dataset quality influenced the final results. While reviewing the errors, I found several examples whose ground-truth labels could reasonably be interpreted differently under my own label definitions. This suggests that annotation consistency likely limited performance as much as the model itself.

If I were to continue this project, I would refine the definitions of `general_opinion` and `specific_feedback`, collect more examples that explicitly distinguish the two categories, and involve multiple annotators to improve labeling consistency. I believe these improvements would have a greater impact than simply training a larger model.
## Overall Accuracy

| Model | Accuracy |
|-------|---------:|
| Zero-shot Baseline | **0.533** |
| Fine-tuned DistilBERT | **0.533** |

Although the overall accuracy remained unchanged, the two models exhibited different behavior at the class level. While the fine-tuned model did not improve overall accuracy, it shifted its decision boundary and learned some classes better than others.

---

## Per-Class Metrics

### Zero-Shot Baseline

| Label | Precision | Recall | F1 |
|------|----------:|-------:|---:|
| specific_feedback | 0.57 | 0.73 | 0.64 |
| general_opinion | 0.45 | 0.50 | 0.48 |
| emotional_reaction | 0.60 | 0.33 | 0.43 |

### Fine-Tuned DistilBERT

| Label | Precision | Recall | F1 |
|------|----------:|-------:|---:|
| specific_feedback | 0.43 | 0.91 | 0.59 |
| general_opinion | 0.50 | 0.10 | 0.17 |
| emotional_reaction | 1.00 | 0.56 | 0.71 |

Compared with the zero-shot baseline, the fine-tuned model substantially improved performance on the **emotional_reaction** class, increasing its F1 score from **0.43** to **0.71**. However, this came at the expense of **general_opinion**, whose F1 score dropped from **0.48** to **0.17**. The F1 score for **specific_feedback** decreased slightly from **0.64** to **0.59**, while recall increased significantly, indicating that the model predicted this label much more aggressively.

---

## Confusion Matrix

The confusion matrix for the fine-tuned model is shown below.

| True Label | Predicted: specific_feedback | Predicted: general_opinion | Predicted: emotional_reaction |
|------------|----------------------------:|---------------------------:|------------------------------:|
| specific_feedback | 10 | 1 | 0 |
| general_opinion | 9 | 1 | 0 |
| emotional_reaction | 4 | 0 | 5 |

The most common error was predicting **specific_feedback** for reviews whose true label was **general_opinion**. Out of ten `general_opinion` examples, the model correctly identified only one and classified the remaining nine as `specific_feedback`.

This pattern suggests that the model relied heavily on the presence of restaurant-related terms (such as food, service, or menu items) rather than distinguishing between actionable feedback and an overall opinion. In contrast, the model performed much better at identifying **emotional_reaction**, indicating that emotionally expressive language forms a clearer and more consistent pattern for the classifier to learn.

Overall, these results indicate that the distinction between **general_opinion** and **specific_feedback** remains the primary challenge of this classification task and would likely benefit from more consistent annotation and additional training examples focused on boundary cases.
⸻

Error Analysis

Error 1

Review

Hopefully this bodes for them going out of business and someone who can cook can come in.

Ground Truth: general_opinion

Prediction: specific_feedback

Confidence: 0.36

Analysis

This review expresses an overall negative opinion with emotional language rather than providing actionable feedback. The model likely focused on the phrase “someone who can cook” and interpreted it as criticism of food quality. This demonstrates that the model tends to associate references to cooking or food with the specific_feedback class, even when the review’s primary purpose is expressing dissatisfaction.

⸻

Error 2

Review

Hands down my favorite Italian restaurant!

Ground Truth: emotional_reaction

Prediction: specific_feedback

Confidence: 0.35

Analysis

This review contains no concrete information about the restaurant. Instead, it communicates enthusiasm. The model appears to have relied on restaurant-related keywords instead of identifying the sentence as an emotional expression. More examples of short enthusiastic reviews during training would likely improve performance.

⸻

Error 3

Review

However, there was so much garlic in the fondue, it was barely edible.

Ground Truth: general_opinion

Prediction: specific_feedback

Confidence: 0.36

Analysis

This example illustrates a limitation of the dataset itself. According to the project definitions, the review contains concrete information about a specific dish and could reasonably be labeled as specific_feedback. In this case, the model’s prediction is understandable despite being counted as incorrect. This highlights how annotation consistency directly affects evaluation metrics.

⸻

Sample Classifications

Review	Predicted Label	Confidence	Notes
“Service was very prompt.”	specific_feedback	0.91	Correct. The review provides concrete information about service quality.
“Hands down my favorite Italian restaurant!”	specific_feedback	0.35	Incorrect. The model relied on restaurant-related keywords instead of emotional tone.
“Will never, ever go back.”	specific_feedback	0.34	Incorrect. The model struggled to distinguish emotional language from general feedback.
“The fries were crispy and delicious.”	specific_feedback	0.88	Correct. The review discusses a specific menu item and its quality.
“Worst restaurant ever!”	emotional_reaction	0.82	Correct. The review primarily expresses strong emotion without supporting details.

⸻

Reflection

The original goal of this project was to distinguish between actionable restaurant feedback, overall opinions, and emotional reactions. While the model learned to recognize emotional language reasonably well, it struggled to separate specific feedback from general opinions.

Many restaurant reviews are short and naturally mix these two styles. For example, a sentence such as “The fries were great” mentions a specific menu item but also functions as a simple opinion. Because many training examples shared this ambiguity, the model frequently predicted specific_feedback whenever food or service was mentioned.

Another important observation is that some errors appear to result from annotation rather than model behavior. A few reviews labeled as general_opinion arguably fit the definition of specific_feedback based on the project’s own taxonomy. This suggests that improving annotation consistency could improve model performance more effectively than simply collecting additional training data.

Overall, the project demonstrates that label quality is often more important than model architecture when building supervised classifiers.

⸻

Spec Reflection

The project specification strongly influenced the design of the label taxonomy. Spending time defining mutually exclusive labels before collecting data made the annotation process more consistent and highlighted ambiguous cases early.

One way my implementation diverged from the original plan was the use of preliminary AI-assisted labeling followed by manual review. This significantly reduced annotation time while still allowing me to verify labels and refine ambiguous examples before training.

⸻

AI Usage

Label Design

ChatGPT was used to refine the label taxonomy, identify ambiguous boundary cases, and improve the wording of label definitions. Several initial label definitions were revised after discussing examples that could reasonably belong to multiple categories.

Dataset Preparation

AI was used to generate preliminary labels for the restaurant review dataset using the predefined taxonomy. Every example was manually reviewed before training, and ambiguous cases were adjusted according to the final labeling rules.

Error Analysis

After evaluation, AI was used to identify common patterns among the model’s incorrect predictions. The suggested patterns were then verified manually by examining the confusion matrix and individual misclassified reviews.

⸻

Repository Structure

ai201-project3-takemeter/
│
├── planning.md
├── README.md
├── takemeter_dataset_200.csv
├── evaluation_results.json
├── confusion_matrix.png
└── notebook.ipynb

⸻

How to Run

1. Clone the repository.
2. Open the provided Colab notebook.
3. Upload takemeter_dataset_200.csv.
4. Run Sections 1–6 of the notebook.
5. Download the generated evaluation files.
6. Compare the zero-shot baseline with the fine-tuned DistilBERT model.

⸻

Future Improvements

If this project were extended, I would:

* Collect a larger and more diverse dataset.
* Improve annotation consistency by involving multiple annotators.
* Refine the distinction between general_opinion and specific_feedback.
* Experiment with larger transformer models and confidence calibration.
* Build a simple web interface that allows users to classify new restaurant reviews interactively.
