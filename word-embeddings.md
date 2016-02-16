---
layout: page
title: Word Embeddings
category: tut
order: 12
---

[Word embeddings][word-embeddings] are a way of representing the individual
words used in natural languages as fixed-length numeric vectors in some
vector space.  Most useful models for word embeddings find vectors for
words where meaning can be captured via (linear) vector composition. For
example, one can answer word analogy questions like the following:

- *woman* is to *sister* as *man* is to what? (*brother*)
- *summer* is to *rain* as *winter* is to what? (*snow*)
- *man* is to *king* as *woman* is to what? (*queen*)
- *fell* is to *fallen* as *ate* is to what? (*eaten*)

We can answer these questions by finding the word vector that is most
similar (via some metric like [cosine similarity][cosine]) to the result of
some vector math operation. For answering the first question, one might
form a query like

$$\arg\max_{\mathbf{v_i}} \frac{(sister - woman + man)^\intercal
\mathbf{v_i}}{||(sister - woman + man)|| \: ||\mathbf{v_i}||}$$

where $$\mathbf{v_i}$$ represents a word embedding vector for a particular
word $$i$$ in our vocabulary.

There are many different models for word embeddings. MeTA implements
the learning algorithm from [GloVe][glove] for learning its word
embeddings. This tutorial will walk you through how to use the tools in
MeTA for learning and interacting with word embeddings on your own data.

## Learning Embeddings

MeTA's GloVe implementation is broken into three steps:

1. Extract a vocabulary from the data for which we would like to construct
   word embeddings
2. Use that vocabulary to extract the co-occurrence matrix from our data
3. Learn word embeddings for each word in our vocabulary using the
   co-occurrence matrix we extracted

Steps 1 and 2 are one-time, upfront costs. Step 3 can be repeated as many
times as you would like (to, e.g., construct embeddings of different
dimensionality) once the vocabulary and co-coccurrence matrix have been
extracted.

### Vocabulary Extraction

To extract a vocabulary from your data, you will need to add the following
section (with parameters adjusted according to your needs) to your
configuration file:

{% highlight toml %}
[embeddings]
prefix = "path/to/store/model/files"
filter = [{type = "icu-tokenizer"}, {type = "lowercase"}]
[embeddings.vocab]
min-count = 10
max-size = 400000
{% endhighlight %}

The `prefix` key indicates the folder where you would like to store the
model files. (This path should be created before running the tools.)

The `filter` key is a [filter chain][filter-chains] to use to extract the
token sequences from your data. You can feel free to change this however
you would like, but the chain *must* insert sentence markers (\<s\> and
\</s\>). The chain given above is a reasonable default for learning uncased
word vectors.

In the `embeddings.vocab` table, you can specify how to prune your
vocabulary. Typically, you will either truncate the vocabulary below a
certain frequency count (`min-count`), or you will truncate the vocabulary
at a certain maximum size (`max-size`) to keep only the most frequent
terms. The less data available for a vocabulary item, the worse its word
embedding will be.

Note that even if you limit your vocabulary, the model will always include
an \<unk\> vector that will be returned when querying for out-of-vocabulary
terms.

To extract the vocabulary, you can now run the `embedding-vocab` tool:

{% highlight bash %}
./embedding-vocab config.toml
{% endhighlight %}

The tool will extract a vocab, prune it, and write the output to
`$prefix/vocab.bin`.

### Co-occurrence Matrix Extraction
Once you've extracted your vocabulary, you are ready for the second pass
through the training text that extracts the word co-occurrence statistics.

You can configure a few properties for this process with the following
(optional) values in the `[embeddings]` section of your configuration file.

{% highlight toml %}
window-size = 15
max-ram = 4096
{% endhighlight %}

The `window-size` key indicates the size of the window in which a word is
counted as having co-occurred with another. The window is symmetric, so a
`window-size` of 15 counts another word as having co-occurred if it was
$$\leq$$ 15 words to the left or $$\leq 15$$ words to the right.

The `max-ram` key is a ***heuristic*** memory limit (in MB). The tool will
collect co-occurrence counts until a buffer of this size in RAM is
exhausted, which is then flushed to disk. Higher values create fewer
temporary files and make collection faster, but obviously this should be
set to some value $$\leq$$ available RAM.

To extract the co-coccurrence matrix, you can now run the
`embedding-coocur` tool:

{% highlight bash %}
./embedding-coocur config.toml
{% endhighlight %}

The tool will extract the co-occurrence matrix and write it to the file
`$prefix/coocur.bin`.

### Embedding Training

Now you are ready to train the embeddings themselves on the global
co-occurrence data we extracted in the previous two steps. This process can
be configured with the following (optional) values in the `[embeddings]`
section of your configuration file.

{% highlight toml %}
max-ram = 4096
vector-size = 50
num-threads = 4
max-iter = 25
learning-rate = 0.05
xmax = 100.0
scale = 0.75
unk-num-avg = 100
{% endhighlight %}

- `max-ram`, as before, is a ***heuristic*** memory limit that is used
    during the first phase of the learning algorithm, which shuffles the
    data for the SGD-based trainer.
- `vector-size` indicates the desired dimensionality of the generated word
    embeddings
- `num-threads` indicates the number of concurrent threads to run during
    training. Each thread will operate on its own separate subset of the
    training data, so this should be set low enough to allow concurrent
    access to separate files for each thread. By default, we use one thread
    per "core" (including hyperthreading cores)
- `max-iter` indicates the number of iterations to run the algorithm for.
    More iterations results in better optimization, but this is the major
    time/quality tradeoff setting.
- `learning-rate` is the initial learning rate. You likely won't need to
    adjust this unless you are using truly massive corpora.
- `xmax` indicates the maximum co-occurrence count for which to stop the
    "dampening" that occurs for rare word pairs. You likely won't need to
    adjust this.
- `scale` indicates the exponent used in the scaling function. You likely
    won't need to adjust this.
- `unk-num-avg` indicates the number of rare words to average for
    constructing the \<unk\> word embedding.

You can now train your word embeddings using the `glove` tool:

{% highlight bash %}
./glove config.toml
{% endhighlight %}

The output will be written as two vector files:
`$prefix/embeddings.target.bin` and `$prefix/embeddings.context.bin`.

## Playing with Embeddings

Now that you've learned some word embeddings on your data, you can explore
your dataset with the `interactive-embeddings` tool.

{% highlight bash %}
./interactive-embeddings config.toml
{% endhighlight %}

This tool will prompt you for vector-space queries and report to you the
top 10 most similar words according to cosine distance with your query. For
example, to answer the analogy questions given at the beginning of the
tutorial, we could use the following queries:

- sister - woman + man
- rain - summer + winter
- king - man + woman
- fallen - fell + ate

Any addition or subtraction expression involving at least one word will be
accepted.

## API for Embeddings

If you want to use word embeddings in your own application, you can load
them into a `word_embeddings` object and query it like so:

{% highlight cpp %}
// load embeddings given the [embeddings] configuration group
auto model = embeddings::load_embeddings(config);

// query the model for a specific word
auto embed = model.at("dog");
embed.tid; // the term id for the vector
embed.v;   // the embedding vector for the term

// query the model to convert a term id to a string_view
auto term = model.term(embed.tid);

// query the model to find the top_k similar embeddings
auto top = model.top_k(embed.v);

top[0].e;     // the embedding, with fields tid and v
top[0].score; // the score that this embedding obtained
{% endhighlight %}

[word-embeddings]: https://en.wikipedia.org/wiki/Word_embedding
[cosine]: https://en.wikipedia.org/wiki/Cosine_similarity
[glove]: http://nlp.stanford.edu/projects/glove/
[filter-chains]: analyzers-filters-tutorial.html
