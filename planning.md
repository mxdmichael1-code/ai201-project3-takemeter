TakeMeter Planning

Community

For this project, I chose public restaurant review communities, including Google Maps, Yelp, and TripAdvisor-style review pages. These communities contain a wide range of review styles, from detailed descriptions of food and service to short emotional reactions or simple overall ratings. This makes them well suited for a classification task because the discourse naturally varies in how informative and useful it is.

The goal of the classifier is not to determine whether a restaurant is good or bad, but to classify the type of review a customer has written.

⸻

Label Taxonomy

Label 1: specific_feedback

A review is labeled specific_feedback if it provides concrete details about the restaurant experience, such as food quality, service, wait time, cleanliness, atmosphere, pricing, staff behavior, or location.

Examples:

* “The ramen broth was rich and flavorful, but the noodles were slightly overcooked.”
* “Service was friendly, although we waited nearly 40 minutes for our food.”

⸻

Label 2: general_opinion

A review is labeled general_opinion if it gives an overall judgment of the restaurant without providing meaningful supporting details.

Examples:

* “Great restaurant. I’d definitely come back.”
* “Pretty average place.”

⸻

Label 3: emotional_reaction

A review is labeled emotional_reaction if it primarily expresses emotion, excitement, frustration, humor, exaggeration, or personal feeling rather than providing an evaluation of the dining experience.

Examples:

* “Absolutely AMAZING!!!”
* “Worst place ever. Never coming back.”

⸻

Hard Edge Cases


Case 1

Review:

“The food was amazing but the service was painfully slow.”

Possible labels:

* specific_feedback
* emotional_reaction

Final label:
specific_feedback

Reason:
Although the review contains emotional wording (“amazing”), it also provides concrete and actionable information about the service being slow. According to my labeling rules, reviews with specific details take priority over emotional expression.

⸻

Case 2

Review:

“The food was good.”

Possible labels:

* specific_feedback
* general_opinion

Final label:
general_opinion

Reason:
The review mentions food, but it does not explain what was good about it. Since there are no concrete details that would help another customer or the restaurant, I classified it as a general opinion.

⸻

Case 3

Review:

“Worst experience ever. Never coming back.”

Possible labels:

* general_opinion
* emotional_reaction

Final label:
emotional_reaction

Reason:
Although the review expresses an overall opinion, its primary purpose is to communicate strong frustration rather than provide an evaluation or specific feedback. Therefore, it is best categorized as an emotional reaction.
⸻

Data Collection Plan

I will collect at least 200 public restaurant reviews from Google Maps, Yelp, and TripAdvisor-style review pages.

The dataset will be stored as a CSV with the columns:

* text
* label
* notes

Target distribution:

* approximately 70 specific_feedback
* approximately 65 general_opinion
* approximately 65 emotional_reaction

If one label becomes underrepresented after collecting 200 reviews, I will collect additional reviews that better represent the missing category until the distribution is reasonably balanced.

⸻

Evaluation Metrics

I will evaluate both the zero-shot baseline and the fine-tuned model using:

* Overall Accuracy
* Precision for each label
* Recall for each label
* F1-score for each label
* Confusion Matrix

Accuracy provides an overall measure of performance, but it does not reveal whether the model performs well on every label. Precision and recall help identify whether the model is overpredicting or missing particular categories, while F1-score balances these two measures. The confusion matrix will show which labels are most commonly confused, helping identify weaknesses in the label definitions or training data.

⸻

Definition of Success

The classifier will be considered successful if:

* it performs better than the zero-shot Groq baseline,
* achieves approximately 70% or higher overall accuracy,
* achieves an F1-score of at least 0.65 for each label,
* and most misclassifications occur on genuinely ambiguous reviews rather than obvious examples.

Such performance would make the classifier useful as a simple tool for organizing restaurant reviews based on the type of feedback they contain.

⸻

AI Tool Plan

Label Stress-Testing

Before beginning annotation, I will provide the label definitions and edge-case rules to ChatGPT and ask it to generate 5–10 restaurant reviews that lie near the boundary between two labels. If I cannot confidently classify them, I will refine the label definitions before collecting the full dataset.

Annotation Assistance

I will use ChatGPT to suggest preliminary labels for batches of restaurant reviews. Every suggested label will be manually reviewed and corrected before being included in the training dataset. Any AI-assisted labels will be disclosed in the README.

Failure Analysis

After evaluating the fine-tuned model, I will provide the incorrectly classified reviews to ChatGPT and ask it to identify common error patterns, such as confusion between general opinions and emotional reactions. I will verify these patterns myself before including them in the evaluation report.
