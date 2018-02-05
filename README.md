# Tracking NIH Grant
Tracking NIH grant from proposal to publication

## Table of Content:
1)	Object goals - Tracking NIH grants from proposals to publications.
2)	Getting the data - Mapping publications to grants.
3)	Exploratory data analysis – some insights.
4)	More deep insights – Are researchers honest about their grants?
5)	Make predictions – Whether a grant will be renewed?
6)	Conclusions and discussions.

## 1.	Object Goals - Tracking NIH grants from proposals to publications.
The national institute of health (NIH) is the primary funding resources for biomedical and public health research in the U.S. The grants funded by the NIH is publicly available. Publications are one of the important product of NIH research grants, and authors are required to cite their NIH support in their publications. Mapping publications to grants is critical component to accurately measure impact. For example, by analyzing funding and publication data, I estimated the time and cost of publications in certain filed, and even identified the publications which did not produce what they promised. At last, I compared several binary classifiers to predict whether a grant will be renewed. Similar strategy can be developed to improve the NIH funding allocation.

## 2.	Getting the data - Mapping publications to grants.
NIH grants information is publicly accessible from [RePORTER](https://exporter.nih.gov/ExPORTER_Catalog.aspx?sid=2&index=0) system. The related information such as grant publications and abstracts are provided. However, detailed publication information such as abstract is lacking, which can be found in NIH database - [Pubmed](https://www.ncbi.nlm.nih.gov/pubmed/). Thus, I used a public API to map publications to grants. More specifically, the [_Entrez_](http://biopython.org/DIST/docs/api/Bio.Entrez-module.html) package from Biopython was used to search publications linked to each Grant-ID. Pulication information such as PMID, PubDate, PubTypeList, FullJournalName and Abstract were extracted. All the information except for abstract were combined into one [csv](https://github.com/lilyvalley/Tracking-NIH-Grants/blob/master/file/FY2010_merge_all%20copy.csv) file. Grant and publication abstracts were extracted into a separate [csv](https://github.com/lilyvalley/Tracking-NIH-Grants/blob/master/file/Abstract.csv) file. Due to the time-consuming process of fetching data from NCBI, only 2010 grant data was used in the current study. Ipython notebook used to scraping publication data can be found [here](https://github.com/lilyvalley/Tracking-NIHGrants/blob/master/ipython_notebook/Mapping%20grant%20to%20publication.ipynb).

## 3.	Exploratory data analysis – some insights.
Based on the time difference between grant approval and publication, we can estimate how long it takes to publish a paper. The number of total publications linked to one specific grant tells us the average cost to produce a paper for that grant. Publication cost and speed differ among NIH institutes, this reflects the funding resources and publication difficulty are vary in research subfileds. To achieve more fair comparison, I only selected the newly funded grant which "Application_type" is 1 in all subsequent analysis. To reduce noise, I only focused on the grants which have at least one publication records. Some interesting insights from studying publications and grants in 2010 are listed below:

  * We are curious, how much it cost to publish? The total number of new funded grants in 2010 which have at least one publication is 4110. The total cost of these grants is 1.85 billion U.S dollars. There are 20415 publications cited these grants, thus, the average cost of publications from new grant awarded in 2010 is about 90000 U.S dollars. 

  * Similarly, we want to know how long it takes to publish? I created new feature such as “time to publish” by calculating time difference between grant approval year and publication year. The average time to publish a paper in 2010 new funded grant is 3.4 years, without considering those grants never published. This is pretty impressive.

  * Are cost and publication speed differ among subfields? “Administering_IC” is a two letter code to designate the NIH agency. The NIH institute it represents can be found [here](https://grants.nih.gov/grants/acronym_list.htm). Studying NIH institutes could reflect the publications in different subfield. For example, the most funded institutes are related to public health such as HM (NCHM), PS (NCHHSTP) and TP (OPHPR). The most productive publication institutes are from research fields of cancer, heart disease and diabetes. Top three are CA (NCI), HL (NHLBI) and DK (NIDDK). While productive does not necessary means fast. Instead, the fast publication institutes are human genome research insitute HG (NHGRI), bioengineering institute EB (NIBIB) and infectious disease institute (NIAID). Genomics and bioengineering are fast evolving fields, and fighting infectious disease are more immediate needs. Figures can be found in the ipython notebook [here](https://github.com/lilyvalley/Tracking-NIH-Grants/blob/master/ipython_notebook/Exploratory%20Data%20Analysis.ipynb).
  
  * Interestingly, a bubble plot of total cost, average cost and publication numbers separates NIH institutes into distinct clusters. Institute RR (research resources, NCRR) has high award and high publication amount, thus the average cost of publishing a paper is low. Institute such as NR (Nursing research, NINR) have high award but does not publish much. General medicine institute is the most cost effective. While cancer research institute award more and publish more.
  
  ![Total cost vs average cost](https://github.com/lilyvalley/Tracking_NIH_Grant/blob/master/figure/scatter_merge.png)

## 4.	More deep insights – Are researchers honest about their grants?
It is import for the funding agent to follow up their awards to make sure the grants are used for the same purpose as researchers announced. This can be achieved by comparing the abstract of grants and publications. 

Term frequency-inverse document frequency (tf-idf) is a common method of assessing the importance of a word in a document corpus. It weights the frequency of a given term in an individual document with the inverse frequency of that term in the entire corpus. We first use tf-idf to transform both abstract then compare the cosine similarity of the two.

  ![Cosine similarity](https://github.com/lilyvalley/Tracking_NIH_Grant/blob/master/figure/cosine_plot.png)
  
In the scatter plot above, 50 abstracts are compared and ranked based on their cosine similarity score. The cosine scores range from 0.6 to 0.1. The higher the score, the more similar the abstracts between grants and publications.  Indeed, the shorter abstracts tend to have lower cosine scores. Some publications are linked to multiple grants. For example, the paper (PMID 23681158) cited five grants. When it compares to a grant for training purpose, the cosine score is very low (0.15). This is due to the research details are not described in the training grant. However, when it compares to a research grant, the cosine score is much higher (0.47). This indicate cosine score can accurately reflect abstract similarity. The contant of the abstracts is shown in the ipython notebook [here](https://github.com/lilyvalley/Tracking-NIH-Grants/blob/master/ipython_notebook/More%20deeper%20insights.ipynb).

## 5.	Make predictions – Whether a grant will be renewed?
The current mechanism of NIH award is based on peer review by experts in the fields, however this process is very expensive and time-consuming. A research even reported the NIH funding allocation seems no better than lottery. If we can make accurate prediction on whether a grant should be funded, we can partially solve the problem. However, NIH does not publish unfunded grants, thus we reframe the question, try to predict whether a grant will be renewed.

To predict whether a grant will be renewed, I first generated a table with labels by looking up the new grant from 2010, whether the grant ID is involved in later years (2011 to 2016). If the grant ID is found, then marked it as renewed (1), otherwise marked it as non-renewed (0). By doing so, I reframe the problem as a binary classification problem.

As an overview, the renewed grants are much more than non-renewed grants. This could be influenced by the sampling method, which only consider grants with publications. As shown in histogram figure bellow, some institute have more non-renewed grants than others, such as AI institute. Total funding year also influence renew rate, while short time grants (1 or 2 years) are more likely to be renewed than long time grants (more than 2 years) as shown in the line plots.

![histogram](https://github.com/lilyvalley/Tracking-NIH-Grants/blob/master/images/grant_renew_plot1.png)
![line plot](https://github.com/lilyvalley/Tracking-NIH-Grants/blob/master/images/grant_renew_plot2.png)


It’s time to prepare data for machine learning!

The main steps involved in data wrangling are listed below and all details can be found in the Ipython notebook [here](https://github.com/lilyvalley/Tracking-NIH-Grants/blob/master/ipython_notebook/Predict%20whether%20a%20grant%20will%20be%20renewed.ipynb).

  1)	Transform categorical attributes to numbers. Categorical attributes including NIH institutes information are transformed into numbers using one-hot encoding.  This method creates one binary attribute per category (per institute).
  2)	Feature scaling. It is important to perform feature scaling. With few exceptions, ML don't perform well when the input numerical attributes have very different scales. Use standard scalar in sklearn package to achieve feature scaling. 
  3)	Build the transformation pipeline to automate the process. Both numeric and categorical transformer are combined into a full pipeline.
  4)	Split data into train/test data sets to avoid data snooping bias.
  5)	Start with training a stochastic gradient descent (SGD) classifier.
  6)	Evaluate the model by using K fold cross-validation.
  7)	Compare and select performance measure such as accuracy (not very informative), confusion matrix/ precision and recall/ f1 score. Plot ROC curve to evaluate model.
  8)	Since the performance of the SGD not optimal, I trained another model using random forest. 
The ROC curve shows much better performance compared to SGD. See below:
  ![ROC comapre](https://github.com/lilyvalley/Tracking-NIH-Grants/blob/master/images/ROC_multiple.png)
  9)	Fine tune the random forest classifier by grid search to find best hyperparameter and best model.
  10)	Feature importance. Random forest is convenient to return feature importance. The heatmap shows the top ranked features. Indeed, the total cost and the grant year are most import to predict renew of a grant.
   ![feature importance](https://github.com/lilyvalley/Tracking-NIH-Grants/blob/master/images/feature_importance.png)
  11)	Use best random forest model to predict our test data set. The performance of test data is usually worse than the training data. The ROC curve and confusion matrix are shown. The precision score, recall score and f1 score are pretty high in this example, indicating the classifier can be used to make accurate prediction on grant renew. 
  ![ROC comapre](https://github.com/lilyvalley/Tracking-NIH-Grants/blob/master/images/ROC_test.png)


## 6.	Conclusions and discussions.

This is just a piece of ice burger of studying grants and publications. In the matrix used to train classifier, only a few columns are included. Feature engineering will be needed to dicover more features. As indicated in my analysis, NIH institutes which reflect research subfields, are important in predicting grant renew. Research topic can also be extracted from analyzing text data. With the help of NLP and Deep learning, more features can be extracted to make better predictions.  
