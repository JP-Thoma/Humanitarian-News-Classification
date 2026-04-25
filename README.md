# Humanitarian-News-Classification
Using the public NewsAPI to retrieve and classify articles related to humanitarian and medical contexts in countries where MSF operates.

## Overview

This project implements a simple and reproducible Python-based data pipeline to retrieve, structure, and enrich news articles related to humanitarian and medical contexts. The goal is to produce a clean dataset aligned with regions where Médecins Sans Frontières (MSF) operates, and to classify articles into relevant thematic categories.

## Approach
### Data Retrieval

Articles are retrieved using the NewsAPI “everything” endpoint via the official Python client. A composite keyword query is applied to ensure that results capture both medical and humanitarian contexts. Specifically, the query combines terms such as “health”, “disease”, “outbreak” with “crisis”, “conflict”, “aid”, “humanitarian”.

To ensure relevance and manageability, results are limited to:

English-language articles
A recent timeframe (from = 2026-04-15)
Articles sorted by publication date

The API key is securely loaded from a .env file to avoid exposing credentials in the repository.

### Relevance Filtering (MSF Context)

As the NewsAPI does not provide country-level filtering for the “everything” endpoint, relevance to MSF operations is ensured through post-processing.

A predefined list of countries where MSF operates—extracted from the official MSF “Where We Work” page (https://www.doctorswithoutborders.org/what-we-do/where-we-work) is used to filter and annotate articles. Articles are retained only if at least one of these countries is explicitly mentioned in the title or description.

This approach ensures that the dataset focuses on regions with active humanitarian medical interventions rather than general global health news.

Data Structuring

The retrieved JSON response is normalised into a tabular format using pandas. Key fields extracted include:

Source
Author
Title
Description
URL
Publication date

Basic cleaning steps are applied, including duplicate removal and filtering of incomplete records.

### Classification Logic

Article classification is performed using a zero-shot classification model from Hugging Face:

facebook/bart-large-mnli

This model assigns each article to one of the following categories:

Health
Conflict
Humanitarian Aid
Displacement
Other

The zero-shot approach allows classification without task-specific training by evaluating the semantic similarity between the article text and candidate labels. This keeps the implementation simple, transparent, and adaptable.

### Sentiment Analysis

Sentiment is derived using a lightweight pre-trained model:

distilbert-base-uncased-finetuned-sst-2-english

Outputs are mapped to:

Positive
Negative
Neutral (approximated using a confidence threshold)

This provides an additional, high-level signal regarding the tone of reporting.

## Output

The final dataset is exported in CSV and/or JSON format, including:

Extracted article fields
Detected country
Assigned category
Sentiment label
Limitations

This implementation prioritises simplicity and transparency over completeness:

- Keyword-based retrieval may miss news articles and add irrelevant articles
- Country detection relies on explicit mentions and may miss regional references (e.g. “Sahel”, “West Africa”)
- Zero-shot classification may introduce noise or misclassification in ambiguous cases
- Sentiment models are not tailored to humanitarian reporting and may oversimplify nuanced language
- The NewsAPI free tier limits the number and coverage of retrievable articles

## Summary
The pipeline demonstrates a pragmatic approach to collecting and structuring humanitarian news data. It balances simplicity, reproducibility and relevance to MSF operations while remaining transparent about its assumptions and limitations.