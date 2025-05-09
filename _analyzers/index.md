---
layout: default
title: Text analysis
has_children: true
nav_order: 5
nav_exclude: true
has_toc: false
permalink: /analyzers/
redirect_from: 
  - /opensearch/query-dsl/text-analyzers/
  - /query-dsl/analyzers/text-analyzers/
  - /analyzers/text-analyzers/
  - /analyzers/index/
---

# Text analysis

When you are searching documents using a full-text search, you want to receive all relevant results. If you're looking for "walk", you're interested in results that contain any form of the word, like "Walk", "walked", or "walking". To facilitate full-text search, OpenSearch uses text analysis.

The objective of text analysis is to split the unstructured free text content of the source document into a sequence of terms, which are then stored in an inverted index. Subsequently, when a similar text analysis is applied to a user's query, the resulting sequence of terms facilitates the matching of relevant source documents.

From a technical point of view, the text analysis process consists of several steps, some of which are optional:

1. Before the free text content can be split into individual words, it may be beneficial to refine the text at the character level. The primary aim of this optional step is to help the tokenizer (the subsequent stage in the analysis process) generate better tokens. This can include removal of markup tags (such as HTML) or handling specific character patterns (like replacing the &#x1F642; emoji with the text `:slightly_smiling_face:`).

2. The next step is to split the free text into individual words---_tokens_. This is performed by a _tokenizer_. For example, after tokenization, the sentence `Actions speak louder than words` is split into tokens `Actions`, `speak`, `louder`, `than`, and `words`.

3. The last step is to process individual tokens by applying a series of token filters. The aim is to convert each token into a predictable form that is directly stored in the index, for example, by converting them to lowercase or performing stemming (reducing the word to its root). For example, the token `Actions` becomes `action`, `louder` becomes `loud`, and `words` becomes `word`.

Although the terms ***token*** and ***term*** may sound similar and are occasionally used interchangeably, it is helpful to understand the difference between the two. In the context of Apache Lucene, each holds a distinct role. A ***token*** is created by a tokenizer during text analysis and often undergoes a number of additional modifications as it passes through the chain of token filters. Each token is associated with metadata that can be further used during the text analysis process. A ***term*** is a data value that is directly stored in the inverted index and is associated with much less metadata. During search, matching operates at the term level.
{: .note}

## Analyzers

In OpenSearch, the abstraction that encompasses text analysis is referred to as an _analyzer_. Each analyzer contains the following sequentially applied components:

1. **Character filters**: First, a character filter receives the original text as a stream of characters and adds, removes, or modifies characters in the text. For example, a character filter can strip HTML characters from a string so that the text `<p><b>Actions</b> speak louder than <em>words</em></p>` becomes `\nActions speak louder than words\n`. The output of a character filter is a stream of characters.

1. **Tokenizer**: Next, a tokenizer receives the stream of characters that has been processed by the character filter and splits the text into individual _tokens_ (usually, words). For example, a tokenizer can split text on white space so that the preceding text becomes [`Actions`, `speak`, `louder`, `than`, `words`]. Tokenizers also maintain metadata about tokens, such as their starting and ending positions in the text. The output of a tokenizer is a stream of tokens.

1. **Token filters**: Last, a token filter receives the stream of tokens from the tokenizer and adds, removes, or modifies tokens. For example, a token filter may lowercase the tokens so that `Actions` becomes `action`, remove stopwords like `than`, or add synonyms like `talk` for the word `speak`.

An analyzer must contain exactly one tokenizer and may contain zero or more character filters and zero or more token filters.
{: .note}

There is also a special type of analyzer called a ***normalizer***. A normalizer is similar to an analyzer except that it does not contain a tokenizer and can only include specific types of character filters and token filters. These filters can perform only character-level operations, such as character or pattern replacement, and cannot perform operations on the token as a whole. This means that replacing a token with a synonym or stemming is not supported. See [Normalizers]({{site.url}}{{site.baseurl}}/analyzers/normalizers/) for further details.

## Supported analyzers

For a list of supported analyzers, see [Analyzers]({{site.url}}{{site.baseurl}}/analyzers/supported-analyzers/index/).

## Custom analyzers

If needed, you can combine tokenizers, token filters, and character filters to create a custom analyzer. For more information, see [Creating a custom analyzer]({{site.url}}{{site.baseurl}}/analyzers/custom-analyzer/).

## Text analysis at indexing time and query time

OpenSearch performs text analysis on text fields when you index a document and when you send a search request. Depending on the time of text analysis, the analyzers used for it are classified as follows:

- An _index analyzer_ performs analysis at indexing time: When you are indexing a [text]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/text/) field, OpenSearch analyzes it before indexing it. For more information about ways to specify index analyzers, see [Index analyzers]({{site.url}}{{site.baseurl}}/analyzers/index-analyzers/).

- A _search analyzer_ performs analysis at query time: OpenSearch analyzes the query string when you run a full-text query on a text field. For more information about ways to specify search analyzers, see [Search analyzers]({{site.url}}{{site.baseurl}}/analyzers/search-analyzers/).

In most cases, you should use the same analyzer at both indexing and search time because the text field and the query string will be analyzed in the same way and the resulting tokens will match as expected.
{: .tip}

### Example

When you index a document that has a text field with the text `Actions speak louder than words`, OpenSearch analyzes the text and produces the following list of tokens: 

Text field tokens = [`action`, `speak`, `loud`, `than`, `word`]

When you search for documents that match the query `speaking loudly`, OpenSearch analyzes the query string and produces the following list of tokens:

Query string tokens = [`speak`, `loud`]

Then OpenSearch compares each token in the query string against the list of text field tokens and finds that both lists contain the tokens `speak` and `loud`, so OpenSearch returns this document as part of the search results that match the query.

## Testing an analyzer

To test a built-in analyzer and view the list of tokens it generates when a document is indexed, you can use the [Analyze API]({{site.url}}{{site.baseurl}}/api-reference/analyze-apis/#apply-a-built-in-analyzer).

Specify the analyzer and the text to be analyzed in the request:

```json
GET /_analyze
{
  "analyzer" : "standard",
  "text" : "Let’s contribute to OpenSearch!"
}
```
{% include copy-curl.html %}

The following image shows the query string.

![Query string with indices]({{site.url}}{{site.baseurl}}/images/string-indices.png)

The response contains each token and its start and end offsets that correspond to the starting index in the original string (inclusive) and the ending index (exclusive):

```json
{
  "tokens": [
    {
      "token": "let’s",
      "start_offset": 0,
      "end_offset": 5,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "contribute",
      "start_offset": 6,
      "end_offset": 16,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "to",
      "start_offset": 17,
      "end_offset": 19,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "opensearch",
      "start_offset": 20,
      "end_offset": 30,
      "type": "<ALPHANUM>",
      "position": 3
    }
  ]
}
```

## Verifying analyzer settings

To verify which analyzer is associated with which field, you can use the get mapping API operation:

```json
GET /testindex/_mapping
```
{% include copy-curl.html %}

The response provides information about the analyzers for each field:

```json
{
  "testindex": {
    "mappings": {
      "properties": {
        "text_entry": {
          "type": "text",
          "analyzer": "simple",
          "search_analyzer": "whitespace"
        }
      }
    }
  }
}
```

## Normalizers

Tokenization divides text into individual terms, but it does not address variations in token forms. Normalization resolves these issues by converting tokens into a standard format. This ensures that similar terms are matched appropriately, even if they are not identical.

### Normalization techniques

The following normalization techniques can help address variations in token forms:

1. **Case normalization**: Converts all tokens to lowercase to ensure case-insensitive matching. For example, "Hello" is normalized to "hello".

2. **Stemming**: Reduces words to their root form. For instance, "cars" is stemmed to "car" and "running" is normalized to "run".

3. **Synonym handling:** Treats synonyms as equivalent. For example, "jogging" and "running" can be indexed under a common term, such as "run".

### Normalization

A search for `Hello` will match documents containing `hello` because of case normalization.

A search for `cars` will also match documents containing `car` because of stemming.

A query for `running` can retrieve documents containing `jogging` using synonym handling.

Normalization ensures that searches are not limited to exact term matches, allowing for more relevant results. For instance, a search for `Cars running` can be normalized to match `car run`.

## Next steps

- Learn more about specifying [index analyzers]({{site.url}}{{site.baseurl}}/analyzers/index-analyzers/) and [search analyzers]({{site.url}}{{site.baseurl}}/analyzers/search-analyzers/).
- See the list of [supported analyzers]({{site.url}}{{site.baseurl}}/analyzers/supported-analyzers/index/).