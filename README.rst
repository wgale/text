.. image:: https://circleci.com/gh/pytorch/text.svg?style=svg
    :target: https://circleci.com/gh/pytorch/text

.. image:: https://codecov.io/gh/pytorch/text/branch/master/graph/badge.svg
    :target: https://codecov.io/gh/pytorch/text

.. image:: https://img.shields.io/badge/dynamic/json.svg?label=docs&url=https%3A%2F%2Fpypi.org%2Fpypi%2Ftorchtext%2Fjson&query=%24.info.version&colorB=brightgreen&prefix=v
    :target: https://pytorch.org/text/

torchtext
+++++++++

This repository consists of:

* `torchtext.data <#data>`_: Generic data loaders, abstractions, and iterators for text (including vocabulary and word vectors)
* `torchtext.datasets <#datasets>`_: Pre-built loaders for common NLP datasets

Note: we are currently re-designing the torchtext library to make it more compatible with pytorch (e.g. ``torch.utils.data``). Several datasets have been written with the new abstractions in `torchtext.experimental <https://github.com/pytorch/text/tree/master/torchtext/experimental>`_ folder. We also created an issue to discuss the new abstraction, and users are welcome to leave feedback `link <https://github.com/pytorch/text/issues/664>`_. 


Installation
============

We recommend Anaconda as Python package management system. Please refer to `pytorch.org <https://pytorch.org/>`_ for the detail of PyTorch installation. The following is the corresponding ``torchtext`` versions and supported Python versions.

.. csv-table:: Version Compatibility
   :header: "PyTorch version", "torchtext version", "Supported Python version"
   :widths: 10, 10, 10

   nightly build, master, 3.6+
   1.5, 0.5, 3.5+
   1.4, 0.4, "2.7, 3.5+"
   0.4 and below, 0.2.3, "2.7, 3.5+"

Using conda;::

    conda install -c pytorch torchtext

Using pip;::

    pip install torchtext

Optional requirements
---------------------

If you want to use English tokenizer from `SpaCy <http://spacy.io/>`_, you need to install SpaCy and download its English model::

    pip install spacy
    python -m spacy download en

Alternatively, you might want to use the `Moses <http://www.statmt.org/moses/>`_ tokenizer port in `SacreMoses <https://github.com/alvations/sacremoses>`_ (split from `NLTK <http://nltk.org/>`_). You have to install SacreMoses::

    pip install sacremoses

For torchtext 0.5 and below, ``sentencepiece``::

    conda install -c powerai sentencepiece

Building from source
--------------------

To build torchtext from source, you need ``git``, ``CMake`` and C++11 compiler such as ``g++``.::

    git clone https://github.com/pytorch/text torchtext
    cd torchtext
    git submodule update --init --recursive
    python setup.py clean install
    # or ``python setup.py develop`` if you are making modifications.

**Note**

When building from source, make sure that you have the same C++ compiler as the one used to build PyTorch. A simple way is to build PyTorch from source and use the same environment to build torchtext.
If you are using nightly build of PyTorch, checkout the environment it was built `here (conda) <https://github.com/pytorch/builder/tree/master/conda>`_ and `here (pip) <https://github.com/pytorch/builder/tree/master/manywheel>`_.

Documentation
=============

Find the documentation `here <https://pytorch.org/text/>`_.

Data
====

The data module provides the following:

* Ability to describe declaratively how to load a custom NLP dataset that's in a "normal" format:

  .. code-block:: python

      >>> pos = data.TabularDataset(
      ...    path='data/pos/pos_wsj_train.tsv', format='tsv',
      ...    fields=[('text', data.Field()),
      ...            ('labels', data.Field())])
      ...
      >>> sentiment = data.TabularDataset(
      ...    path='data/sentiment/train.json', format='json',
      ...    fields={'sentence_tokenized': ('text', data.Field(sequential=True)),
      ...            'sentiment_gold': ('labels', data.Field(sequential=False))})

* Ability to define a preprocessing pipeline:

  .. code-block:: python

      >>> src = data.Field(tokenize=my_custom_tokenizer)
      >>> trg = data.Field(tokenize=my_custom_tokenizer)
      >>> mt_train = datasets.TranslationDataset(
      ...     path='data/mt/wmt16-ende.train', exts=('.en', '.de'),
      ...     fields=(src, trg))

* Batching, padding, and numericalizing (including building a vocabulary object):

  .. code-block:: python

      >>> # continuing from above
      >>> mt_dev = datasets.TranslationDataset(
      ...     path='data/mt/newstest2014', exts=('.en', '.de'),
      ...     fields=(src, trg))
      >>> src.build_vocab(mt_train, max_size=80000)
      >>> trg.build_vocab(mt_train, max_size=40000)
      >>> # mt_dev shares the fields, so it shares their vocab objects
      >>>
      >>> train_iter = data.BucketIterator(
      ...     dataset=mt_train, batch_size=32,
      ...     sort_key=lambda x: data.interleave_keys(len(x.src), len(x.trg)))
      >>> # usage
      >>> next(iter(train_iter))
      <data.Batch(batch_size=32, src=[LongTensor (32, 25)], trg=[LongTensor (32, 28)])>

* Wrapper for dataset splits (train, validation, test):

  .. code-block:: python

      >>> TEXT = data.Field()
      >>> LABELS = data.Field()
      >>>
      >>> train, val, test = data.TabularDataset.splits(
      ...     path='/data/pos_wsj/pos_wsj', train='_train.tsv',
      ...     validation='_dev.tsv', test='_test.tsv', format='tsv',
      ...     fields=[('text', TEXT), ('labels', LABELS)])
      >>>
      >>> train_iter, val_iter, test_iter = data.BucketIterator.splits(
      ...     (train, val, test), batch_sizes=(16, 256, 256),
      >>>     sort_key=lambda x: len(x.text), device=0)
      >>>
      >>> TEXT.build_vocab(train)
      >>> LABELS.build_vocab(train)

Datasets
========

The datasets module currently contains:

* Sentiment analysis: SST and IMDb
* Question classification: TREC
* Entailment: SNLI, MultiNLI
* Language modeling: abstract class + WikiText-2, WikiText103, PennTreebank
* Machine translation: abstract class + Multi30k, IWSLT, WMT14
* Sequence tagging (e.g. POS/NER): abstract class + UDPOS, CoNLL2000Chunking
* Question answering: 20 QA bAbI tasks
* Text classification: AG_NEWS, SogouNews, DBpedia, YelpReviewPolarity, YelpReviewFull, YahooAnswers, AmazonReviewPolarity, AmazonReviewFull

Others are planned or a work in progress:

* Question answering: SQuAD

See the ``test`` directory for examples of dataset usage.

Experimental Code
=================

We have re-written several datasets under ``torchtext.experimental.datasets``:

* Sentiment analysis: IMDb
* Language modeling: abstract class + WikiText-2, WikiText103, PennTreebank

A new pattern is introduced in `Release v0.5.0 <https://github.com/pytorch/text/releases>`_. Several other datasets are also in the new pattern:

* Unsupervised learning dataset: Enwik9
* Text classification: AG_NEWS, SogouNews, DBpedia, YelpReviewPolarity, YelpReviewFull, YahooAnswers, AmazonReviewPolarity, AmazonReviewFull

Disclaimer on Datasets
======================

This is a utility library that downloads and prepares public datasets. We do not host or distribute these datasets, vouch for their quality or fairness, or claim that you have license to use the dataset. It is your responsibility to determine whether you have permission to use the dataset under the dataset's license.

If you're a dataset owner and wish to update any part of it (description, citation, etc.), or do not want your dataset to be included in this library, please get in touch through a GitHub issue. Thanks for your contribution to the ML community!
