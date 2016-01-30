---
title: Classification
layout: page
category: tut
order: 5
---

Beyond its search and information retrieval capabilities, MeTA also
provides functionality for performing document classification on your
corpora. We will start this tutorial by discussing how you can perform
document classification experiments on your data **without writing a single
line of code**, and then discuss more complicated examples later on.

The first step in setting up your classification task is, of course,
selecting your corpus. MeTA's built-in `classify` program works with all of
MeTA's [corpus formats][corpus-formats], giving you a lot of freedom in how
you decide to store your unprocessed data.

[corpus-formats]: overview-tutorial.html#corpus-input-formats

MeTA uses a compressed format for its internal `forward_index`
representation, but you can at any point in time dump the contents of your
`forward_index` to a libsvm-formatted data file by using the provided
`forward-to-libsvm` tool if you would like to feed the data to another tool
not yet integrated with MeTA itself.

## Creating a Forward Index From Scratch

To create a `forward_index` directly from your corpus input, your
configuration file would look something like this:

{% highlight toml %}
corpus = "line.toml"
dataset = "20newsgroups"
forward-index = "20news-fwd"
inverted-index = "20news-inv"

[[analyzers]]
method = "ngram-word"
ngram = 1
filter = "default-chain"
{% endhighlight %}

Here, we've specified the locations for an inverted and a forward index,
and based on our analyzers configuration we can see that we're using
unigram words as our features. If you want to use more complex feature
representations, please [refer to the analyzers, tokenizers, and filters
tutorial][ana-tut] for more information.

[ana-tut]: analyzers-filters.tutorial.html

## Creating a Forward Index from LIBSVM Data

In many cases, you may already have pre-processed corpora to perform
classification tasks on, and MeTA gracefully handles this. The most common
input for these classification tasks is the LIBSVM file format, and MeTA
supports this format directly as an input corpus.

To create a `forward_index` from data that is already in LIBSVM format,
your configuration file would look something like this:

{% highlight toml %}
corpus = "libsvm.toml"
dataset = "rcv1"
forward-index = "rcv1-fwd"
inverted-index = "rcv1-inv"

[[analyzers]]
method = "libsvm"
{% endhighlight %}

The `forward_index` will recognize that this is a LIBSVM formatted corpus
and will simply read the existing features from the corpus and convert them
into MeTA's internal compressed format. An corresponding `inverted_index`
cannot be created through this method, and so you will not be able to use
dual-index classifiers such as `knn` that require an `inverted_index`, but
most of the regular classifiers that don't require search features will
work just fine (e.g., SGD and Naive Bayes).

## Selecting a Classifier

To actually run the `classify` executable, you will need to decide on a
classifier to use. You may see a list of these [in the API documentation
for the `classifier`
class](doxygen/classmeta_1_1classify_1_1classifier.html) (they are listed
as subclasses). The public static id member of each class is the
identifier you would use in the configuration file.

A recommended default configuration is given below, which learns a linear
SVM via stochastic gradient descent and uses a one-vs-all reduction:

{% highlight toml %}
[classifier]
method = "one-vs-all"
    [classifier.base]
    method = "sgd"
    loss = "hinge"
{% endhighlight %}

Here is an example configuration that uses Naive Bayes:

{% highlight toml %}
[classifier]
method = "naive-bayes"
{% endhighlight %}

Here is an example that uses *k*-nearest neighbor with *k* = 10 and Okapi BM25
as the ranking function:

{% highlight toml %}
[classifier]
method = "knn"
k = 10
    [classifier.ranker]
    method = "bm25"
{% endhighlight %}

Running `./classify config.toml` from your build directory will now create
a `forward_index` (if necessary) and run 5-fold cross validation on your
data using the prescribed classifier. Here is some sample output:

<div>
<code>
<pre>
            chinese   english   japanese
          ------------------------------
  chinese | <strong>0.802</strong>     0.011     0.187
  english | 0.0069    <strong>0.807</strong>     0.186
 japanese | 0.0052    0.0039    <strong>0.991</strong>

------------------------------------------------
<strong>Class</strong>       <strong>F1 Score</strong>    <strong>Precision</strong>   <strong>Recall</strong>
------------------------------------------------
chinese     0.864       0.802       0.936
english     0.88        0.807       0.967
japanese    0.968       0.991       0.945
------------------------------------------------
<strong>Total</strong>       <strong>0.904</strong>       <strong>0.867</strong>       <strong>0.949</strong>
------------------------------------------------
1005 predictions attempted, overall accuracy: 0.947
</pre>
</code>
</div>

## Online Learning
If your dataset cannot be loaded into memory in its entirety, you should
look at [the online learning tutorial](online-learning.html) for more
information about how to handle this case.

## Manual Classification

If you want to customize the classification process (such as providing
your own test/training split, or changing the number of cross-validation
folds), you should interact with the classifiers directly by writing some
code. Refer to `classify.cpp` and [the API documentation for
`classifier`](doxygen/classmeta_1_1classify_1_1classifier.html).

The next few sections will guide you through the high level structure of
the classify APIs.

### Datasets and Dataset Views

A [`dataset` object][dataset] represents a collection of `instance`s that
have been loaded into memory. This may be the entirety of your corpus or
just some small segment of it. Typically, you will instantiate `dataset`
objects by passing in a `forward_index` object to retrieve documents from,
but you can also create them manually from your own data as well.

[dataset]: doxygen/classmeta_1_1learn_1_1dataset.html

The most common concrete `dataset` type you'll use for classification is
[the `multiclass_dataset`][multiclass-dataset]. This is a labeled dataset
with categorical labels (each `instance` has an associated `class_label`,
which is a string). If you wish to load the entirety of a `forward_index`
into memory, you can use the single argument constructor like so:

{% highlight cpp %}
using namespace meta;

// parse the configuration file
auto config = cpptoml::parse_file(argv[1]);

// create or load a forward index
auto f_idx = index::make_index<index::forward_index>(*config);

// load your index into a collection of instances for training/testing
classify::multiclass_dataset dataset{f_idx};
{% endhighlight %}

[multiclass-dataset]: doxygen/classmeta_1_1classify_1_1multiclass__dataset.html

Now that you have a dataset, you can now create [`dataset_view`
objects][dataset-view] to represent read-only views of parts (or all of) a
specific `dataset`. These view objects can then passed down to the
classifiers for either training or testing. The most commonly used
`dataset_view` object is the `multiclass_dataset`'s [corresponding
`multiclass_dataset_view`][multiclass-dataset-view], which is typically
created from a `multiclass_dataset` and a pair of iterators into that
dataset indicating the extent that view represents.

For example, if I wanted to have a training set consisting of the first
half of my data, and a testing set consisting of the second half of my
data, I can construct a training view and a testing view as follows:

{% highlight cpp %}
// an mdv of the first half of the dataset, for training
classify::multiclass_dataset_view train{dataset, dataset.begin(),
                                        dataset.begin() + dataset.size() / 2};

// an mdv for the second half of the dataset, for testing
classify::multiclass_dataset_view test{dataset,
                                       dataset.begin() + dataset.size() / 2,
                                       dataset.end()};
{% endhighlight %}

Creating these views is important to allow for things like shuffling
without disturbing the underlying data. I can now shuffle both training and
test sets before I begin training and testing my classifier.

{% highlight cpp %}
train.shuffle();
test.shuffle();
{% endhighlight %}

[dataset-view]: doxygen/classmeta_1_1learn_1_1dataset__view.html
[multiclass-dataset-view]: doxygen/classmeta_1_1classify_1_1multiclass__dataset__view.html

### Training and Testing Classifiers

Now, let's train a classifier and get some statistics about its performance
on the test set. Classifiers are typically created with a TOML configuration
group (either read from a file or created programmatically) and a
corresponding `dataset_view` that represents the training data to use.
**Construction of a classifier implies training it.**

To train a Naive Bayes classifier, I could do the following:

{% highlight cpp %}
auto cls_cfg = cpptoml::make_table();
cls_cfg->insert("method", "naive-bayes");
auto cls = classify::make_classifier(*cls_cfg, train);
{% endhighlight %}

Finally, to test the classifier I can do the following:

{% highlight cpp %}
auto confusion_mtrx = cls->test(test);
confusion_mtrx.print();       // prints the confusion matrix itself
confusion_mtrx.print_stats(); // prints statistics from the matrix
{% endhighlight %}

If I wanted to instead do 10-fold cross validation on the dataset I
initially loaded, I could do the following:

{% highlight cpp %}
auto confusion_mtrx = classify::cross_validate(*cls_cfg, dataset, 10);
{% endhighlight %}

Once your model is trained, you may wish to save it for later use. All of
our classifiers support being serialized to the disk. To do so, you can use
the `save()` method like so:

{% highlight cpp %}
// configure and train cls first, and then...
std::ofstream output{"my-model.dat", std::ios::binary};
cls->save(output);
{% endhighlight %}

To load a model from a file, simply use the `load_classifier()` method on a
binary input stream:

{% highlight cpp %}
// loading a model from a file
std::ifstream input{"my-model.dat", std::ios::binary};
auto loaded_cls = classify::load_classifier(input);
{% endhighlight %}

## Writing Your Own Classifiers

The first step for writing your own classifier is to determine what *kind*
of classifier you are writing. You should ask yourself the following
questions:

1. Does your classifier predict categorical labels or binary labels?
2. Does your classifier support online learning or only batch learning?

Based on the answers, you should pick one of the following base classes:

- `classifier` is the base class you should use for multiclass
  (categorical) classifiers that *do not* support online learning
- `online_classifier` is the base class you should use for multiclass
    classifiers that *do* support online learning
- `binary_classifier` is the base class you should use for binary
    classifiers that *do not* support online learning
- `online_binary_classifier` is the base class you should use for binary
    classifiers that *do* support online learning

For registration purposes (more on this later), your classifier should have
a public static `id` member of type `util::string_view` that is unique.

To facilitate saving and loading classifiers, your classifier should have a
`save(std::ostream&)` function that writes out your classifier's model
information in binary format to the stream parameter (we strongly recommend
using something like `io::packed::write` for this). **The very first line
of this should write out the classifier's id** like so:

{% highlight cpp %}
io::packed::write(out, id);
{% endhighlight %}

This allows the toolkit to be able to load your classifier from a file
directly, without having to manually specify the type at load time.

You should also have a constructor from a `std::istream&` to load your
classifier from a file. **The id will have already been read from the
stream**, so you should begin immediately reading the things you wrote
*after* the first line of `save()`.

### Registering Classifiers

Once your classifier is written, you should register it with the toolkit to
enable creating it from a configuration file and loading it from disk. To
do so, ensure that your classifier has a public static `id` member (of type
`util::string_view`), and then register it somewhere in `main()` like this:

{% highlight cpp %}
using namespace meta;

// if you have a multi-class classifier
classify::register_classifier<my_classifier>();

// if you have a multi-class classifier that requires an inverted_index
// (this is not common; examples include knn and nearest centroid)
classify::register_multi_index_classifier<my_classifier>();

// if you have a binary classifier
classify::register_binary_classifier<my_binary_classifier>();
{% endhighlight %}

If you need to read parameters from the configuration group given for your
classifier, you should specialize the `make_classifier()` function like so:

{% highlight cpp %}
// if you have a multi-class classifier
namespace meta
{
namespace classify
{
template <>
std::unique_ptr<classifier>
    make_classifier<my_classifier>(
        const cpptoml::table& config,
        multiclass_dataset_view training);
}
}

// if you have a multi-class classifier that requires an
// inverted_index
namespace meta
{
namespace classify
{
template <>
std::unique_ptr<classifier>
    make_multi_index_classifier<my_classifier>(
        const cpptoml::table& config,
        multiclass_dataset_view training,
        std::shared_ptr<index::inverted_index> inv_idx);
}
}

// if you have a binary classifier
namespace meta
{
namespace classify
{
template <>
std::unique_ptr<classifier>
    make_binary_classifier<my_binary_classifier>(
        const cpptoml::table& config,
        binary_dataset_view training);
}
}
{% endhighlight %}
