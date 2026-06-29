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

Fine-Tuning Setup

Base Model

DistilBERT (distilbert-base-uncased)

Training Configuration

Hyperparameter	Value
Epochs	3
Learning Rate	2e-5
Batch Size	16

The model was fine-tuned using the Hugging Face Transformers library on Google Colab with a T4 GPU.

⸻

Zero-Shot Baseline

Before fine-tuning, a zero-shot baseline was evaluated using a large language model through the Groq API.

The prompt defined the three labels, included examples for each label, and instructed the model to output only one valid label for each review.

This baseline provides a reference point for determining whether task-specific fine-tuning improves classification performance.

⸻

Evaluation Results

Overall Accuracy

Model	Accuracy
Zero-shot Baseline	0.533
Fine-tuned DistilBERT	0.533

Although the overall accuracy remained unchanged, the two models behaved differently at the class level.

Per-Class Metrics

Zero-Shot Baseline

Label	Precision	Recall	F1
specific_feedback	0.57	0.73	0.64
general_opinion	0.45	0.50	0.48
emotional_reaction	0.60	0.33	0.43

Fine-Tuned DistilBERT

Label	Precision	Recall	F1
specific_feedback	0.43	0.91	0.59
general_opinion	0.50	0.10	0.17
emotional_reaction	1.00	0.56	0.71
