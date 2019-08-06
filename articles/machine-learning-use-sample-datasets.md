<properties 
	pageTitle="Use the sample datasets in Machine Learning Studio | Azure" 
	description="Descriptions of the sample datasets that are included in Azure Machine Learning Studio" 
	services="machine-learning" 
	documentationCenter="" 
	authors="garyericson" 
	manager="paulettm" 
	editor="cgronlun"/>

<tags 
	ms.service="machine-learning" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="04/22/2015" 
	ms.author="garye"/>


# Use the sample datasets in Azure Machine Learning Studio

[top]: #machine-learning-sample-datasets

When you create a new workspace in Azure Machine Learning, a number of sample experiments and datasets are included by default.
Many of these sample datasets are used by the sample models in the [Azure Machine Learning Model Gallery](http://azure.microsoft.com/documentation/services/machine-learning/models/), and others are included as examples of various types of data used in machine learning. 

Some of these datasets are available in Azure BLOB storage. For these datasets the tables below provide a direct link.
You can use these datasets in your experiments by using the [Reader][reader] module - to see an example of how to access a dataset, view the properties of the [Reader][reader] module in the sample experiment associated with the dataset.

The rest of these sample datasets are listed under **Saved Datasets** in the module palette to the left of the experiment canvas when you open or create a new experiment in ML Studio.
You can use any of these datasets in your own experiment by dragging it to your experiment canvas.


<!--
For a list of sample experiments available in ML Studio, see [Machine Learning Sample Experiments][sample-experiments].

[sample-experiments]: machine-learning-sample-experiments.md
-->

[AZURE.INCLUDE [machine-learning-free-trial](../includes/machine-learning-free-trial.md)]

## Datasets associated with sample models

The following datasets are used by sample models in the [Azure Machine Learning Model Gallery](http://azure.microsoft.com/documentation/services/machine-learning/models/). 
For more information about the model and its associated experiment, follow the link in the table to see the model details.

<table>

<tr>
  <th align=left>Sample model</th>
  <th align=left>Associated experiment</th>
  <th align=left>Dataset name</th>
  <th align=left>Dataset description</th>
</tr>

<tr ID=bill-gates-rgb-image>
  <td valign=top><a href="http://azure.microsoft.com/documentation/articles/machine-learning-sample-color-quantization-using-k-means-clustering/">Color quantization using K-Means clustering</a></td>
  <td valign=top>Sample Experiment - Color Based Image Compression using K-Means Clustering - Development</td>
  <td valign=top>Bill Gates RGB Image</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-bill-gates-rgb-image](../includes/machine-learning-sample-dataset-bill-gates-rgb-image.md)]
  </td>
</tr>

<tr ID=german-credit-card-uci-dataset>
  <td valign=top><a href="http://azure.microsoft.com/documentation/articles/machine-learning-sample-credit-risk-prediction/">Credit risk prediction</a></td>
  <td valign=top>Sample Experiment - German Credit - Development</td>
  <td valign=top>German Credit Card UCI dataset</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-german-credit-card-uci-dataset](../includes/machine-learning-sample-dataset-german-credit-card-uci-dataset.md)]
  </td>
</tr>

<tr ID=crm-dataset-shared>
  <td rowspan=4 valign=top><a href="http://azure.microsoft.com/documentation/articles/machine-learning-sample-crm-task/">CRM task</a></td>
  <td rowspan=4 valign=top>Sample Experiments - CRM - Development</td>
  <td valign=top>CRM Dataset Shared</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-crm-dataset-shared](../includes/machine-learning-sample-dataset-crm-dataset-shared.md)]
  </td>
</tr>
<tr ID=crm-appetency-labels-shared>
  <td valign=top>CRM Appetency Labels Shared</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-crm-appetency-labels-shared](../includes/machine-learning-sample-dataset-crm-appetency-labels-shared.md)]
  </td>
</tr>
<tr ID=crm-churn-labels-shared>
  <td valign=top>CRM Churn Labels Shared</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-crm-churn-labels-shared](../includes/machine-learning-sample-dataset-crm-churn-labels-shared.md)]
  </td>
</tr>
<tr ID=crm-upselling-labels-shared>
  <td valign=top>CRM Upselling Labels Shared</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-crm-upselling-labels-shared](../includes/machine-learning-sample-dataset-crm-upselling-labels-shared.md)]
  </td>
</tr>

<tr ID=wikipedia-sp-500-dataset>
  <td valign=top><a href="http://azure.microsoft.com/documentation/articles/machine-learning-sample-finding-similar-companies/">Finding similar companies</a></td>
  <td valign=top>Sample Experiment - S & P 500 Company Clustering - Development</td>
  <td valign=top>Wikipedia SP 500 Dataset</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-wikipedia-sp-500](../includes/machine-learning-sample-dataset-wikipedia-sp-500.md)]
  </td>
</tr>

<tr ID=airport-codes-dataset>
  <td rowspan=3 valign=top><a href="http://azure.microsoft.com/documentation/articles/machine-learning-sample-flight-delay-prediction/">Flight delay prediction</a></td>
  <td rowspan=3 valign=top>Sample Experiment - Flight Delay Prediction - Development</td>
  <td valign=top>Airport Codes Dataset</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-airport-codes](../includes/machine-learning-sample-dataset-airport-codes.md)]
  </td>
</tr>

<tr ID=flight-delays-data>
  <td valign=top>Flight Delays Data</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-flight-delays-data](../includes/machine-learning-sample-dataset-flight-delays-data.md)]
</td>
</tr>

<tr ID=weather-dataset>
  <td valign=top>Weather Dataset</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-weather-dataset](../includes/machine-learning-sample-dataset-weather-dataset.md)]
  </td>
</tr>

<tr ID=intrusion-detection>
  <td valign=top> <a href="http://azure.microsoft.com/documentation/articles/machine-learning-sample-network-intrusion-detection/">Network intrusion detection</a></td>
  <td valign=top>Sample Experiment - Network Intrusion Detection - Development</td>
  <td valign=top><a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/network_intrusion_detection.csv">network_intrusion_detection.csv</a><br></td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-network-intrusion](../includes/machine-learning-sample-dataset-network-intrusion.md)]
  </td>
</tr>

<tr ID=bike-rental-uci-dataset>
  <td valign=top><a href="http://azure.microsoft.com/documentation/articles/machine-learning-sample-prediction-of-number-of-bike-rentals/">Prediction of the number of bike rentals</a></td>
  <td valign=top>Sample Experiment - Demand Forecasting of Bikes</td>
  <td valign=top>Bike Rental UCI dataset</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-bike-rental-uci-dataset](../includes/machine-learning-sample-dataset-bike-rental-uci-dataset.md)]
  </td>
</tr>

<tr ID=student-performance>
  <td valign=top><a href="http://azure.microsoft.com/documentation/articles/machine-learning-sample-prediction-of-student-performance/">Prediction of student performance</a></td>
  <td valign=top>Sample Experiment - Student Performance - Development</td>
  <td valign=top><a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/student_performance.txt">student_performance.txt</a></td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-student-performance](../includes/machine-learning-sample-dataset-student-performance.md)]
  </td>
</tr>

<tr ID=book-reviews-from-amazon>
  <td valign=top><a href="http://azure.microsoft.com/documentation/articles/machine-learning-sample-sentiment-analysis/">Sentiment analysis</a></td>
  <td valign=top>Sample Experiment - Sentiment Classification - Development</td>
  <td valign=top>Book Reviews from Amazon</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-book-reviews-from-amazon](../includes/machine-learning-sample-dataset-book-reviews-from-amazon.md)]
  </td>
</tr>

</table>



## Datasets used in sample experiments

The following datasets are used by sample experiments that are included in Azure ML Studio but are not used by models in the [Azure Machine Learning Model Gallery](http://azure.microsoft.com/documentation/services/machine-learning/models/). 

You can find these datasets listed under **Saved Datasets** in the module palette to the left of the experiment canvas when you open or create a new experiment in ML Studio.
You can use any of these datasets in your own experiment by dragging it to your experiment canvas.

You can find all of the sample experiments listed here in the **Samples** tab of the **Experiments** page in ML Studio.
To make a copy of a sample experiment that you can edit, open the experiment in ML Studio and click **Save As**.


<table>

<tr>
  <th align=left>Dataset name</th>
  <th align=left>Sample experiment</th>
  <th align=left>Dataset description</th>
</tr>

<tr ID=breast-cancer-features>
  <td valign=top>Breast Cancer Features
  <td valign=top>Sample Experiment - Breast Cancer - Development</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-breast-cancer-features](../includes/machine-learning-sample-dataset-breast-cancer-features.md)]
  </td>
</tr>

<tr ID=breast-cancer-info>
  <td valign=top>Breast Cancer Info</td>
  <td valign=top><i>same</i></td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-breast-cancer-info](../includes/machine-learning-sample-dataset-breast-cancer-info.md)]
  </td>
</tr>

<tr ID=direct-marketing>
  <td valign=top><a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/direct_marketing.csv">direct_marketing.csv</a></td>
  <td valign=top>Sample Experiment - Direct Marketing - Development - Training</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-direct-marketing](../includes/machine-learning-sample-dataset-direct-marketing.md)]
  </td>
</tr>

<tr ID=imdb-movie-titles>
  <td valign=top>IMDB Movie Titles</td>
  <td valign=top>Sample Experiment - Movie Recommender - Development</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-imdb-movie-titles](../includes/machine-learning-sample-dataset-imdb-movie-titles.md)]
  </td>
</tr>

<tr ID=movie-tweets>
  <td valign=top>Movie Tweets</td>
  <td valign=top>Sample Experiment - Ranking of Movie Tweets According to their Future Popularity - Development</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-movie-tweets](../includes/machine-learning-sample-dataset-movie-tweets.md)]
  </td>
</tr>

<tr ID=lyrl2004-tokens-test>
  <td valign=top><a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/lyrl2004_tokens_test.csv">lyrl2004_tokens_test.csv</a></td>
  <td valign=top>Sample Experiment - News Categorization - Development</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-lyrl2004-tokens-test](../includes/machine-learning-sample-dataset-lyrl2004-tokens-test.md)]
  </td>
</tr>

<tr ID=lyrl2004-tokens-train>
  <td valign=top><a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/lyrl2004_tokens_train.csv">lyrl2004_tokens_train.csv</a></td>
  <td valign=top><i>same</i></td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-lyrl2004-tokens-train](../includes/machine-learning-sample-dataset-lyrl2004-tokens-train.md)]
  </td>
</tr>

<tr ID=rcv1-v2-topics-qrels>
  <td valign=top><a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/rcv1-v2.topics.qrels.csv">rcv1-v2.topics.qrels.csv</a></td>
  <td valign=top><i>same</i></td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-rcv1-v2-topics](../includes/machine-learning-sample-dataset-rcv1-v2-topics.md)]
  </td>
</tr>

</table>



## Miscellaneous sample datasets

The following datasets are included in ML Studio to provide examples of various types of data used in machine learning.
You can find them listed under **Saved Datasets** in the module palette to the left of the experiment canvas when you open or create a new experiment in ML Studio.
You can use any of these datasets in your own experiment by dragging it to your experiment canvas.

<table>
<tr>
  <th align=left>Dataset name</th>
  <th align=left>Description</th>
</tr>

<tr>
  <td valign=top>Adult Census Income Binary Classification dataset</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-adult-census-income](../includes/machine-learning-sample-dataset-adult-census-income.md)]
  </td>
</tr>

<tr>
  <td valign=top>Automobile price data (Raw)</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-automobile-price](../includes/machine-learning-sample-dataset-automobile-price.md)]
  </td>
</tr>

<tr>
  <td valign=top>Blood donation data</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-blood-donation](../includes/machine-learning-sample-dataset-blood-donation.md)]
  </td>
</tr>

<tr>
  <td valign=top>Breast cancer data</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-breast-cancer-data](../includes/machine-learning-sample-dataset-breast-cancer-data.md)]
  </td>
</tr>

<tr>
  <td valign=top>Energy Efficiency Regression data</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-energy-efficiency-regression](../includes/machine-learning-sample-dataset-energy-efficiency-regression.md)]
  </td>
</tr>

<tr>
  <td valign=top>Flight on-time performance (Raw)</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-flight-on-time-performance](../includes/machine-learning-sample-dataset-flight-on-time-performance.md)]
  </td>
</tr>

<tr>
  <td valign=top>Forest fires data</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-forest-fires](../includes/machine-learning-sample-dataset-forest-fires.md)]
  </td>
</tr>

<tr>
  <td valign=top>Iris two class data</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-iris-two-class](../includes/machine-learning-sample-dataset-iris-two-class.md)]
  </td>
</tr>

<tr>
  <td valign=top>MPG data for various automobiles</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-automobile-mpg](../includes/machine-learning-sample-dataset-automobile-mpg.md)]
  </td>
</tr>

<tr>
  <td valign=top>Pima Indians Diabetes Binary Classification dataset</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-pima-indians-diabetes](../includes/machine-learning-sample-dataset-pima-indians-diabetes.md)]
  </td>
</tr>

<tr>
  <td valign=top>Restaurant customer data</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-restaurant-customer-data](../includes/machine-learning-sample-dataset-restaurant-customer-data.md)]
  </td>
</tr>

<tr>
  <td valign=top>Restaurant feature data</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-restaurant-feature-data](../includes/machine-learning-sample-dataset-restaurant-feature-data.md)]
  </td>
</tr>

<tr>
  <td valign=top>Restaurant ratings</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-restaurant-ratings](../includes/machine-learning-sample-dataset-restaurant-ratings.md)]
  </td>
</tr>

<tr>
  <td valign=top>Steel Annealing multi-class dataset</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-steel-annealing](../includes/machine-learning-sample-dataset-steel-annealing.md)]
  </td>
</tr>

<tr>
  <td valign=top>Telescope data</td>
  <td valign=top>
[AZURE.INCLUDE [machine-learning-sample-dataset-telescope-data](../includes/machine-learning-sample-dataset-telescope-data.md)]
  </td>
</tr>


</table>


<!-- Module References -->
[reader]: https://msdn.microsoft.com/library/azure/4e1b0fe6-aded-4b3f-a36f-39b8862b9004/
