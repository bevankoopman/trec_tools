# TREC TOOLS

A simple toolkit to process TREC files. If you do not know what TREC is, you surely do not need this package.

## Installing

```
pip install trectools
```

## Background

The aim of this module is to facilitate typical procedures used when analysing data from a TREC/CLEF/NTCIR campaign.
The main object in TREC campaign is participant retrieval system. A retrieval system is takes as input some information need represented by a query and generates a list of documents that are relevant for that query. This information is represented in a TREC campaign as a participant run, which is a file with the following structure:  

qid Q0 docno rank score tag

where:  
- **qid**	is the query number
- **Q0**	is the literal Q0
- **docno**	is the id of a document returned for qid
- **rank**	(1-999) is the rank of this response for this qid
- **score**	is a system-dependent indication of the quality of the response
- **tag**	is the identifier for the system

Example:  
1 Q0 nhslo3844_12_012186 1 1.73315273652 mySystem  
1 Q0 nhslo1393_12_003292 2 1.72581054377 mySystem  
1 Q0 nhslo3844_12_002212 3 1.72522727817 mySystem  
1 Q0 nhslo3844_12_012182 4 1.72522727817 mySystem  
1 Q0 nhslo1393_12_003296 5 1.71374426875 mySystem  

Once a campaign ends, the evaluation phase starts.
Usually, it is impossible to judge every document retrieved by every participant run for every query.
There is a huge cost, both in terms of money and time, to make judgements.
Many strategies have been proposed to select which documents to judge.
Without going into many details, a pool of documents has to be created.
Once documents in that pool are judged with respect to a query, a file is created containing all these judgements.
This file is usually called 'qrel' and contains lines like this:

qid 0 docno relevance  

where:  
- **qid**	is the query number
- **0**	is the literal 0
- **docno**	is the id of a document in your collection
- **relevance**	is how relevant is docno for qid

Example:  
1	0	aldf.1864_12_000027	1  
1	0	aller1867_12_000032	2  
1	0	aller1868_12_000012	0  
1	0	aller1871_12_000640	1  
1	0	arthr0949_12_000945	0  
1	0	arthr0949_12_000974	1  

Finally, the information retrieval community uses some evaluation metric to quantify how good a participant system is.
Many of common metrics, such as precision@N, mean average precision, bpref and others, are implemented in a tool called [trec_eval] (http://trec.nist.gov/trec_eval/). Although trec_eval lacks many other important measures (e.g., nDCG or RBP), it provides a consistent format for system result:


label qid value

where:  
- **label**	is any string, usually representing a metric
- **qid**	is the query number or 'all' to represent a aggregate value
- **value**	is numeral result of a metric

Example:
num_rel_ret             7   77
map                     7   0.4653
P_10                    9   0.9000
num_rel_ret             all 1180 
map                     all 0.1323
gm_map                  all 0.0504

The three main modules found in this package are inspired by the main files created in a TREC campaign: a participant run, a qrel e a result file: TrecRun, TrecQrel, TrecRes. Also, there is a 'misc' module to implement many common operations involving one or more module (such as comparing statistical significance of different runs). See the section below for some examples.

### Code Examples

```python
> from trectools import TrecRun, TrecQrel, TrecRes, misc

> myRun = TrecRun("~/mysystem.run")
> myRun.topics()
{1,2,3,4,5,6,7}

> myRun.get_top_documents(topic=1,n=2)
['nhslo3844_12_012186', 'nhslo1393_12_003292']

> myQrel = TrecQrel("~/assessor.qrel")
> myQrel.describe()
count    2076.000000
mean        0.268786
std         0.575825
min         0.000000
25%         0.000000
50%         0.000000
75%         0.000000
max         2.000000
> myQrel.get_number_of(1)
278

> myQrel.get_number_of(2)
140

> myQrel.check_agreement(myQrel)
1.0

> myRes = myRun.evaluate_run(qrel)
> myRes.get_result(metric="P_10")
0.8700

> myRes.get_results_for_metric("P_10")
{1:0.9000, 2:0.8000, ...} 

> myRun2 = TrecRun("~/mysystem2.run")

> myRes2 = myRun2.evaluate_run(qrel)
> myRes.compare_with(myRes2, metric="map")
Ttest_indResult(statistic=1.2224721254608264, pvalue=0.22486892703278308)

> list_of_results = [myRes, myRes2]
> misc.sort_systems_by(list_of_results, "P_10")
[(0.8700, 'myRes1'), (0.8300, 'myRes2')]

> misc.get_correlation( misc.sort_systems_by(list_of_results, "P_10"), misc.sort_systems_by(list_of_results, "map") )
KendalltauResult(correlation=0.99999999999999989, pvalue=0.11718509694604401)

> misc.get_correlation( misc.sort_systems_by(list_of_results, "P_10"), misc.sort_systems_by(list_of_results, "map"), correlation="tauap" )
1.0

