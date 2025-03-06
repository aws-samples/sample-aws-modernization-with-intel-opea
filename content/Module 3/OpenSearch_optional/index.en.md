---
title : "Explore OpenSearch (Optional)"
weight : 201
---

OpenSearch provides a wealth of features for searching your data. In this section, you will explore OpenSearch's query API to see some of the different capabilities.

## Schema and mapping

When you use OpenSearch, you can apply a schema, called the `mapping` to your index. The mapping determines how OpenSearch analyzes and enables the fields in your JSON documents for retrieval. OpenSearch has automatic schema detection for quick starts, but for most situations, it's best to set the schema directly. You do this when you create the index. Of coourse, OPEA and the OpenSearch microservice have set the mapping for you, but you can retrieve it for the embeddings index with the below command.

:::code{showCopyAction=true language=bash}
curl -XGET https://localhost:9200/rag-opensearch/_mapping --insecure -u admin:strongOpea0! | jq .
:::

You should see a response like this:

:::code{language=JSON}
{
    "rag-opensearch": {
        "mappings": {
            "properties": {
                "metadata": {
                    "properties": {
                        "source": {
                            "type": "text",
                            "fields": { "keyword": { "type": "keyword", "ignore_above": 256 } }
                        }
                    }
                },
                "text": {
                    "type": "text",
                    "fields": { "keyword": { "type": "keyword", "ignore_above": 256 } }
                },
                "vector_field": {
                    "type": "knn_vector",
                    "dimension": 768,
                    "method": {
                        "engine": "nmslib",
                        "space_type": "l2",
                        "name": "hnsw",
                        "parameters": {
                            "ef_construction": 512,
                            "m": 16
}}}}}}}
:::

There are three fields for the `rag-opensearch` index: `metadata`, `text`, and `vector_field`. The `metadata` field expects nested JSON, with a `source` field. You access nested JSON for queries using dot notation - `metadata.source`. You can see that the `metadata.source`, and `text` fields are defined as type `text` with a subfield of type `keyword`. `text` type fields are [parsed, and their terms are `analyzed`](https://opensearch.org/docs/latest/analyzers/) to produce tokens for matching. `keyword` fields, in contrast, are [`normalized` and queried for exact match](https://opensearch.org/docs/latest/analyzers/normalizers/). The final field is the `vector_field`, a `knn_vector` type field to hold vectors with 768 dimensions. The storage engine is [Non-Metric Space Library](https://github.com/nmslib/), using the Hidden Navigable Small Worlds (HNSW) algorithm. 

## Lexical queries

OpenSearch is a lexical search engine in addition to being a vector search engine. You can query the rag content using text queries. Use the below query to search the OpenSearch documents, with its default TF/IDF ranking algorithm (see also [Okapi BM25, text-based ranking](https://en.wikipedia.org/wiki/Okapi_BM25)).

:::alert
If your Cloud Shell has terminated, you may need to reestablish port forwarding to the OpenSearch microservice. See the previous module for instructions.
:::

Execute the following command to run the query

:::code{showCopyAction=true language=bash}
curl -XGET https://localhost:9200/rag-opensearch/_search \
  --insecure -u "admin:strongOpea0!" \
  -H "Content-type: application/json" \
  -d '{"query": {
    "simple_query_string": {
      "query": "what is nike 2023 revenue?",
      "fields": ["text"]
    }
  },
  "_source": "False",
  "size": 1,
  "highlight": {
    "fields": {
      "text": {}
    }
  }
}' | jq
:::

The first line of the above command executes the `curl` command. The URL contains the endpoint (`localhost:9200`, which is forwarded to the OpenSearch microservice) and the API specification. In this case, you are directing the query to the `rag-opensearch` index, and calling the `_search` API. The following lines specify the TLS and authentication parameters and then, following the `-d` parameter, the body of the request specifies the query.

This query is a [simple query string query](https://opensearch.org/docs/latest/query-dsl/full-text/simple-query-string/), for the text "what is nike's 2023 revenue?". `simple_query_string` takes a text query, analyzes the text to produce individual tokens -- technically `terms` -- and matches them to the `text` field in all of the documents in the index. It scores and sorts the results based on the TF/IDF scoring algorithm to bring the most relevant document to the top. 

The additional directives in the query tell OpenSearch to remove all fields from the result (`"_source": "False"` -- you can remove this directive to see the original values from the document), return only 1 result (`"size": 1`), and highlight matching terms in the `text` source field `"highlight": ...`).

Finally, the command line pipes the query results to `jq` to format the json and make it legible.

You should see output like this:

```
{
  "took": 8,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 244,
      "relation": "eq"
    },
    "max_score": 6.5231514,
    "hits": [
      {
        "_index": "rag-opensearch",
        "_id": "c0d241e5-59f9-4e5f-a1e3-0cd47c09ecf5",
        "_score": 6.5231514,
        "_source": {},
        "highlight": {
          "text": [
            "We believe providing for growth and retention of our employees <em>is</em> essential in fostering such a\nculture",
            "The program also measures our employees’ emotional\ncommitment to <em>NIKE</em> as well as NIKE's culture of diversity",
            "<em>NIKE</em> also provides multiple points of contact for employees to speak up if they experience\nsomething",
            "not align with our values or otherwise violates our workplace policies, even if they are uncertain <em>what</em>",
            "they observed or heard <em>is</em> a violation of\ncompany policy."
          ]
        }
      }
    ]
  }
}
```

OpenSearch's response contains an initial metadata section that reports the `took` (server side) time for processing the query (8 ms in this case), whether the query timed out, and a section on the shards responding. There follows the `hits` (matches), with the `total` number of matches, the maximum score, and then the matching documents themselves. Each document has the `_index` it came from, the document's `_id`, the score for that document, the source fields (excluded by the query in this example), and a set of highlight snippets. The highlights use HTML `em` tags to show where the query terms matched portions of the document.

This response is not so great. You can see that the response has mentions of Nike, but includes unimportant terms like `what` and `is`. 

You can experiment with different query terms, replacing the `"query"` text ("what is nike 2023 revenue?") to see some other responses. It's also interesting to see more than one result. Set `"size"` to `2` or more and compare the output. Try "workplace policies", "footwear apparel", "men sales", and "women sales".

## Vector queries - exact

Try running the query as an exact k-Nearest-Neighbor query. First, you'll need to get the embedding for the query. Use the below command to retrieve the embedding from the `chatqna-tei` microservice. This microservice returns an array of arrays, but you just need the inner array for the OpenSearch query. The `jq` pops the 0th element and puts it in `$embedding`. 

In order to run the command, you'll need the port you used to forward to the tei microservice above. You can use the `ps aux | grep kubectl` command-line command to see which processes are running and what ports they are mapping. If you have followed along with the guide, you should have the `chatqna-tei` microservice running on port 9800.

:::code{showCopyAction=true language=bash}
embedding=$(curl -X POST localhost:9800/embed \
    -d '{"inputs":"What was the Nike revenue in 2023?"}' \
    -H 'Content-Type: application/json')
embedding=`echo $embedding | jq '.[0]'`
:::

You can use `echo $embedding` to see the generated embedding. Now you'll create the local file `query.json` with the embeding merged into the query, and then run an exact k-Nearest-Neighbors (k-NN) query to compare the query embeddingg to every document (chunk) in the index and retrieve the closest matches.

:::code{showCopyAction=true language=bash}
echo "{ \
    \"size\": 2, \
    \"_source\": \"text\", \
    \"query\": { \
      \"script_score\": { \
        \"query\": { \"match_all\": {} }, \
        \"script\": { \
          \"source\": \"knn_score\", \
          \"lang\": \"knn\", \
          \"params\": { \
            \"field\": \"vector_field\", \
            \"query_value\": ${embedding}, \
            \"space_type\": \"l2\" \
    }}}}}" | jq -c . > query.json
curl -XGET https://localhost:9200/rag-opensearch/_search \
  --insecure -u "admin:strongOpea0!" \
  -H "Content-type: application/json" \
  -d @query.json | jq .
:::

This query is a `script_score` query, employing a saved script to do k-NN score calculation, comparing the query vector to every document in the index. The `script_score` query includes a sub-`query`, which you can use to apply filters to non-vector fields. ChatQnA just sends the file key in the `metadata.source` field, so this query just uses a `match_all`, which matches every document in the index. The `script` portion of the query specifies the `knn_score` script, with parameters that tell the script which `field` has the vector embedding for the doc, passes the embedding as the `query_value` and specifies **l2** as the distance metric (`space_type`).

You should see a response like this:

```
{
  "took": 31,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 271,
      "relation": "eq"
    },
    "max_score": 0.74458516,
    "hits": [
      {
        "_index": "rag-opensearch",
        "_id": "ebe97241-0b49-4f4f-9183-f2976dd1086d",
        "_score": 0.74458516,
        "_source": {
          "text": "believe will further accelerate our digital transformation. We believe this unified approach will accelerate growth and unlock more efficiency for our business, while driving\nspeed and responsiveness as we serve consumers globally.\nFINANCIAL HIGHLIGHTS\n• In fiscal 2023, NIKE, Inc. achieved record Revenues of $51.2 billion, which increased 10% and 16% on a reported and currency-neutral basis, respectively\n• NIKE Direct revenues grew 14% from $18.7 billion in fiscal 2022 to $21.3 billion in fiscal 2023, and represented approximately 44% of total NIKE Brand revenues for\nfiscal 2023\n• Gross margin for the fiscal year decreased 250 basis points to 43.5% primarily driven by higher product costs, higher markdowns and unfavorable changes in foreign\ncurrency exchange rates, partially offset by strategic pricing actions\n• Inventories as of May 31, 2023 were $8.5 billion, flat compared to the prior year, driven by the actions we took throughout fiscal 2023 to manage inventory levels\n• We returned $7.5 billion to our shareholders in fiscal 2023 through share repurchases and dividends\n• Return on Invested Capital (\"ROIC\") as of May 31, 2023 was 31.5% compared to 46.5% as of May 31, 2022. ROIC is considered a non-GAAP financial measure, see\n\"Use of Non-GAAP Financial Measures\" for further information.\nFor discussion related to the results of operations and changes in financial condition for fiscal 2022 compared to fiscal 2021 refer to Part II, Item 7. Management's"
        }
      },
      {
        "_index": "rag-opensearch",
        "_id": "2a741638-cf9a-4b40-b48e-32d18c4e3046",
        "_score": 0.7338308,
        "_source": {
          "text": "(INCL. HEDGES) SELLING PRICE\n(NET OF\nDISCOUNTS)*.Table of Contents\nFISCAL 2023 NIKE BRAND REVENUE HIGHLIGHTS\nThe following tables present NIKE Brand revenues disaggregated by reportable operating segment, distribution channel and major product line:\nFISCAL 2023 COMPARED TO FISCAL 2022\n• NIKE, Inc. Revenues were $51.2 billion in fiscal 2023, which increased 10% and 16% compared to fiscal 2022 on a reported and currency-neutral basis, respectively.\nThe increase was due to higher revenues in North America, Europe, Middle East & Africa (\"EMEA\"), APLA and Greater China, which contributed approximately 7, 6,\n2 and 1 percentage points to NIKE, Inc. Revenues, respectively.\n• NIKE Brand revenues, which represented over 90% of NIKE, Inc. Revenues, increased 10% and 16% on a reported and currency-neutral basis, respectively. This\nincrease was primarily due to higher revenues in Men's, the Jordan Brand, Women's and Kids' which grew 17%, 35%,11% and 10%, respectively, on a wholesale\nequivalent basis.\n• NIKE Brand footwear revenues increased 20% on a currency-neutral basis, due to higher revenues in Men's, the Jordan Brand, Women's and Kids'. Unit\nsales of footwear increased 13%, while higher average selling price (\"ASP\") per pair contributed approximately 7 percentage points of footwear revenue\ngrowth. Higher ASP was primarily due to higher full-price ASP, net of discounts, on a wholesale equivalent basis, and growth in the size of our NIKE Direct"
}}]}}
```

This is the correct response. Notice that the first result contains the text "NIKE, Inc. Revenues were $51.2 billion in fiscal 2023". Highlighting is not supported for vector fields (there's no source text!) so for this query you just received the contents of the `text` field unprocessed.

Exact k-NN is great when you have relatively few documents. However as your document set grows, latency will grow along with it and beyond a few 100s of 1000s of documents, will be too slow. Instead, you can use approximate nearest neighbor search. The below command uses HNSW to find the nearest neighbors. 

:::code{showCopyAction=true language=bash}
echo "{ \
  \"size\": 2, \
  \"_source\": \"text\", \
  \"query\": { \
    \"knn\": { \
      \"vector_field\": { \
        \"vector\": ${embedding}, \
        \"k\": 10 \
      }}}}" | jq -c . > query.json
curl -XGET https://localhost:9200/rag-opensearch/_search \
  --insecure -u "admin:strongOpea0!" \
  -H "Content-type: application/json" \
  -d @query.json | jq .
:::

This query is a `knn` query, which uses the algorithm you specified in the field mapping to determine the nearest neighbors. You just pass in a `vector` and a value for `k` (the count of neighbors to retrieve), and opensearch does the rest.

Again, you can see the correct document is the first result retrieved. Using approximate k-NN you can scale your OpenSearch vector database to billions of vectors and 1000s of queries per second.

## Hybrid search with OpenSearch

[OpenSearch supports hybrid search](https://opensearch.org/docs/latest/search-plugins/hybrid-search/) -- where you specify both a lexical and vector query, along with a normalization and merge strategy. OpenSearch runs both queries normalizes and merges the results. When you perform hybrid search, you set a [Search Pipeline](https://opensearch.org/docs/latest/search-plugins/search-pipelines/index/), and send queries through that pipeline. Use the below command to use OpenSearch's REST API to set a search pipeline.

:::code{showCopyAction=true language=bash}
curl -XPUT https://localhost:9200/_search/pipeline/nlp-search-pipeline \
  --insecure -u "admin:strongOpea0!" \
  -H "Content-type: application/json" \
  -d '{
  "description": "Hybrid search pipeline",
  "phase_results_processors": [
    {
      "normalization-processor": {
        "normalization": {
          "technique": "min_max"
        },
        "combination": {
          "technique": "arithmetic_mean",
          "parameters": {
            "weights": [
              0.3,
              0.7
]}}}}]}'
:::

This pipeline uses [min/max normalization](https://en.wikipedia.org/wiki/Normalization_(statistics)) to set all of the lexical and vector scores in the range `[0, 1]`. It uses the arithmetic mean to combine the scores, with a weight of `0.3` for the first query clause and `0.7` for the second query clause. Note, this is not a query itself, when you send queries to this search pipeline, OpenSearch applies the weights. The `phase_results_processor` is a flexible, generic construct - the query clauses can be either lexical or vector.

To use the pipeline, you send a query to the pipeline API. Use the below command to create the query in the file `hybrid_query.json`.

:::code{showCopyAction=true language=bash}
echo "{ \
  \"_source\": { \"excludes\": \"vector_field\"}, \
  \"size\": 4, \
  \"query\": { \
    \"hybrid\": { \
      \"queries\": [{ \
          \"match\": { \
            \"text\": { \"query\": \"footwear revenue\" } \
          }}, \
          { \"knn\": { \
              \"vector_field\": { \
                \"vector\": ${embedding}, \
                \"k\": 2 \
}}}]}}}" | jq -c . > hybrid_query.json
curl -XGET https://localhost:9200/rag-opensearch/_search?search_pipeline=nlp-search-pipeline \
  --insecure -u "admin:strongOpea0!" \
  -H "Content-type: application/json" \
  -d @hybrid_query.json | jq .
:::

This hybrid query contains two sub-queries - a lexical query for "footwear revenue" and a vector query with an embedding representing "What is Nike 2023 revenue?". The value for `k`, 2, ensures that the results will contain at most two vector matches.

The `curl` command sends this query to the `nlp-search-pipeline` you just defined. You should get these results (we've removed the result metadata and other portions to show the relevant output)

```
"_score": 0.7,
"text": "believe will further accelerate our digital transformation. We believe this unified
approach will accelerate growth and unlock more efficiency for our business, while driving\nspeed
and responsiveness as we serve consumers globally.\nFINANCIAL HIGHLIGHTS\n• In fiscal 2023, NIKE,
Inc. achieved record Revenues of $51.2 billion, which increased 10% and 16% on a reported and
currency-neutral basis, respectively\n• NIKE Direct revenues grew 14% from $18.7 billion in fiscal
2022 to $21.3 billion in fiscal 2023, and represented approximately 44% of total NIKE Brand revenues
for\nfiscal 2023\n• Gross margin for the fiscal year decreased 250 basis points to 43.5% primarily
driven by higher product costs, higher markdowns and unfavorable changes in foreign\ncurrency
exchange rates, partially offset by strategic pricing actions\n• Inventories as of May 31, 2023 were
$8.5 billion, flat compared to the prior year, driven by the actions we took throughout fiscal 2023
to manage inventory levels\n• We returned $7.5 billion to our shareholders in fiscal 2023 through
share repurchases and dividends\n• Return on Invested Capital (\"ROIC\") as of May 31, 2023 was
31.5% compared to 46.5% as of May 31, 2022. ROIC is considered a non-GAAP financial measure,
see\n\"Use of Non-GAAP Financial Measures\" for further information.\nFor discussion related to the
results of operations and changes in financial condition for fiscal 2022 compared to fiscal 2021
refer to Part II, Item 7. Management's"

"_score": 0.53328085,
"text": "(INCL. HEDGES) SELLING PRICE\n(NET OF\nDISCOUNTS)*.Table of Contents\nFISCAL 2023 NIKE
BRAND REVENUE HIGHLIGHTS\nThe following tables present NIKE Brand revenues disaggregated by
reportable operating segment, distribution channel and major product line:\nFISCAL 2023 COMPARED TO
FISCAL 2022\n• NIKE, Inc. Revenues were $51.2 billion in fiscal 2023, which increased 10% and 16%
compared to fiscal 2022 on a reported and currency-neutral basis, respectively.\nThe increase was
due to higher revenues in North America, Europe, Middle East & Africa (\"EMEA\"), APLA and Greater
China, which contributed approximately 7, 6,\n2 and 1 percentage points to NIKE, Inc. Revenues,
respectively.\n• NIKE Brand revenues, which represented over 90% of NIKE, Inc. Revenues, increased
10% and 16% on a reported and currency-neutral basis, respectively. This\nincrease was primarily due
to higher revenues in Men's, the Jordan Brand, Women's and Kids' which grew 17%, 35%,11% and 10%,
respectively, on a wholesale\nequivalent basis.\n• NIKE Brand footwear revenues increased 20% on a
currency-neutral basis, due to higher revenues in Men's, the Jordan Brand, Women's and
Kids'. Unit\nsales of footwear increased 13%, while higher average selling price (\"ASP\") per pair
contributed approximately 7 percentage points of footwear revenue\ngrowth. Higher ASP was primarily
due to higher full-price ASP, net of discounts, on a wholesale equivalent basis, and growth in the
size of our NIKE Direct"

"_score": 0.42839527,
"text": "RESULTS OF OPERATIONS\n(Dollars in millions, except per share data)\nFISCAL 2023\nFISCAL
2022\n% CHANGE\nFISCAL 2021\n% CHANGE\nRevenues\n$\n51,217 \n$\n46,710 \n10 % $\n44,538 \n5 %\nCost
of sales\n28,925 \n25,231 \n15 %\n24,576 \n3 %\nGross
profit\n22,292 \n21,479 \n4 %\n19,962 \n8 %\nGross margin\n43.5 %\n46.0 %\n44.8 %\nDemand creation
expense\n4,060 \n3,850 \n5 %\n3,114 \n24 %\nOperating overhead
expense\n12,317 \n10,954 \n12 %\n9,911 \n11 %\nTotal selling and administrative
expense\n16,377 \n14,804 \n11 %\n13,025 \n14 %\n% of revenues\n32.0 %\n31.7 %\n29.2 %\nInterest
expense (income), net\n(6)\n205 \n— \n262 \n— \nOther (income) expense,
net\n(280)\n(181)\n— \n14 \n— \nIncome before income
taxes\n6,201 \n6,651 \n-7 %\n6,661 \n0 %\nIncome tax
expense\n1,131 \n605 \n87 %\n934 \n-35 %\nEffective tax rate\n18.2 %\n9.1 %\n14.0 %\nNET
INCOME\n$\n5,070 \n$\n6,046 \n-16 % $\n5,727 \n6 %\nDiluted earnings per common
share\n$\n3.23 \n$\n3.75 \n-14 % $\n3.56 \n5 %\n2023 FORM 10-K 31.Table of Contents\nCONSOLIDATED
OPERATING RESULTS\nREVENUES\n(Dollars in millions)\nFISCAL\n2023\nFISCAL\n2022\n% CHANGE\n%
CHANGE\nEXCLUDING\nCURRENCY\nCHANGES\nFISCAL\n2021\n% CHANGE\n%
CHANGE\nEXCLUDING\nCURRENCY\nCHANGES\nNIKE, Inc. Revenues:\nNIKE Brand Revenues
by:\nFootwear\n$\n33,135  $\n29,143 \n14 %\n20 %
$\n28,021 \n4 %\n4 %\nApparel\n13,843 \n13,567 \n2 %\n8 %\n12,865 \n5 %\n6 %\nEquipment\n1,727 \n1,624 \n6 %\n13 %\n1,382 \n18 %\n18 %\nGlobal
Brand Divisions\n58 \n102 \n-43 %\n-43 %\n25 \n308 %\n302 %\nTotal NIKE Brand Revenues\n$\n48,763 
$\n44,436 \n10 %\n16 %
$\n42,293 \n5 %\n6 %\nConverse\n2,427 \n2,346 \n3 %\n8 %\n2,205 \n6 %\n7 %\nCorporate\n27"

"_score": 0.3,
"text": "• Footwear revenues increased 25% on a currency-neutral basis, due to higher revenues in
Men's, the Jordan Brand, Women's and Kids'. Unit sales of footwear\nincreased 9%, while higher ASP
per pair contributed approximately 16 percentage points of footwear revenue growth. Higher ASP per
pair was primarily due to\nhigher full-price ASP and growth in NIKE Direct.\n• Apparel revenues
increased 14% on a currency-neutral basis, primarily due to higher revenues in Men's. Unit sales of
apparel increased 2%, while higher ASP per\nunit contributed approximately 12 percentage points of
apparel revenue growth. Higher ASP per unit was primarily due to higher full-price ASP and growth in
NIKE\nDirect, partially offset by lower NIKE Direct ASP, reflecting higher promotional
activity.\nReported EBIT increased 7% due to higher revenues and the following:\n• Gross margin
contraction of 60 basis points primarily due to higher product costs reflecting higher input costs,
inbound freight and logistics costs and product mix,\nhigher other costs and unfavorable changes in
standard foreign currency exchange rates. This was partially offset by higher full-price ASP, net of
discounts, primarily\ndue to strategic pricing actions and product mix.\n• Selling and
administrative expense increased 4% due to higher operating overhead and demand creation
expense. Operating overhead expense increased primarily"
```

You can see the first match is identical to the approximate nearest-neighbor query above. The second and fourth match are from the lexical query "footwear revenue", and the second query is from the vector match. You've blended the more abstract, semantic information on revenue with the more concrete matches for "footwear revenue". When building search systems, you may not know exactly the types of queries that your users will run. By employing hybrid queries like this one, you can present results that are semantic and results that are lexical.

You can also swap the weights in the `nlp-search-pipeline` to see how that changes the order of the results. If you set the weight for the first clause (the lexical clause in the query) to 0.9 and the vector to 0.1, you'll see the first result changes to the result that begins `• Footwear revenues increased 25% on a currency-neutral basis,...`, with a score of 0.9, and the vector match which contains the text `In fiscal 2023, NIKE, Inc. achieved record Revenues of $51.2 billion` moves to the second position, with a score of 0.1. All of the other vector matches come after that.

## Conclusion

You've executed and compared lexical, exact k-NN, approximate k-NN, and hybrid search directly against OpenSearch's APIs. At present, OPEA uses only approximate k-NN. In the future we hope to bring these more advanced techniques!