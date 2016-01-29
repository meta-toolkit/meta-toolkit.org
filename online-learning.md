---
layout: page
title: Online Learning
category: tut
order: 6
---

In many cases, the datasets we deal with in large machine learning
problems are too large to fit into the working-set memory of a modest
computer. Fortunately, MeTA's use of indexes for storing its data make it
very capable of handling these cases when coupled with a classifier that
supports online learning (such as `sgd` coupled with any of the
appropriate loss functions, or a `one_vs_all` or `one_vs_one` ensemble of
these classifiers). In this tutorial, we'll explore performing online
learning of an `sgd`-trained support vector machine (SVM) on a dataset from
the LIBSVM dataset website,
[rcv1.binary](http://www.csie.ntu.edu.tw/~cjlin/libsvmtools/datasets/binary.html#rcv1.binary).
(This dataset is not actually large enough to require an online learning
algorithm, but it is large enough to demonstrate the value in this
approach, and one can easily see how the method can be extended to
datasets that truly do require an online-learning approach.)

Download the training and testing sets and place them in your data
directory under a folder called `rcv1`. Then, construct an `rcv1.dat` from
the following command (or equivalent):

{% highlight bash %}
bzcat rcv1_test.binary.bz2 rcv1_train.binary.bz2 > rcv1.dat
{% endhighlight %}

(Note: we will be using the given test set as the training set and the
given training set as the test set: this is mostly just so that the
training set is sizeable enough to appreciate the memory usage of the
toolkit during training: reversing the splits gives us approximately 1.2GB
of training data to process).

Next, create the corpus configuration file `libsvm.toml` in the same folder
as `rcv1.dat` with the following content:

{% highlight toml %}
type = "libsvm-corpus"
num-docs = 687641
{% endhighlight %}

The approach we will take here is to run the `sgd` training algorithm
several times on "mini-batches" of the data. The dataset has a total of
677399 training examples, and if we loaded these all into memory it would
take nearly a gigabyte of working memory. Instead, we will load the data
into memory in chunks of <= 50000 documents and train on each chunk
individually.

Below is the relevant portion of our `config.toml` for this example:

{% highlight toml %}
corpus = "libsvm.toml"
dataset = "rcv1"
forward-index = "rcv1-fwd"
inverted-index = "rcv1-inv"

# the size of the mini-batches: this may need to be set empirically based
# on the amount of memory available on the target system
batch-size = 50000
# the document-id where the test set begins
test-start = 677399

[[analyzers]]
method = "libsvm"

[classifier]
method = "one-vs-all"
    [classifier.base]
    method = "sgd"
    loss = "hinge"
{% endhighlight %}

Now, we can run the provided application with `./online-classify
config.toml`, see results that look something like the following:

<div>
<code>
<pre>
$ ./online-classify config.toml
Training batch 1/14
 > Loading instances into memory: [=========================] 100% ETA 00:00:00
 Training batch 2/14
 > Loading instances into memory: [=========================] 100% ETA 00:00:00
 Training batch 3/14
 > Loading instances into memory: [=========================] 100% ETA 00:00:00
 Training batch 4/14
 > Loading instances into memory: [=========================] 100% ETA 00:00:00
 Training batch 5/14
 > Loading instances into memory: [=========================] 100% ETA 00:00:00
 Training batch 6/14
 > Loading instances into memory: [=========================] 100% ETA 00:00:00
 Training batch 7/14
 > Loading instances into memory: [=========================] 100% ETA 00:00:00
 Training batch 8/14
 > Loading instances into memory: [=========================] 100% ETA 00:00:00
 Training batch 9/14
 > Loading instances into memory: [=========================] 100% ETA 00:00:00
 Training batch 10/14
 > Loading instances into memory: [=========================] 100% ETA 00:00:00
 Training batch 11/14
 > Loading instances into memory: [=========================] 100% ETA 00:00:00
 Training batch 12/14
 > Loading instances into memory: [=========================] 100% ETA 00:00:00
 Training batch 13/14
 > Loading instances into memory: [=========================] 100% ETA 00:00:00
 Training batch 14/14
 > Loading instances into memory: [=========================] 100% ETA 00:00:00

 > Loading instances into memory: [=========================] 100% ETA 00:00:00

           -1       1
         ------------------
      -1 | <strong>0.971</strong>    0.0289
       1 | 0.0207   <strong>0.979</strong>

------------------------------------------------
<strong>Class       F1 Score    Precision   Recall</strong>
------------------------------------------------
-1          0.974       0.978       0.971
1           0.976       0.973       0.979
------------------------------------------------
<strong>Total       0.975       0.975       0.975</strong>
------------------------------------------------
20242 predictions attempted, overall accuracy: 0.975
Took 4.519s
</pre>
</code>
</div>

This general process should be able to be extended to work with any dataset
that cannot be fit into memory, provided an appropriate `batch-size` is set
in the configuration file.
