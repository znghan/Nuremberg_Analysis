![alt text](images/title.png)

## Introduction
During my time at Insight Data Science, I consulted for [tolstoy.ai](https://tolstoy.ai/). The company specializes in applying Natural Language Processing to analyzing large corpus of text data. The client of the company who requested this project is the law school of a prominent university. As part of its long-term effort to digitize and analyze the huge corpus of legal documents in its archive, it has enlisted the help of the company to process some of these documents and more efficiently extract structured content from the text.  

## Problem Statement
The school has, in its archive, the entire transcript of the Nuremberg Trials in the form of text. They want to extract all evidence references from within the transcript. These are instances in which the lawyer is citing a piece of document to prove her side of the argument during a court proceeding. The document can be submitted into evidence by either the prosecution or the defense. Below are some examples of evidence references.  

<img src="images/ProblemStatement1.png" width="800">

After extracting these references, the school wants to classify them as either coming from the prosecution or the defense team. Once we know the reference number and whether it is coming from the prosecution or defense, we can uniquely map the in-transcript reference to the actual evidence document the school has in its archive. So when a legal researcher is reading through the transcript, she can immediately link to and look up the original document, making the research process very efficient and convenient. 
   
<img src="images/ProblemStatement2.png" width="560">


## Create Training Set
I was working with just the raw text of the transcript with no labels. As a first step, I wrote a regular expression script in Python that managed to extract 40136 references from the text. Then based on discussion with the school who has the domain expertise, I came up with simple heuristics that can determines very quickly whether the reference is coming from prosecution or defense team. For example, if the reference says "Exhibit Frank No. 5", where Frank is the name of one of the defendants, we can be pretty sure that it is a defense evidence. I applied the heuristic on our forty thousand references, and was able to label about 58% of them. The plan is to develop a model based on these heuristically labeled data so that we can further classify the 42% which is still unlabeled. 

<img src="images/TrainSet.png" width="400">

The 58% labeled set is further divided into a training and a validation set according to a 80-20 split. The model will be trained on the training set, while the validation set will be reserved for hyperparameter tuning and model selection.  


## Feature Engineering 

So how do we tell whether a reference is prosecution or defense? It is really all about the context in which the reference occurred. I captured the idea of context in my model in two ways. The first feature that I used is the "Speaker". It is a categorical variable that encodes who was actually talking and referring to the particular evidence. The chart below showed the top 5 speakers, by number of times he was speaking, across the prosecution and the defense evidence. For the prosecution evidences, all 5 speakers were on the prosecution team while for the defense evidences, all 5 speakers were German defense counsels. This shows that knowing whether the reference is being mentioned by the prosecution or the defense team is very predictive of the type of reference it is. 

<img src="images/Features1.png" width="640">

The second feature is the word choice. I extracted the paragraph around the reference and tried to understand if different words tend to be used. The word cloud below shows the most common words for prosecution evidence. The three most common words are "prosecution", "english", and "mr".   

<img src="images/Features2.png" width="560">

In comparison, the most common words for defense evidence, shown below, are "affidavit", "number", and "dr". Apparently the German defense counsels were very precise about numbers and quantities (number of prisoners in concentration camp, number of troops deployed in a certain operation etc). In addition, Germans tend to address anyone with a doctorate as "Dr", including all members of the defense counsel who has a doctorate in Law.  

<img src="images/Features3.png" width="560">


## Classification

I compared several models including Logistic Regression, Random Forest, and Gradient Boosting. The Random Forest performed the best in the validation set. 

## Performance Evaluation

There are two metrics that I used to evaluate my work, corresponding to the two tasks I need to perform: extraction and classify. The first metric is the extraction rate. It captures the proportion of references within the transcript that I was able to extract. To evaluate that, I randomly selected 50 pages, read them and manually tagged out the references. I found a total of 199 references in the 50 pages, and my script managed to extract 187 of them. This is an extraction rate of 94%. In comparison, the client had an old script that was only able to extract 65% of the references.

The second metric, or set of metrics, is the classification performance on the references that I did extract. I took a random sample of 200 references from among the 42% that is unlabeled. I manually read over their contexts, labeled them, and compared to the model prediction. My model achieved an F1-score of 96%, with comparable precision and recall. The confusion matrix is shown below.

<img src="images/Evaluation.png" width="480">
 
