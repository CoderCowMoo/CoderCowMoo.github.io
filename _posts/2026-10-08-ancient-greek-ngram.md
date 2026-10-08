---
layout: post
title: "Building an Ngram for Ancient Greek name generation"
date: 2026-10-08
categories: [Machine Learning, Projects]
tags: [python, experiments]
excerpt_separator: "<!--more-->"
---

A while ago I had an idea about a very naive approach to this problem, then I researched Markov chains
<!--more-->

## The Initial Idea

What if for a big corpus of Ancient Greek names, we find out the probability of each letter occuring at each index? So for index 0 we'd have a probability distribution, and so on for the rest. This is obviously very simple. It led to research about Markov chains, because I remembered that they were vaguely similar (at least in the use of a prob distribution).

## Markov Chain vs nGram?

Practically, an nGram is a subset, or a specific application of Markov chains – which are mainly of the mathematical domain – onto a sequence of data, like a string of text. I chose to build ngrams on a character tokenization basis, with each word being the Ancient Greek names. Before we get into that though, we must start with the data.

## Where did these names come from?

I found two data sources:
1. [BehindTheName](https://www.behindthename.com/names/usage/ancient-greek/)
2. [Kate Monk's Onomastikon](https://tekeli.li/onomastikon/Ancient-World/Greece/)

These two sources provided more information than just the name. BehindTheName contained information like gender, Greek script, origin, and description. Kate Monk kindly split up the names on gender, and provided information about the roots of some of the names.

For this project however, just the names were used. For preprocessing, each name was `.lower()`'d and edited while being added to the final list to have a start and end marker. E.g. `Acacius` -> `^acacius$`. This was a GPT-5.6 Sol (Medium) suggestion, a good one that I'll keep in mind for future projects.

## Can we start generating now?

The first idea that I thought of is actually a real thing, called a unigram. I never ended up implementing it, instead jumping straight into the bigram and trigram models ([`ancient_greek_gen.py` file](https://github.com/CoderCowMoo/ancient-greek-ngram/blob/master/ancient_greek_gen.py)). These were hardcoded and produced OK performance at best.

| Bigram | Trigram |
|  :---: |   :---:  |
| lenausit | mosiust |
| eonteusi | losmosiod | 
| leoneon | theros | 
| cylysu | ianosi | 
| boxacara | laessana |

Actually, these were generated without the `^$` markers mentioned before. The length and the first letter were decided by probability distribution, and we'd simply sample from the bigram model until the length has been reached.

This approach isn't great. It also doesn't track whether a name has already appeared in the dataset before.

## A better approach

It was now time to generalise the ngram model. We can do away with the need for a length probability distribution or first letter distribution, thanks to our addition of the `^$` markers.

The generalised_ngram [code](https://github.com/CoderCowMoo/ancient-greek-ngram/blob/master/generalised_ngram.py) uses a default order of 6, meaning that the model considers at most the past 5 letters in deciding the next letter to be chosen. Its very likely when randomly building a word, that some sequences of 5 letters would have not been seen before.

For this reason, the program also falls back to lower and lower order ngrams, until in the rare case that the sequence doesn't exist at all in the dataset, we randomly choose a single letter in the alphabet ([line 98](https://github.com/CoderCowMoo/ancient-greek-ngram/blob/24a68ce50b1697ce0406dfc77f4186f9efcaa9f2/generalised_ngram.py#L98)).

An unexpected result when testing was that the performance was amazing (subjectively). Amazingly _suspicious_... When I checked the actual data, what do I find but that obviously, the ngram model has been simply regurgiating the dataset. This is real overfitting of a very simple model.

Thus, in order to generate 1000 generated Ancient Greek names, using an order 6 nGram model, approximately 49,000 names were generated and thrown away because they were already in the dataset. However, those that stayed are clearly super good names!

| Bigram | Trigram | 6gram |
|  :---: |   :---:  | :---: |
| lenausit | mosiust | isodemostrate |
| eonteusi | losmosiod | phalanthe |
| leoneon | theros | nikomache |
| cylysu | ianosi | thymos |
| boxacara | laessana | alemenes |

## Future Direction

Amazing results I'd say, but thats just my opinion. I could of course go around asking other people for their opinions, but that would still be hard to quantify exactly. That's exactly the next problem I need to solve for this generation task.

How can we benchmark? Well we'd need to first split up the data into training and testing data, then we need to compare the generated text with the test data.

We could go sequence by sequence in the test data and see how likely the model is to create the test sequence from sequentially more data.

I dont know everything about how exactly, but some buzzwords to investigate are cross entropy, log likelihood, perplexity.

## Conclusion

This was fun to create and to see the results. I need to get into the mindset of understanding what my model is actually predicting, and the mindset of preparing for benchmarking, as thats how you even know if the work you've done has had the effect you've desired.

The code for this experiment can be found at my [GitHub repo](https://github.com/CoderCowMoo/ancient-greek-ngram).