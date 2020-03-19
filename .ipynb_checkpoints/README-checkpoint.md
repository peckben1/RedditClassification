DSI 10, Project 3: Subreddit NLP
Ben Peck


Contents:
- Problem Statement
- Guide to Files
- Methods
- Conclusions


Problem Statement:

My objective is to accurately predict which of two subreddits a post originated in based on the post's textual content, even when the two subreddits are on similar topics. In order to achieve this, I intend to build and test multiple classification models, namely K Nearest Neighbors and Naive Bayes Multinomial, to select the best-performing model. Success will be evaluated based solely on predictive accuracy, as type I and type II errors are of equal consequence in this context. Being able to reliably distinguish between two groups interacting with the same topic from different perspectives is of clear consequence to advertisers, campaigners, and others seeking to identify target populations holding or receptive to certain views. 


Guide to Files:

- code
    - GetData.ipynb: rough draft code for pulling data from the PushShift API and saving to csv
    - GetDataPolished.ipynb: polished, streamlined version of GetData
    - ProcessData.ipynb: code for cleaning and modeling data
    - CommonWords.ipynb: a copy of the first half of ProcessData adapted to show the most common words by subreddit
- data
    - posts.csv: data pulled from PushShift API
- presentation
    - Project3Peck.pdf: presentation slide deck


Methods: 

I selected two subreddits for my analysis, r/latterdaysaints and r/exmormon. These subreddits are run by communities supportive of and critical of the LDS (aka Mormon) Church, respectively. Because they deal with many of the same topics, they will be difficult to separate based solely on the presence of specific technical words, but should be separable based on words typifying the unique sentiment and culture of each group. 
Using Reddit's PushShift API, I pulled the 10,000 most recent submissions from each sub. Because r/exmormon is a larger and more active sub, this resulted in a larger slice chronologically being taken from r/latterdaysaints, but as posting across time seems fairly stable in both subs I didn't think this would be detrimental to analysis. The only columns retained from the data were subreddit, title, selftext (the text content of the body of the post), createdutc (time of post), and score. All posts with blank, removed, or deleted selftext were dropped from the data, leaving approximately 5,000 posts from each subreddit - the drop did not unbalance the subreddit proportions. The title was then concatenated to the selftext (with a space in between) to create a single text feature called all_text. Some replacements were run on all_text to clean up formatting strings and refine the text for tokenization (eg. "&amp;" was changed to simply "&". ) The data was also run through a count vectorizer with a max_features of 100, summed, and sorted in descending order to find the most common words in the dataset (stop words excluded). This process was repeated with data filtered by subreddit to find the most common words for each individually. The number of max features in this cvec was temporarily raised to 1000 for EDA purposes, but this was not left in the final notebook as the output was excessively large. 
For modeling, two pipelines were created. The first pipeline grid searched over a count vectorizer and a K Nearest Neighbors estimator with parameters as follows: 
{'knn__n_neighbors': 3, 5, 7; 'cvec__ngram_range': (1,1); 'cvec__max_features': 1000, 2000, 5000; 'cvec__stop_words': 'english', None} 
The best parameters for this gridsearch were found to be: 
{'knn__n_neighbors': 3; 'cvec__ngram_range': (1,1); 'cvec__max_features': 5000; 'cvec__stop_words': 'english'} 
However, on test data this model's accuracy score was only 62.7% - hardly better than the baseline score of just over 50%. With clear room for improvement, a second pipeline was made. 
The second pipeline grid searched over a count vectorizer and a naive bayes multinomial estimator with parameters as follows:
{'nbm__alpha': 0.1, 0.5, 1, 5; 'cvec__ngram_range': (1,1), (1,2); 'cvec__max_features': 1000, 2000, 5000, None} 
Stop words were set to 'english' to prevent the fitting of a model with neither max features nor stop words. The best parameters for this gridsearch were found to be:
{'cvec__max_features': None; 'cvec__ngram_range': (1, 2); 'nbm__alpha': 0.5}
With an accuracy score of 99.7% on training data and 85.4% on testing data, this model was far from perfect but performed far better than the previous model, accurately enough to be considered a relatively reliable predictor. 


Conclusions:

The naive bayes multinomial was able to predict the subreddit a post originated from with reasonably high accuracy, demonstrating that this process is possible and that NBM is likely a good model for this particular application. The subreddits dealt with many of the same topics and shared common words which occur at relatively low frequency in general usage, including "church," "lds," "gt" (for "Gospel Topics," a series of essay), "mormon," "ward," and many others. However, each sub also had its own set of more distinctive words and phrases reflecting its stance and culture. More generally religious terminology relating to day-to-day practice appeared in greater proportion on r/latterdaysaints, while r/exmormon saw far higher use of words and phrases like "left," "tscc" (for "the so-called Church"), "coffee," "cult," and curse words (which are generally avoided by devout Mormons). The next steps would involve examining the strength of different features in the naive bayes multinomial model (the values of which were retrieved but I was unable to interpret) to more accurately define the linguistic footprint of the two communities and to further refine the model, especially by examining sentiment. As the two subs deal with the same topic but largely opposing sentiments, analysis of the co-occurrence of certain words and the use of sentiment words could prove a powerful tool for differentiating the two groups, especially when examined in conjunction with score. The community-awarded score on Reddit is a measure of sentiment in its own way, so the incidence of words and phrases with high scores could be enlightening (e.g. "batpized today" anecdotally seems to guarantee a high score on r/latterdaysaints and a lower score on r/exmormon). The model as it stands is reasonably successful, but more work could certainly be done. 