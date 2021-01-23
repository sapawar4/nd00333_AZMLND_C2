# Operationalizing Machine Learning
## Sandeep Pawar

The purpose of this project is to operationalize, i.e deploy & consume machine learning models, via Azure ML service. The machine learning model s are created by two different approaches, first by using AutoML using UI for a bank marketing dataset and second by using pipelines to create AutoML model. The best models, for these both datasets, are deployed in service with Azure Container Instance. Application Insights is enabled to mitor the performance of the model in service. REST endpoints are tested by deploying a local container with Swagger UI. These steps can be used to deploy any machine learning in service and monitor its performance. 

## Architectural Diagram
![architecture](https://raw.githubusercontent.com/sapawar4/nd00333_AZMLND_C2/master/starter_files/images/arch.JPG)
An architectural diagram is an image that helps visualize the flow of operations from start to finish. As can be seen in the architecture above, the datastore, dataset and compute are managed in the Azure ML workspace. AutoML models are created by training multiple different types of models and the best model based on chosen accuracy metric is deployed to service. The same or different compute cluster can be used to deploy the model for real-time consumption by any application. REST API is created for the application to consume the model.

## Key Steps
- ### Enable Authentication
	- Authentication is enabled to ensure the workspace artifacts (compute, dataset etc.) are accessible and continuous flow of operation is possible.

- ### Preprocess data
	- For the bank marketing data, exploratory data analysis showed that the `duration` column leaked information and should not be included in the model. 
	- Version 2 of the same dataset was created by dropping the `duration` column
	
![](https://raw.githubusercontent.com/sapawar4/nd00333_AZMLND_C2/master/starter_files/images/image1.png)
- ### AutoML Experiment
	- First a compute cluster is created which will be used by AutoML experiment to train multiple models in parallel. For this project, a `Standard_DS12_V2` compute cluster was used
	- The bank-marketing V2 created in the first step was selected & column `y` was selected as the target column 
	 ![](https://raw.githubusercontent.com/sapawar4/nd00333_AZMLND_C2/master/starter_files/images/image3.png)	 
	- Since this was a classification task, "Classification" option was selected and DS12v2 was selected as the compute cluster
	![](https://raw.githubusercontent.com/sapawar4/nd00333_AZMLND_C2/master/starter_files/images/image4.png)
	- This is an imbalanced dataset. Hence `AUC_weighted` was used as the primary metric. To mitigate overfitting, 5 fold k-fold-crossvalidation option was selected
	- ![](https://raw.githubusercontent.com/sapawar4/nd00333_AZMLND_C2/master/starter_files/images/image6.png)	- After 22 minutes of runtime , the AutoML experiment was over. 

	- ![](https://raw.githubusercontent.com/sapawar4/nd00333_AZMLND_C2/master/starter_files/images/image7.png)
	- The best model was a `VotingEnsemble` model (same as Project 1) which had an AUC = 80.5% and accuracy of 90%
![](https://raw.githubusercontent.com/sapawar4/nd00333_AZMLND_C2/master/starter_files/images/image9.png)
	- This model was deployed to service using Azure Container Instance (ACI). Note that `enable authentication` option was enabled.
	- ![](https://raw.githubusercontent.com/sapawar4/nd00333_AZMLND_C2/master/starter_files/images/image10.png)	- To monitor the performance of the model, "Application Insights" needs to be turned on. To do so, `logs.py` file was used to enable it programmatically. 
	- ![](https://raw.githubusercontent.com/sapawar4/nd00333_AZMLND_C2/master/starter_files/images/image13.png)
![](https://raw.githubusercontent.com/sapawar4/nd00333_AZMLND_C2/master/starter_files/images/image14.png)
![](https://raw.githubusercontent.com/sapawar4/nd00333_AZMLND_C2/master/starter_files/images/image15.png)
	- As seen below, this enables us to monitor various metrics related to performance of the model such as latency, exception errors etc. 
	- ![](https://raw.githubusercontent.com/sapawar4/nd00333_AZMLND_C2/master/starter_files/images/image16.png)
- ### Swagger Documentation
	- To consume the deployed model via REST API, we need to follow multiple steps 
	- Download the `swagger.json` for Azure ML Studio
	- ![](https://raw.githubusercontent.com/sapawar4/nd00333_AZMLND_C2/master/starter_files/images/image17.png)	- Use `swagger.sh` to spin up swagger docker
![](https://raw.githubusercontent.com/sapawar4/nd00333_AZMLND_C2/master/starter_files/images/image18.png)	- Swagger UI is avainle at `localhost:8000`
![](https://raw.githubusercontent.com/sapawar4/nd00333_AZMLND_C2/master/starter_files/images/image22.png)	- `serve.py` enables us to create a HTTP Server to host `swagger.json` on your localhost at one of the ports, which further becomes an input to Swagger UI.
![](https://raw.githubusercontent.com/sapawar4/nd00333_AZMLND_C2/master/starter_files/images/image26.png)![](https://raw.githubusercontent.com/sapawar4/nd00333_AZMLND_C2/master/starter_files/images/image28.png)![](https://raw.githubusercontent.com/sapawar4/nd00333_AZMLND_C2/master/starter_files/images/image27.png)![](https://raw.githubusercontent.com/sapawar4/nd00333_AZMLND_C2/master/starter_files/images/image29.png)
	- To consume the endpoint locally, `endpoint.py` is used. The `coring_uri` and `key` are updated by using Azure ML studio
![](https://raw.githubusercontent.com/sapawar4/nd00333_AZMLND_C2/master/starter_files/images/image31.png)
	- Sample input is sent to the model and response is obtained
	- ![](https://raw.githubusercontent.com/sapawar4/nd00333_AZMLND_C2/master/starter_files/images/image30.png)
- ### Using Pipelines to create, deploy and consume AutoML Model
In this step ``aml-pipelines-with-automated-machine-learning-step.ipynb`` is uploaded to Azure ML studio. Workspace is initiated and accessed by interactive authentication. Compute cluster created above is used to run the pipeline. This pipeline included accessing, cleaning the data and creating automl config. The bets model is obtained and deployed in service. The status of the pipeline runs can be monitored in the notebook or the AzureML studio under pipeline runs & experiments. 


![](https://raw.githubusercontent.com/sapawar4/nd00333_AZMLND_C2/master/starter_files/images/image12.png)

![](https://raw.githubusercontent.com/sapawar4/nd00333_AZMLND_C2/master/starter_files/images/image19.png)

![](https://raw.githubusercontent.com/sapawar4/nd00333_AZMLND_C2/master/starter_files/images/image20.png)


![](https://raw.githubusercontent.com/sapawar4/nd00333_AZMLND_C2/master/starter_files/images/image21.png)



![](https://raw.githubusercontent.com/sapawar4/nd00333_AZMLND_C2/master/starter_files/images/image23.png)
	- Below screenshot shows that the pipeline endpoint is active in service
![](https://raw.githubusercontent.com/sapawar4/nd00333_AZMLND_C2/master/starter_files/images/image25.png) 

## Screen Recording
The video link detailing the steps are here: https://youtu.be/imEgt8sUv2w

## Standout Suggestions
- Use different sampling methods such as oversampling, undersampling, SMOTE etc to see if they improve the performance
- Use "Explanation" feature to understand the model performance and identify important features
- Since this model will be used by a bank, model fairness study should be done to make sure model is not biased against any section of the population
- Use a different sampling method such as Monte Carlo sampling instead of cross-validation to see how the model performance will change.
- Run atoml longer. In this project, AutoML was run for 30 minutes, instead it can be run longer and Deep Learning can be used to see if it improves the performance.

