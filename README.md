## 1.1 Vectorizer

Vectorization of text data is an essential task in Machine Learning applications because computers only understand numbers. We’ll use a pre-trained distilbert sentence transformer model to generate embeddings from an input sentence. The model returns a 768 dimensional dense vector for a sentence.

### 1.1.1 The Embedding API

We have built an API to return a 500 dimensional dense vector for a sentence.

During the time of API development, we couldn’t find a model that generates embedding size of 500, so we’ve taken a model that generates embedding size of 768 and we strip aways its 268 dimensions to return a 500 size embedding.

#### 1.1.1.1 Local Deployment

- Clone the repo
- We’ve used docker-compose for local deployment on a Windows machine with Docker Desktop installed.
- Navigate to the `/api` directory and run `docker compose up`. This will first build the docker image and start the local environment.

#### 1.1.1.2 Using the API

The API documentation is provided by swagger. It is accessible at [http://localhost:8001/docs#/](http://localhost:8001/docs#/).

### 1.1.1.2 Application Structure 

The project root contains two sub-directories:

- `api`
- `notebooks`

The `notebooks` directory contains our experiments. Even before we built our API, we tested the sentence transformer in the notebook environment. The code we used in the notebook environment, was later used to build the API.

The `api` directory contains two sub-directories:

- `app`
- `embedding_model`

The `embedding_model` directory contains `core.py` for parsing the `config.yml` file, which contains settings like embedding size. We’re using `strictyaml` package to parse in a type-safe manner.

Later we use this setting in the `embedding.py` file to strip away dimensions and return a 500 size vector.

`validation.py` is responsible for data validation of input data, we use pydantic models for data validation.

`embedding.py` is responsible for loading the sentence transformer model to generate the embedding.

Even though, It’s an overkill to have such an elaborate setup for config management for just an embedding task like ours, but having such a setup for configuration management scales well in production, when dealing with several hundreds of features in a model training pipeline, by allowing to use the same config for Training as well as Predictions.

The `app` directory contains two sub-directories:

- `schemas`
- `tests`

Under `schemas` we define the Input and Output data Models in `vectorize.py` using Pydantic, to validate if we’re accepting the input data and returning the output data in the correct format.

Under `tests` we define our test in `test_api.py` using Pytest, to validate if we’re returning the embeddings of size 500, without any errors.

We’re using `tox` to invoke pytest. Using `tox` allows us to have a separate `test_requirements.txt` file without polluting the `requirements.txt` file used by the application.

We’re also using `tox` to format our code using `black`.

The `app` directory contains following files:

- `api.py`
- `config.py`
- `main.py`

`main.py` is the entrypoint of our application. Here, we setup logging, routing and CORS configurations.

`config.py` is the module responsible for handling logging and providing CORS origins.

### 1.1.1.3 Future Enhancements 

#### API

By stripping down 268 dimensions, there is information loss, also we’re wasting computation to first calculate a vector of size 768, only to strip away 268 of its dimensions. A possible quick win is to find another transformer model that generates embeddings close to 500 or fine-tune the model.

#### Deployment

We can use kubernetes services provided by Docker Desktop to perform the deployment into a local kubernetes cluster with one master node and one worker node. Also this’ll make our local environment virtually identical to a production deployment in a cloud service like AWS EKS if we want to go for a managed solution or use vanilla EC2 instances if we want to manage the master and worker nodes ourselves. We can also setup Auto Scaling group to optimize for computing using EC2 instances.
For lighter workloads such as generating embeddings, we can use AWS Lambda.

#### Model Versioning

Currently we load it during runtime when we launch the application. We can store different versions of the model using AWS S3 and DVC. To load the model on demand.

#### Model Monitoring

We can deploy ELK stack and store the input and embeddings in Elasticsearch, so we can later pull data for analysis. Over time when we get distribution shifts for features, i.e, embeddings, it can lead to various problems like covariate shift, label shift and concept shift, for the downstream tasks. To detect these shifts, we can generate embeddings using our test datasets for current vs previous models, then we can run Kolmogorov–Smirnov test to detect distribution shifts beyond a significance. We can setup notifications via email/slack when distribution shift occurs beyond a significance level.

#### Continuous Integration

We can run tests for pull-requests, build the docker image and push to AWS ECR

#### Differential Testing

We can do differential Testing as part of Continuous Integration, by comparing the results of embeddings generated by the current model and the previous model on test data using Konglomorov-Smirnov test with significance level of 0.05

#### Security

Authentication and Authorization using JWT. We can create a microservice to authenticate the user and issue an access token and a refresh token. Use the access token to authorize the API request. If the access token expires, allow to generate a new access token using the refresh token. If the refresh token expires, redirect to login.
