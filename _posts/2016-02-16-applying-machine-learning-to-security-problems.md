---
layout: post
title: Applying Machine Learning to Security Problems
comments: true
---

Anomaly detection and remediation is a hard and expensive process ridden with
false alarms and rabbit holes. The security community has been increasingly
interested in the potential for data-driven tools to filter out noise and automatically
detect malicious activity in large networks. However, while capable
of overcoming the limitations of static, rule-based techniques, machine learning
is not a silver bullet solution to detecting and responding to attacks. In this
post, we will detail two obstacles in applying machine learning to security, with
attention to how to solve them.

At its core, the (supervised) learning process is two-fold: training a model on
known data and testing it on unknown samples. In particular, adaptable models
require a continuous flow of labeled data to train with. Unfortunately, the
creation of such labeled data is the most expensive and time-consuming part
of the data science process. Data is usually messy, incomplete, and inconsistent.
While there are many tools to experiment with different algorithms and
their parameters, there are few tools to help one develop clean, comprehensive
datasets. Often times this means asking practitioners with deep domain expertise
to help label existing data elements. You can also try to purchase “good” data,
but this can be hard to come by in the security context (and may go stale very quickly).
You can also try to use a combination of unsupervised and supervised learning called—unsurprisingly—semi-supervised learning

On top of that, bias in training data can hamper the effectiveness of a model
to discern between output classes. In the security context, data bias can be
interpreted in two ways.

First, attack methodologies are becoming more dynamic than ever before. If a
predictive model is trained on known patterns and vulnerabilities, it may not
necessarily detect an unprecedented attack that does not conform to to those
1trends. If a predictive model is trained on known patterns and vulnerabilities
(i.e. using features from malware that is file-system resident),
it may not necessarily detect an unprecedented attack that does not conform
 to those trends (i.e. misses features from malware that is only memory resident).

Bias can sneak up on you, as well. You may think you can use the Alexa listings to,
say, obtain a list of benign domains, but that assumption may turn out to be a bad
idea since there is no guarantee that those sites are clean. Getting good ground
truth in security is hard.


Second, data bias also comes in the form of *class representation*. To understand
class representation bias, one can look to a core foundation of statistics: Bayes
theorem.


Bayes theorem describes the probability of event A given event B:

$$ P(A | B) = \frac { P (A) P(B | A)}{P(B)} $$

Expanding the probability P (B) for the set of two mutually exclusive outcomes,
we arrive at the following equation:

$$ P (B) = (A 1 )P (B|A 1 ) + (\neg A 2 )P (B|\neg A 2 ) $$

Combining the above equations, we arrive at the following alternative statement
of Bayes’ theorem:

$$ P (A|B) = \frac{P (A)P (B|A)} {P (A 1 )P (B|A 1 ) + P (\neg A 2 )P (B|\neg A 2 )} $$

Let’s apply this theorem to a concrete security problem to show the emergent
issues of training predictive models on biased data.

Suppose company $$X$$ has $$1000$$ employees, and a security vendor has deployed an
intrusion detection system (IDS) alerting the company $$X$$ when it detects a malicious
URL sent to an employee’s inbox. Suppose there are $$10$$ malicious URLs
sent to employees of company $$X$$ per day. Finally, suppose the IDS analyzes
$$10000$$ incoming URLs to company $$X$$ per day.

Let $$I$$denote an incident (an incoming malicious URL) and $$\neg I$$ denote a non-
incident (an incoming benign URL). Similarly, let $$A$$ denote an alarm (the
IDS classifies incoming URL as malicious) and $$\neg A$$ denote a non-alarm (the
IDS classifies URL as benign). That means $$P (A|I) = P (\text{hit})$$ and $$P (A| \neg I) =
P (\text{false alarm})$$.


What’s the probability that an alarm is associated with a real incident? In other
words, how much can we trust the IDS under these conditions?

Using Bayes’ Theorem from above, we know:

$$ P (I|A) = \frac{P (I)P (A|I)}{P (I)P (A|I) + P (\neg I)P (A|\neg I)}$$

Put another way,

$$ P (\text{IDS is accurate}) = \frac{P (\text{incident})P (\text{hit})}{P (\text{incident})P (\text{hit}) + P (\text{non-incident})P (\text{false alarm})}$$

Now let’s calculate $$P(\text{incident})$$ and $$P(\text{non-incident})$$, given the parameters of
the IDS problem we defined above:


$$ P(\text{incident}) =\frac{\text{10 incidents per day}}{\text{10000 audits per day}} $$

$$ P (\text{non-incident}) = 1 − P (\text{incident}) = 0.999$$

These probabilities emphasize the bias present in the distribution of analyzed
URLs. The IDS has little sense of what incidents entail, as it is trained on very
few examples of it. Plugging the probabilities into the equation above, we find
that:

Thus, to have reasonable confidence in an IDS under these biased conditions,
we must have not only unrealistically high hit rate, but also unrealistically low
false positive rate. For example, for an IDS to be $$80$$ percent accurate, even with
a best case scenario of a 100 percent hit rate, the IDS’ false alarm rate must be
$$4 \times 10^{−4}$$ . In other words, only $$4$$ out of $$10000$$ alarms can be false
positives to achieve this accuracy.

In the real world, detection hit rates are much lower and false alarm rates are
much higher. Thus, class representation bias in the security context can make
machine learning algorithms inaccurate and untrustworthy. When models are
trained on only a few examples of one class but many examples of another, the
bar for reasonable accuracy is extremely high, and in some cases unachievable.
Predictive algorithms run the risk of being ”the boy who cried wolf” – annoying
and prone to desensitizing security professionals to incident alerts. That last thing
you want to do is create a fancy new system that only exacerbates the problem that
was identified at the core of the Target/Home Depot breaches.

Data scientists can avoid these obstacles with a few measures:

1) Data scientists should pay particular attention to training models with large
and balanced data that are representative of all output classes. Take balanced
subsamples of your data if necessary, and understand your data[^1].

2) Data scientists should focus on getting a plethora of labeled data. Amazon’s
Mechanical Turk is a useful tool for this. Look at open sourced data, and encourage
data gathering expeditions.

3) Encourage security expertise on the team. Domain expertise is pretty important
to the performance of machine learning algorithms applied in the security
space. To keep up with the changing threat landscape, one must have security
experience.

4) Incorporate unsupervised methods into the solution of the data science problem.
Focus on organization, presentation, visualization, filtering of data - not
just prediction.

5) Weigh the trade-off between accuracy vs. coverage.

Machine learning has the potential to change how we detect and respond to
malicious activity in our networks. Machine learning can weed out signal from
noise to help incident responders focus on what’s important. It can help administrators
discover patterns in network activity never seen before. However, when applying these
algorithms to security we must be aware of caveats of the approach, so we may overcome
them

[^1]:[Classification on Biased Data](http://divac.ist.temple.edu/~vucetic/documents/vucetic01ecml.pdf)