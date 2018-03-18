# Topic Modeling New York Times Best Sellers

## Objective
The purpose of this project was to understand what topics of books people are most interested in reading by analyzing the New York Times Best Sellers lists. The NYT provides limited categorization of their lists by format (hardcover, paperback) and genre (fiction, nonfiction, children's), whereas Amazon classifies their books for sale by over 30 categories.

Finding a more detailed breakdown of best sellers would enable better insights to what topics drive book sales and how that relates to cultural trends. For example, authors and publishers seeking to grow their audiences could use topic-level best sellers data in addition to existing reader segmentation capabilities to improve demand estimation and target their marketing more effectively. As a greater share of book sales transition to electronic formats, it could also offer content-based recommendations of best sellers for readers who reveal their preferred topics or the previous best sellers they enjoyed.

## Tools
- Requests for querying APIs:
  - New York Times Books API for weekly lists of best sellers since 2008, including basic metadata
  - Google Books API for longer text descriptions of the best sellers
- MongoDB for storage of book metadata in JSON format
- Pandas for data manipulation
- NLTK for pre-processing (lemmatization)
- Scikit-learn for tokenizing (count vectorizer, TFIDF) and modeling (SVD)
- Gensim for text modeling (LDA, LSI/LSA)
- Matplotlib and Seaborn for plotting

## Data Acquisition & Merging
Both the NYT and Google APIs are rate limited at 1,000 daily requests, so recurring queries were made across all best seller lists for 8 consecutive days. Each book's primary ISBN-13 from NYT was used as a query parameter for fetching book descriptions from Google. Only about 44% of unique ISBNs from NYT had matching records on Google; however, both data sources provided book descriptions. On average the NYT descriptions were 100 characters in length compared to Google's 600 characters, so the longer text was used where available. Descriptions with fewer than 37 characters were discarded because they were discovered to have irrelevant information. Finally, book titles were combined with descriptions to ensure all relevant term signals were included. Resulting corpus size was 11,327 documents.

## Document Pre-Processing
Excessively common terms and phrases related to the marketing of NYT best sellers needed to be removed from the documents. These were manually identified by inspection of a random sample and calculating counts of the most frequent unigrams and bigrams. Empty or null descriptions were also excluded. WordNet Lemmatizer was used to group inflected or variant forms of the same word, although part of speech was assumed to be noun for all terms, which likely did not group all variants accurately. TFIDF was used with Latent Semantic Analysis whereas Count Vectorizer was applied prior to Latent Dirichlet Allocation.

Other tokenization parameters included English stop words, accent stripping, minimum document frequency of 5, maximum document frequency of 7%, and two-character alphabetic characters only. While both unigrams and bigrams were tested, unigrams were found to generate the highest relative topic significances and lowest perplexity scores. The result of above parameters for unigrams preserved 9,086 tokens, about 31% of the original total without filtering.

It was also discovered that the vast majority of unigrams occurred in an extremely small number of documents, suggesting probable issues with noise from this long tail distribution. Roughly 90% of terms occurred in fewer than 20 documents, which represented less than 0.1% of the corpus.

## Reducing Dimensionality (LSA)
The primary method for selecting the optimal number of tokens, or dimensions, was by evaluating singular values and quality of term-topic groups from LSA with varying topic numbers. Topic numbers from 1 to 50 were tested, and the constructed elbow plot suggested 3 to 5 topics may be appropriate. For the proposed business case, it would be more useful to have a number moderately higher than this, however confirmation through LDA was examined.

## Topic Labeling (LDA)
Latent Dirichlet Allocation was favored due to the fewer number of topic assignments per document that results from the approach. The perplexity scores from 50-pass LDA with 1 to 15 topics were assessed, again with 3 topics as suggested optimum. However, it should be noted that perplexity scores do not necessarily align with human intuition for quality of topic assignments.

## Topic Evaluation
The quality of topic groupings and their constituent terms were evaluated across more than two dozen topic numbers for both LSA and LDA. While LSA indicated that 3 or fewer topics accounted for the majority of significance, the term-topic combinations for higher numbers also did not lead to intuitive groupings. The main issue was repetitive terms across many topics (overlap), even after stricter filtering in pre-processing. Higher topic numbers with LDA also suffered from this problem, although the ranked terms by probability per topic appeared more relevant to each topic. In essence, LDA seemed to generate more meaningful topics even though it could not resolve the issue of repetitive terms across higher numbers of topics (>5).

Ultimately only 3 topics were chosen and provided labels as follows, in order of ranked probability per topic:
- female / novel / drama / relationships
- war / history / memoir / politics
- food / self-help / business / health

On a per document level, 35% of books had a topic probability assignment of 90% or higher for any one of the above 3 topics. Conversely, 65% of books had "less confident" topic assignments with topic probabilities below 90% for one topic or less than or equal to 50% for two or more topics.

## Other Approaches
During the dimensionality reduction steps KMeans clustering was explored to determine if cluster assignments performed any better at grouping similar documents. Manual examination of cluster assignments for 40 to 60 clusters was done, which did not yield particularly informative groups. It is likely that a smaller number of clusters would perform better, but this was not pursued further.

Cosine similarities were calculated on LSA topic assignments for higher numbers of topics, and comparing the top 5 most similar books per document again indicated that fewer topics yielded more intuitive results, leading to the above decision to model only 3 topics.

## Challenges & Next Steps
Ultimately the topics and topic labels generated through this process made some sense, but I believe they have limited utility in practice. Since they group together multiple similar yet distinct themes, it would not be possible to build an accurate recommender system. However, as shown in presentation deck, looking at the number of best sellers per topic label over time does represent general trends and events, as well as show the relative popularity of the 3 topic groups.

Future work on this project will focus on further cleaning and pre-processing of the data to extract more topic signal without including excessive noise from the long tail unigrams. It is clear from the constituent terms that each of the 3 topics could in theory be broken down further, however the primary challenge is to prevent unnecessary term overlap across higher numbers of topics.

Finally, it is likely that other natural language algorithms and workflows would handle the characteristics of this corpus better, such as Non-Negative Matrix Factorization (NMF). Given the relatively short length of each document (book title + marketing description), NMF could both offer more interpretability (only positive coefficients for each constituent term) and more exclusive assignment of terms across topics.
