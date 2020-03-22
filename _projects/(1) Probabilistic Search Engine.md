---
name: Probabilistic Search Engine
tools: [Information Retrieval, BM25, SPIMI, Python]
image: https://cdn.dribbble.com/users/1751513/screenshots/4006096/search_site_ux_v2_800x600.jpg
description: Develop a probabilistic search engine based on BM25 algorithm.
---

# Probabilistic Search Engine

![preview](https://cdn.dribbble.com/users/108183/screenshots/4605344/search_icon_interaction.gif)

## 1. Background

In this project, I will apply the **BM25** algorithm to convert my indexer, implemented in the last project using the SPIMI algorithm, into a probabilistic search engine. And then I will do comprehensive tests and analyze resuts.

## 2. Implementation

Based on the SPIMI indexer implemented in last project, I will use the BM25 algorithm, known as Okapi BM25, as my default similarity algorithm, which is what’s used to score results as they relate to a query. 

>In information retrieval, BM25 (BM stands for Best Matching) is a ranking function used by search engines to rank matching documents according to their relevance to a given search query. It is based on the probabilistic retrieval framework developed in the 1970s and 1980s by Stephen E. Robertson, Karen Spärck Jones, and others.

### 2.1 Techniques

**BM25** is a bag-of-words retrieval function that ranks a set of documents based on the query terms appearing in each document, regardless of the inter-relationship between the query terms within a document (e.g., their relative proximity).

It is not a single function, but actually a whole family of scoring functions, with slightly different components and parameters. One of the most prominent instantiations of the function is as follows.

Given a query **Q**, containing keywords 
$q_1, ...q_n$,the BM25 score of a document **D** is:

$scroe(D, Q) = \sum_{i=1}^{n}{IDF(q_i)\times{\frac{f(q_i, D)\times(k_1 + 1)}{f(q_i, D) + k_1 \times(1 - b + b \times\frac{|D|}{avgdl})}}}$

We can see a few common components like $q_i$, $IDF(q_i)$, $f(q_i,D)$, $k_1$, b, and something about field lengths. Here’s what each of these is all about:

1. **$q_i$** is the ith query term.

    For example, if I search for “shane” there’s only 1 query term, so $q_0$ is “shane”. If I search for “shane connelly” in English, the program will see the whitespace and tokenize this as 2 terms: $q_0$ will be “shane” and $q_1$ will be “connelly”. These query terms are plugged into the other bits of the equation and all of it is summed up.

2. **$IDF(q_i)$** is the inverse document frequency of the ith query term.

    The IDF component of our formula measures how often a term occurs in all of the documents and “penalizes” terms that are common. The actual formula BM25 uses for this part is:

    $IDF(q_i) = \log[\frac{N}{df_t}]$
    * **N** is the number of documents in the collection
    * **$df_t$** is the document frequency of the ith query term.

3. **$f(q_i,D)$** is “how many times does the ith query term occur in document D?”

    The way to think about f(qi,D) is that the more times the query term(s) occur a document, the higher its score will be.

4. **$k_1$** is a variable which helps determine term frequency saturation characteristics.

    That is, it limits how much a single query term can affect the score of a given document. It does this through approaching an asymptote. A higher/lower k1 value means that the slope of “tf() of BM25” curve changes. This has the effect of changing how “terms occurring extra times add extra score.” An interpretation of k1 is that for documents of the average length, it is the value of the term frequency that gives a score of half the maximum score for the considered term. **By default, k1 has a value of 1.2 in the project**.

5. **b** is a variable which shows up in the denominator and that it’s multiplied by the ratio of the field length we just discussed.

    If b is bigger, the effects of the length of the document compared to the average length are more amplified. To see this, you can imagine if you set b to 0, the effect of the length ratio would be completely nullified and the length of the document would have no bearing on the score. **By default, b has a value of 0.75 in the project**.
    
So, the final BM25 formula that I will use in the project is:

$RSV_d = \sum_{i=1}^{n}{\log[\frac{N}{df_t}]\times{\frac{(k_1 + 1)\times {tf_{td}}}{k_1 \times[(1 - b) + b \times\frac{L_d}{Lave})] + tf_{td}}}}$

### 2.2 Implementation

I will implement the project by following four steps:

1. Fetching Documents (**InvertedIndex.py**)
2. Preprocessing Documents (**TextNormalizer.py**)
3. Performing SPIMI (**Inverter.py**)
4. Ranking (**BM25.py**)
4. Querying (**QueryProcessor.py**)

#### 2.2.1 Fetching

This is the module responsible for extracting all of the documents from the corpus. It parses line by line for the <REUTERS> tag, extracts the NEWID attribute, the takes all the content between the following <BODY></BODY> tags. The module returns a collection of documents to the main module for the next step.

```python
path = 'reuters21578/*.sgm'
# path for each reuters file
files = glob.glob(path)
block_files = []
N = 0
doc_length_dict = {}
# loop through each file
for file in files:
    # parse each document separately
    soup = BeautifulSoup(open(file), 'html.parser')
    documents = soup.find_all('reuters')
    N += len(documents)
    tokens = []
    # loop through all the documents
    for doc in documents:
    # tokenize each document
        if doc.body is not None and doc.body.text is not None:
        text = doc.body.text
          doc_id = int(doc['newid'].encode("utf-8"))
          doc_length_dict[doc_id] = len(text.split())
          tokens = tokens + normalizer_method(text, doc_id)
    block_files = block_files + spimi_invert(tokens, block_size)
```

#### 2.2.2 Preprocessing

Then I will implement the **Lossy Dictionary Compression** techniques, known as **Normalization**, by removing numbers, blanks, punctuations etc.

In the part, I will include all the methods regarding the lossy dictionary compression in the **TextNormalizer.py** file.

```python
from nltk.stem import PorterStemmer
from string import punctuation
def unfiltered(text, id):
    return token_tuples(text.split(), id)
def no_numbers(text, id):
    text = remove_numbers(text)
    return token_tuples(text.split(), id)
def case_folding(text, id):
    text = remove_numbers(text)
    tokens = tokenize(text)
    return token_tuples(tokens, id)
def thirty_stop_words(text, id):
    text = remove_numbers(text)
    tokens = tokenize(text)
    tokens = remove_stop_words_30(tokens)
    return token_tuples(tokens, id)
def one_hundred_fifty_stop_words(text, id):
    text = remove_numbers(text)
    tokens = tokenize(text)
    tokens = remove_stop_words_150(tokens)
    return token_tuples(tokens, id)
def stemming(text, id):
    text = remove_numbers(text)
    tokens = tokenize(text)
    tokens = remove_stop_words_150(tokens)
    tokens = stem_tokens(tokens)
    return token_tuples(tokens, id)
```

The detailed statistical data is shown below :

<table class="tg">
  <tr>
    <th></th>
    <th colspan="3"> Distinct Terms </th>
    <th colspan="3"> Non Positional Postings</th>
    <th colspan="3"> Tokens</th>
  </tr>
  <tr>
    <td></td>
    <td> number </td>
    <td> Δ% </td>
    <td> T% </td>
    <td> number </td>
    <td> Δ% </td>
    <td> T% </td>
    <td> number </td>
    <td> Δ% </td>
    <td> T% </td>
  </tr>
  <tr>
    <td> unfiltered </td>
    <td> 129070 </td>
    <td></td>
    <td></td>
    <td> 2918161 </td>
    <td></td>
    <td></td>
    <td> 3091405 </td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td> no numbers </td>
    <td> 69704 </td>
    <td> -32.89% </td>
    <td> -32.89% </td>
    <td> 1621327 </td>
    <td> -8.79% </td>
    <td> -8.79% </td>
    <td> 2625084 </td>
    <td> -9.64% </td>
    <td> -9.64% </td>
  </tr>
  <tr>
    <td> 30 stop words </td>
    <td> 52875 </td>
    <td> -0.24% </td>
    <td> -33.13% </td>
    <td> 1503153 </td>
    <td> -14.04% </td>
    <td> -24.83% </td>
    <td> 2625084 </td>
    <td> -31.54% </td>
    <td> -38.52% </td>
  </tr>
  <tr>
    <td> stemming </td>
    <td> 41783 </td>
    <td> -31.47% </td>
    <td> -64.60% </td>
    <td> 1081540 </td>
    <td> -18.96% </td>
    <td> -12.88% </td>
    <td> 1706555 </td>
    <td> -0% </td>
    <td> -24% </td>
  </tr>
</table>

**Table 2-1** above shows the number of terms for different levels of preprocessing (column 2). The number of terms is the main factor in determining the size of the dictionary. The number of non-positional postings (column 3) is an indicator of the expected size of the non-positional index of the collection. The expected size of a positional index is related to the number of positions it must encode (column 4).

In general, the statistics in Table 2-1 show that preprocessing affects the size of the dictionary and the number of non-positional postings greatly. **Stemming** reduces the number of (distinct) terms by **31.47%** and the number of non-positional postings by **18.96%**. The treatment of the most frequent words is also important. The rule of 30 states that the **30 most common words **account for **33.13%** of the tokens in written text. 


#### 2.2.3 SPIMI

The SPIMI algorithm is implemented here. All of the block files are opened, and their top lines are read into an array. The minimum term alphabetically is identified (as well as duplicates in other first lines), postings lists are combined and the (term, postingList) pair are written into the index file.

#### 2.2.4 Ranking

In the ranking part, I will use the BM25 formual discussed above to calculate the RSV.

```python
def RankDocuments(index_file, inverted_index):
    k1 = 1.2
    b = 0.75
    # fetch collection stats
    with open("collection_stats", 'rb') as stats_files:
        N, doc_length_dict, avg_doc_length = cPickle.load(stats_files)
    stats_files.close()

    doc_length_dict = dict(doc_length_dict)

    for term, postings in inverted_index.items():
        for index, post in enumerate(postings):
            tftd = post[1]
            doc_id = post[0]
            ld = doc_length_dict[doc_id]
            inverted_index[term][index] = (post[0], post[1], calculate_rsv(avg_doc_length, b, k1, ld, tftd))
    with open(index_file, "wb") as output_file:
        cPickle.dump(inverted_index, output_file, protocol=cPickle.HIGHEST_PROTOCOL)
    output_file.close()

def calculate_rsv(avg_doc_length, b, k1, ld, tftd):
    avg_doc_length = float(avg_doc_length)
    b = float(b)
    k1 = float(k1)
    ld = float(ld)
    tftd = float(tftd)
    return float(((k1 + 1) * tftd) / (k1 * (((1 - b) + b * (ld / avg_doc_length))) + tftd))
```

#### 2.2.5 Querying

Here, I will provide the mechanism for processing boolean queries and returning results in the console.

```python
def ranked_search():
    print "This query returns the top 20 results, using BM25 Ranking"
    query = get_user_input("Enter a search query: ")
    inverted_index = get_inverted_index(index_file)
    matches = GetRankedResults(query, inverted_index)
    for match in matches[:20]:
        print "Document ID: %s RSVd: %s" % (str(match[0]), str(match[1]))
```

## 3. Analysis & Test

### 3.1 Scenario 01 - Test Queries I Designed

1. **Case 1**

    ***Purpose:***
    
    Check whether the program can handle the situation and return the correct result when doing a single keyword (**out 0f dictionary**) query.
    
    ***Steps:***
    
    1. Open a termianl and go to the project directory.
    2. Run the **QueryProcessor.py** file with the memory size argument. **python QueryProcessor.py**
    3. Enter a keyword which is not in the dictionary, like **johnnys**
    4. Check the result.
    
    ***Hypothesis & Analysis:***
    
    The console shows **nothing**
    
    ***Results:***
    
<img alt="Scenario 01 Case01" src="/assets/project_imgs/probabilistic_search_engine/single_keyword_query_case02.png" width="50%" height="50%">
    
2. **Case 2**

    ***Purpose:***
    
    Check whether the program can return the correct result when doing a multiple keywords query.
    
    ***Steps:***
    
    1. Open a termianl and go to the project directory.
    2. Run the **QueryProcessor.py** file with the memory size argument. **python QueryProcessor.py**
    3. Enter multiple keywords, like **karim or kids**
    4. Check the result.
    
    ***Hypothesis & Analysis:***
    
    The console shows **"Document ID: 3602 RSVd: 4.79258156572
Document ID: 16211 RSVd: 4.5970339289
Document ID: 7733 RSVd: 4.48897999477
Document ID: 644 RSVd: 3.89785235629
Document ID: 11031 RSVd: 3.86137006891
Document ID: 4751 RSVd: 3.84926124291
Document ID: 4304 RSVd: 3.80157596942
Document ID: 4884 RSVd: 3.74078851365
Document ID: 2219 RSVd: 3.72090931867
Document ID: 13656 RSVd: 3.69276890223
Document ID: 20237 RSVd: 3.54385578501
Document ID: 20634 RSVd: 3.42837894915
Document ID: 19787 RSVd: 3.41724383167
Document ID: 1734 RSVd: 3.21294199835
Document ID: 8023 RSVd: 3.21294199835
Document ID: 2196 RSVd: 3.16652710592
Document ID: 8999 RSVd: 3.14829859423
Document ID: 1084 RSVd: 3.11632715575
Document ID: 11049 RSVd: 3.0668424205
Document ID: 1459 RSVd: 3.05326181993"**
    
    ***Results:***
    
    
<img alt="Scenario 01 Case02" src="/assets/project_imgs/probabilistic_search_engine/multiple_keywords_query_or_case01.png" width="50%" height="50%">

3. **Case 3**

    ***Purpose:***
    
    Check whether the program can return the correct result when doing a multiple keywords query.
    
    ***Steps:***
    
    1. Open a termianl and go to the project directory.
    2. Run the **QueryProcessor.py** file with the memory size argument. **python QueryProcessor.py**
    3. Enter multiple keywords, like **China and America and Canada**
    4. Check the result.
    
    ***Hypothesis & Analysis:***
    
    The console shows **"Document ID: 12099 RSVd: 7.31857501882
Document ID: 17822 RSVd: 6.87862251291
Document ID: 11751 RSVd: 6.6872899477
Document ID: 19097 RSVd: 6.38319547296
Document ID: 3835 RSVd: 6.22082840232
Document ID: 314 RSVd: 6.1786491228
Document ID: 15146 RSVd: 6.02305581972
Document ID: 18467 RSVd: 6.00009003099
Document ID: 8143 RSVd: 5.97777021357
Document ID: 4041 RSVd: 5.81501042011
Document ID: 4125 RSVd: 5.81501042011
Document ID: 17704 RSVd: 5.51386239759
Document ID: 17235 RSVd: 5.46280878741
Document ID: 3966 RSVd: 5.25812402851
Document ID: 516 RSVd: 5.12488472774
Document ID: 18201 RSVd: 5.09456921888
Document ID: 13884 RSVd: 4.87054093075
Document ID: 21530 RSVd: 4.86205437385
Document ID: 3546 RSVd: 4.58830699081
Document ID: 4235 RSVd: 4.58574834194"**
    
    ***Results:***
    
    
<img alt="Scenario 01 Case03" src="/assets/project_imgs/probabilistic_search_engine/multiple_keywords_query_and_case01.png" width="50%" height="50%">


### 3.2 Scenario 02 - Test Queries in the Project

1. **Case 1 - Democrats’ welfare and healthcare reform policies**

    ***Steps:***
    
    1. Open a termianl and go to the project directory.
    2. Run the **QueryProcessor.py** file with the memory size argument. **python QueryProcessor.py**
    3. Enter the keywords **Democrats’ welfare and healthcare reform policies**
    4. Check the result.
    
    ***Hypothesis & Analysis:***
    
    The console shows **"Document ID: 18731 RSVd: 4.98325477421
Document ID: 20449 RSVd: 4.49406152388
Document ID: 20023 RSVd: 4.32147517301
Document ID: 18161 RSVd: 4.2352933021
Document ID: 19134 RSVd: 4.21012046909
Document ID: 7401 RSVd: 4.19064247081
Document ID: 7433 RSVd: 4.15733703164
Document ID: 18683 RSVd: 3.99380512109
Document ID: 4268 RSVd: 3.93476195994
Document ID: 4006 RSVd: 3.84699849479
Document ID: 11204 RSVd: 3.8462341037
Document ID: 17940 RSVd: 3.82003719958
Document ID: 18469 RSVd: 3.65612408837
Document ID: 9248 RSVd: 3.6381502828
Document ID: 9096 RSVd: 3.5315440187
Document ID: 8139 RSVd: 3.43537385346
Document ID: 8307 RSVd: 3.43537385346
Document ID: 951 RSVd: 3.40575146814
Document ID: 6940 RSVd: 3.3418908256
Document ID: 7019 RSVd: 3.27046865523"**
    
    ***My Result:***

    
<img alt="Test Queries in the Project Case01" src="/assets/project_imgs/probabilistic_search_engine/test_queries_from_ta_case01.png" width="35%" height="50%">

2. **Case 2 - Drug company bankruptcies**

    ***Steps:***
    
    1. Open a termianl and go to the project directory.
    2. Run the **QueryProcessor.py** file with the memory size argument. **python QueryProcessor.py**
    3. Enter the keywords **Drug company bankruptcies**
    4. Check the result.
    
    ***Hypothesis & Analysis:***
    
    The console shows **"Document ID: 4050 RSVd: 9.53754573569
Document ID: 16771 RSVd: 7.92054563718
Document ID: 8209 RSVd: 6.53326444025
Document ID: 16994 RSVd: 6.13805385759
Document ID: 7125 RSVd: 5.55729196867
Document ID: 6089 RSVd: 5.01804901894
Document ID: 11434 RSVd: 4.70332817454
Document ID: 6544 RSVd: 4.58168664107
Document ID: 1805 RSVd: 4.24169669188
Document ID: 21239 RSVd: 4.08054405021
Document ID: 3577 RSVd: 3.97974378547
Document ID: 3328 RSVd: 3.9634259294
Document ID: 1391 RSVd: 3.94166965107
Document ID: 19640 RSVd: 3.86826141176
Document ID: 9542 RSVd: 3.86191227835
Document ID: 2679 RSVd: 3.81911580189
Document ID: 10026 RSVd: 3.70516180368
Document ID: 17604 RSVd: 3.67697375953
Document ID: 21251 RSVd: 3.6477876276
Document ID: 18716 RSVd: 3.62361602612"**
    
    ***My Result:***

    
<img alt="Test Queries in the Project Case02" src="/assets/project_imgs/probabilistic_search_engine/test_queries_from_ta_case02.png" width="35%" height="50%">

3. **Case 3 - George Bush**

    ***Steps:***
    
    1. Open a termianl and go to the project directory.
    2. Run the **QueryProcessor.py** file with the memory size argument. **python QueryProcessor.py**
    3. Enter the keywords **George Bush**
    4. Check the result.
    
    ***Hypothesis & Analysis:***
    
    The console shows **"Document ID: 20891 RSVd: 6.7415735697
Document ID: 8593 RSVd: 6.04271446855
Document ID: 4008 RSVd: 4.43298306691
Document ID: 20719 RSVd: 4.33954840096
Document ID: 20860 RSVd: 4.10518997796
Document ID: 16824 RSVd: 3.92099423062
Document ID: 7525 RSVd: 3.76855636171
Document ID: 2711 RSVd: 3.47062066523
Document ID: 16780 RSVd: 3.24262168624
Document ID: 4853 RSVd: 2.85617002809
Document ID: 2766 RSVd: 2.84335570593
Document ID: 10400 RSVd: 2.8180689457
Document ID: 6564 RSVd: 2.80559348062
Document ID: 5459 RSVd: 2.69809439266
Document ID: 2733 RSVd: 2.64185805748
Document ID: 15284 RSVd: 2.60922760572
Document ID: 16115 RSVd: 2.59852917817
Document ID: 10682 RSVd: 2.59852917817
Document ID: 5334 RSVd: 2.58791812436
Document ID: 21393 RSVd: 2.58791812436"**
    
    ***My Result:***
    
    
<img alt="Test Queries in the Project Case03" src="/assets/project_imgs/probabilistic_search_engine/test_queries_from_ta_case03.png" width="35%" height="50%">


## 4. Conclusion

As we can seen from above, BM25 improves upon TF*IDF.And it has its roots in probabilistic information retrieval. Probabilistic information retrieval is a fascinating field unto itself. Basically, it casts relevance as a probability problem. A relevance score, according to probabilistic information retrieval, ought to reflect the probability a user will consider the result relevant.

As far as I can see, the **BM25** algorithm has one advantage: 

1. BM25 is popular because of its efficiency. It can be seen as the state-of-the-art TF-IDF-like retrieval model.  It performs very well in many ad-hoc retrieval tasks

It also has one big disadvantage:

1. It is full of hacks/heuristics. This fact makes it hard to extend this framework.

### 4.1 What I've Learned

I’ve deepened my understanding of probabilistic search methodology, BM25 algorithm and developed a greater appreciation for the technology and techniques behind probabilistic search. I’ve also improved my rusty Python skills and used them to find fast ways to calculate things like the intersection of two postings lists, list comprehensions, etc.

<p class="text-center">
{% include elements/button.html link="https://github.com/Lei-Xu/Probabilistic-Search-Engine" text="Learn More" %}
</p>