# Humanitarian-News-Classification
This project implements a simple and reproducible Python-based data pipeline to retrieve, structure, and enrich news articles related to humanitarian and medical contexts relevant to the mission of Médecins Sans Frontières (MSF). The goal is to produce a clean dataset to classify articles into relevant thematic categories and label them with an adequate sentiment.

## Approach
### Data Retrieval
Articles are retrieved using the NewsAPI “everything” endpoint via the official Python client. A composite keyword query is applied to ensure that results capture both medical and humanitarian contexts. Specifically, the query combines terms such as “health”, “disease”, “outbreak” with “crisis”, “conflict”, “aid”, “humanitarian”.

To ensure relevance and manageability, results are limited to:

- English-language articles
- A recent timeframe (from = 2026-04-15, to = 2026-04-25)
- Articles sorted by publication date

The API key is securely loaded from a .env file to avoid exposing credentials in the repository.

### Data Preprocessing
The retrieved JSON response is normalised into a tabular format using pandas. Key fields extracted include:
- source
- author
- title
- description
- url
- publishedAt
- content

Basic cleaning steps are applied, including duplicate removal and filtering of incomplete records. As the NewsAPI does not provide country-level filtering for the “everything” endpoint, a predefined list of countries where MSF operates—extracted from the official MSF “Where We Work” page (https://www.doctorswithoutborders.org/what-we-do/where-we-work) is used to annotate articles. This approach ensures that the dataset holds additional geographic information on active humanitarian medical interventions.

### Classification Logic
Article classification is performed using a zero-shot classification model from Hugging Face: facebook/bart-large-mnli

This model assigns each article to one of the following categories:
- "Conflict-Driven Crisis (war, violence, attacks on healthcare)"
- "Epidemic & Disease-Driven Crisis (cholera, outbreak, infectious disease)"
- "Food & Nutrition-Driven Crisis (famine, malnutrition, starvation)"
- "Displacement-Driven Crisis (refugees, camps, migration)"
- "Environmental Disaster-Driven Crisis (flood, drought, earthquake)"

The zero-shot approach allows classification without task-specific training by evaluating the semantic similarity between the article text and candidate labels. This keeps the implementation simple, transparent, and adaptable.

### Sentiment Analysis
Sentiment is derived using a lightweight pre-trained model: cardiffnlp/twitter-roberta-base-sentiment-latest

Outputs are mapped to:
- Positive
- Negative
- Neutral

This provides an additional, high-level signal regarding the tone of reporting.

## Output
The final dataset is exported in CSV (with the option to export in JSON format), including:
- Extracted article fields
- Detected country
- Assigned category
- Sentiment label
The final dataset is called: humanitarian_news_classification.csv

## Limitations
This implementation prioritises simplicity and transparency over completeness:
- Keyword-based retrieval may miss news articles and add irrelevant articles
- Filtering by English-language articles excludes relevant news in other (local) languages
- Country detection relies on explicit mentions and may miss regional references (e.g. “Sahel”, “West Africa”)
- Article classification is performed using a zero-shot classification model, which assigns categories based on semantic similarity between article text and predefined labels. The labels are designed to reflect primary drivers of medical need (e.g. conflict, disease, displacement), and include descriptive context to improve classification performance. As classification is single-label and based on semantic similarity, articles involving multiple overlapping drivers (e.g. conflict-induced displacement leading to disease outbreaks) may be assigned to only one dominant category.
- Sentiment analysis is performed using a pre-trained transformer model that provides three classes (positive, neutral, negative). This avoids the need for heuristic thresholding and allows neutral sentiment to be explicitly modelled. While the model is trained on social media data, it provides a reasonable approximation for general news text but is not tailored to humanitarian reporting and may oversimplify nuanced language.
- The NewsAPI free tier limits the number and coverage of retrievable articles

## Summary
The pipeline demonstrates a pragmatic approach to collecting and structuring humanitarian news data. It balances simplicity, reproducibility and relevance to MSF operations while remaining transparent about its assumptions and limitations.