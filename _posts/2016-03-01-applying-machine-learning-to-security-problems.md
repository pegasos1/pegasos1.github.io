---
layout: post
title: Applying Machine Learning to Security Problems
comments: true
---

Anomaly detection is a hard process ridden with false alarms. The security community has been increasingly interested in the potential for data-driven tools to filter out noise and automatically detect malicious activity in large networks. However, while capable of overcoming the limitations of static, rule-based techniques, machine learning is not a silver bullet solution to detecting and responding to attacks.

Adaptable models require a continuous flow of labeled data to train with. Unfortunately, the creation of such labeled data is the most expensive and time-consuming part of the data science process. Data is usually messy, incomplete, and inconsistent. While there are many tools to experiment with different algorithms and their parameters, there are few tools to help one develop clean, comprehensive datasets. Often times this means asking practitioners with deep domain expertise to help label existing datasets. But ground truth be hard to come by in the security context, and may go stale very quickly.

On top of that, bias in training data can hamper the effectiveness of a model to discern between output classes. In the security context, data bias can be interpreted in two ways.

First, attack methodologies are becoming more dynamic than ever before. If a predictive model is trained on known patterns and vulnerabilities (e.g. using features from malware that is file-system resident), it may not necessarily detect an unprecedented attack that does not conform to those trends (e.g. misses features from malware that is only memory resident).

Second, data bias also comes in the form of *class representation*[^1]. To understand class representation bias, one can look to a core foundation of statistics: Bayes theorem.


Bayes theorem describes the probability of event A given event B:

$$ P(A | B) = \frac { P (A) P(B | A)}{P(B)} $$

Expanding the probability P (B) for the set of two mutually exclusive outcomes, we arrive at the following equation:

$$ P (B) = (A_1 )P (B|A_1 ) + (\neg A_2 )P (B|\neg A_2 ) $$

Combining the above equations, we arrive at the following alternative statement of Bayes’ theorem:

$$ P (A|B) = \frac{P (A)P (B|A)} {P (A_1 )P (B|A_1 ) + P (\neg A_2 )P (B|\neg A_2 )} $$

Let’s apply this theorem to a concrete security problem to show the emergent issues of training predictive models on biased data.

Suppose company $$X$$ has $$1000$$ employees, and a security vendor has deployed an intrusion detection system (IDS) alerting the company $$X$$ when it detects a malicious URL sent to an employee’s inbox. Suppose there are $$10$$ malicious URLs sent to employees of company $$X$$ per day. Finally, suppose the IDS analyzes $$10000$$ incoming URLs to company $$X$$ per day.

Let $$I$$ denote an incident (an incoming malicious URL) and $$\neg I$$ denote a non-incident (an incoming benign URL).
Similarly, let $$A$$ denote an alarm (the IDS classifies incoming URL as malicious) and $$\neg A$$ denote a non-alarm (the
IDS classifies URL as benign). That means $$P (A|I) = P (\text{hit})$$ and $$P (A| \neg I) =
P (\text{false alarm})$$.


What’s the probability that an alarm is associated with a real incident? In other words, *how much can we trust the IDS under these conditions?*

Using Bayes’ Theorem from above, we know:

$$ P (I|A) = \frac{P (I)P (A|I)}{P (I)P (A|I) + P (\neg I)P (A|\neg I)}$$

Put another way,

$$ P (\text{IDS is accurate}) = \frac{P (\text{incident})P (\text{hit})}{P (\text{incident})P (\text{hit}) + P (\text{non-incident})P (\text{false alarm})}$$

Now let’s calculate $$P(\text{incident})$$ and $$P(\text{non-incident})$$, given the parameters of the IDS problem we defined above:


$$ P(\text{incident}) =\frac{\text{10 incidents per day}}{\text{10000 audits per day}} = 0.001$$

$$ P (\text{non-incident}) = 1 − P (\text{incident}) = 0.999$$

These probabilities emphasize the bias present in the distribution of analyzed URLs. The IDS has little sense of what incidents entail, as it is trained on very few examples of it. Plugging the probabilities into the equation above, we find that:

$$ P (\text{IDS is accurate}) = \frac{0.001 * P (\text{hit})}{0.001 * P (\text{hit}) + 0.999 * P (\text{false alarm})}$$

Thus, to have reasonable confidence in an IDS under these biased conditions, we must have not only unrealistically high hit rate, but also unrealistically low false positive rate. For example, for an IDS to be $$80$$ percent accurate, even with a best case scenario of a 100 percent hit rate, the IDS’ false alarm rate must be $$4 \times 10^{−4}$$ . In other words, only $$4$$ out of $$10000$$ alarms can be false
positives to achieve this accuracy.

In the real world, detection hit rates are much lower and false alarm rates are much higher. Thus, class representation bias in the security context can make machine learning algorithms inaccurate and untrustworthy. When models are trained on only a few examples of one class but many examples of another, the bar for reasonable accuracy is extremely high, and in some cases unachievable. Predictive algorithms run the risk of being ”the boy who cried wolf” – annoying and prone to desensitizing security professionals to incident alerts.

Security data scientists can avoid these obstacles with a few measures:

1) **Apply structure to data with supervised and semi-supervised learning.**

2) **Undersample the majority class and/or oversample the minority class.** See scikit learn's stratified data splitting functions [^2] and this repo [^3]

3) **Generate synthetic data from minority class via algorithms like SMOTE.** See this repo[^3] again.

5) **Build models that penalize classification to the majority class.**

6) **Focus on organization, presentation, visualization, filtering of data - not just prediction.**

7) **Encourage data gathering expeditions.**

8) **Encourage security expertise on the team.** Security expertise can help you think of viable solutions to problems when data is insufficient.

9) **Weigh the trade-off between accuracy vs. coverage.** The effects of false positives are particularly detrimental in the security space, meaning that for some applications it may be more useful to sacrifice the volume of accurate classifications for higher confidence.

Machine learning has the potential to change how we detect and respond to malicious activity in our networks by weeding out signal from noise. It can help security professionals discover patterns in network activity never seen before. However, when applying these algorithms to security we have to be aware of caveats of the approach so we can address them.

[^1]:[Classification on Biased Data](http://divac.ist.temple.edu/~vucetic/documents/vucetic01ecml.pdf)
[^2]:[Sklearn's Stratified K-Fold](http://scikit-learn.org/stable/modules/generated/sklearn.cross_validation.StratifiedKFold.html#sklearn.cross_validation.StratifiedKFold)
[^3]:[Handling Imbalanced Data](https://github.com/fmfn/UnbalancedDataset)
