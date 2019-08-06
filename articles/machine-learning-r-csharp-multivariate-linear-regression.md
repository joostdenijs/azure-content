<properties 
	pageTitle="Multivariate Linear Regression | Azure" 
	description="Multivariate Linear Regression" 
	services="machine-learning" 
	documentationCenter="" 
	authors="jaymathe" 
	manager="paulettm" 
	editor="cgronlun"/>

<tags 
	ms.service="machine-learning" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="04/21/2015" 
	ms.author="jaymathe"/> 


#Multivariate Linear Regression   
 

 
Suppose you have a dataset and would like to quickly predict a dependent variable (y) for each individual (i) based on independent variables. Linear regression is a popular statistical technique used for such predictions. Here the dependent variable y is assumed to be a continuous value.  


[AZURE.INCLUDE [machine-learning-free-trial](../includes/machine-learning-free-trial.md)]  

A simple scenario could be where the researcher is trying to predict the weight of an individual (y) based on their height (x). A more advanced scenario could be where the researcher has additional information for the individual (such as weight, gender, race) and attempts to predict the weight of the individual. This [web service]( https://datamarket.azure.com/dataset/aml_labs/multivariate_regression) fits the linear regression model to the data and outputs the predicted value (y) for each of the observations in the data.

>This web service could be consumed by users – potentially through a mobile app, through a website, or even on a local computer, for example. But the purpose of the web service is also to serve as an example of how Azure Machine Learning can be used to create web services on top of R code. With just a few lines of R code and clicks of a button within Azure Machine Learning Studio, an experiment can be created with R code and published as a web service. The web service can then be published to the Azure Marketplace and consumed by users and devices across the world with no infrastructure setup by the author of the web service.  

##Consumption of web service  
This web service gives the predicted values of the dependent variable based on the independent variables for all of the observations. The web service expects the end user to input data as a string where rows are separated by commas (,) and columns are separated by semicolons (;). The web service expects 1 row at a time and expects the first column to be the dependent variable. An example dataset could look like this:

![Sample data][1]

Observations without a dependent variable should be input as “NA” for y. The data input for the above dataset would be the following string: “10;5;2,18;1;6,6;5.3;2.1,7;5;5,22;3;4,12;2;1,NA;3;4”. The output is the predicted value for each of the rows based on the independent variables. 

>This service, as hosted on the Azure Marketplace, is an OData service; these may be called through POST or GET methods. 

There are multiple ways of consuming the service in an automated fashion (an example app is [here](http://microsoftazuremachinelearning.azurewebsites.net/MultipleLinearRegressionService.aspx )).

###Starting C# code for web service consumption:

	public class Input
	{
	        public string value;
	}
	
	public AuthenticationHeaderValue CreateBasicHeader(string username, string password)
	{
	        byte[] byteArray = System.Text.Encoding.UTF8.GetBytes(username + ":" + password);
	        return new AuthenticationHeaderValue("Basic", Convert.ToBase64String(byteArray));
	}
	
	void Main()
	{
	        var input = new Input() { value = TextBox1.Text };
	        var json = JsonConvert.SerializeObject(input);
	        var acitionUri = "PutAPIURLHere,e.g.https://api.datamarket.azure.com/..../v1/Score";
	        var httpClient = new HttpClient();
	
	        httpClient.DefaultRequestHeaders.Authorization = CreateBasicHeader("PutEmailAddressHere", "ChangeToAPIKey");
	        httpClient.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
	
	        var response = httpClient.PostAsync(acitionUri, new StringContent(json));
	        var result = response.Result.Content;
	    	var scoreResult = result.ReadAsStringAsync().Result;
	}




##Creation of web service  
>This web service was created using Azure Machine Learning. For a free trial, as well as introductory videos on creating experiments and [publishing web services](machine-learning-publish-a-machine-learning-web-service.md), please see [azure.com/ml](http://azure.com/ml). Below is a screenshot of the experiment that created the web service and example code for each of the modules within the experiment.


From within Azure Machine Learning, a new blank experiment was created and two [Execute R Script][execute-r-script] modules were pulled onto the workspace. This web service runs an Azure Machine Learning experiment with an underlying R script. There are 2 parts to this experiment: schema definition, and training model + scoring. The first module defines the expected structure of the input dataset, where the first variable is the dependent variable and the remaining variables are independent. The second module fits a generic linear regression model for the input data.  
  
![Experiment flow][3]

####Module 1:
 
####Schema definition  
	data <- data.frame(value = "1;2;3,4;5;6,7;8;9", stringsAsFactors=FALSE) maml.mapOutputPort("data");  

####Module 2:
####LM modeling   
	data <- maml.mapInputPort(1) # class: data.frame  
  
	data.split <- strsplit(data[1,1], ",")[[1]]  
	data.split <- sapply(data.split, strsplit, ";", simplify = TRUE)  
	data.split <- sapply(data.split, strsplit, ";", simplify = TRUE)  
	data.split <- as.data.frame(t(data.split)) 
	data.split <- data.matrix(data.split) 
	data.split <- data.frame(data.split) 
	model <- lm(data.split)  

	out=data.frame(predict(model,data.split))  
	out <- data.frame(t(out))

	maml.mapOutputPort("out");  
 
##Limitations
This is a very simple example of a multiple linear regression web service. As can be seen from the example code above, no error catching is implemented and the service assumes everything is a continuous variable (no categorical features allowed), as the service only inputs numeric values at the time of the creation of this web service. Also, the service currently handles limited data size, due to the request/response nature of the web service call and the fact that the model is being fit every time the web service is called. 

##FAQ
For frequently asked questions on consumption of the web service or publishing to the Azure Marketplace, see [here](machine-learning-marketplace-faq.md).

[1]: ./media/machine-learning-r-csharp-multivariate-linear-regression/multireg-img1.png
[2]: ./media/machine-learning-r-csharp-multivariate-linear-regression/multireg-img2.png
[3]: ./media/machine-learning-r-csharp-multivariate-linear-regression/multireg-img3.png


<!-- Module References -->
[execute-r-script]: https://msdn.microsoft.com/library/azure/30806023-392b-42e0-94d6-6b775a6e0fd5/
