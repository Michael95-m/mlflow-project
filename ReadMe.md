# Duration Prediction for NYC Taxi 

The purpose of this project is to practise MLops Tools taught in DataTalks Mlops zoomcamp.

## Data

I am using the dataset from the NYC Taxi and Limousine Commission (TLC).
We are going to predict the duration of the ride for the green taxi. 

### Download the dataset
The dataset can be downloaded using download_green_taxi_data in preprocess.py.

download_url function from fastdownload lib is used to download the dataset.

```
year = 2022
month = 2
dest = Path("data/filename")

download_green_taxi_data(year, month, dest)

```

## Train

train.py is used for training the model.

```
# Train Lasso model using scikit-learn lib
python train.py lasso 

# Train xgboost model using Xgboost lib
python train.py xgboost

# Train xgboost model after performing hypyerparameter tuning using hyperopt lib
python train.py xgboost_hopt

# Train both Lasso model and xgboost model with hyperparameter tuning
python train.py all
```

## Experiment Tracking using MLflow
For experiment tracking purpose, I am using mlflow. The very purpose of this porject is to practise MLflow for experiment tracking. 

Import MLflow model first.
```
import mlflow
```
In Mlflow, tracking data and artifacts can be stored 
    - locally in **mlruns** folder
    - using a SQLAlchemy-compatible database or 
    - using a tracking server

In this project, I am tracking the experiment using a local server and sqlite database.

After that, we can set the name of the experiment. MLflow will  create a new experiment if the experiment with this name doesn't exist.
If we don't set a name for experiment, mlflow will store data under default experiment.

```
mlflow.set_tracking_uri("http://127.0.0.1:5000")
mlflow.set_experiment(experiment-name)
```

### Tracking runs using MLflow
We can track specific data for each using mlflow.start_run(). We can set tags and log metric, param, artifacts and models.

```
def objective(params: dict) -> dict:

    # feature processing
    ...


    with mlflow.start_run():

        
        mlflow.set_tag("Developer", "Shane")
        mlflow.set_tag("model", "xgboost")

        mlflow.log_params(params)
        
        booster = xgb.train(
            params=params,
            dtrain=train,
            num_boost_round=50,
            evals=[(valid, 'validation')],
            early_stopping_rounds=50
        )

        y_pred = booster.predict(valid)
        rmse = mean_squared_error(y_val, y_pred, squared=False)


        mlflow.log_metric("rmse", rmse)
        mlflow.xgboost.log_model()

        signature = infer_signature(X_test, y_preds)
        mlflow.xgboost.log_model(booster, artifact_path="model", signature=signature)
```

We can also use auto logging for tracking. 
MLflow supports autolog for various machine learning and deep learning libs.

In order to use auto log for sklearn model, we can use **mlflow.sklearn.autolog()**.

```
def train_linear_sklearn(model : linear_model ,X_train : scipy.sparse._csr.csr_matrix, 
                         y_train : scipy.sparse._csr.csr_matrix, 
                         X_val : scipy.sparse._csr.csr_matrix, 
                         y_val : scipy.sparse._csr.csr_matrix) -> linear_model:
    
    mlflow.sklearn.autolog()

    with mlflow.start_run() as run:

        mlflow.set_tag("Developer", "Shane")

        print("Training Model")

        lr = model()

        lr.fit(X_train, y_train)

        print("Model Training Done")

        y_pred = lr.predict(X_val)

        rmse = mean_squared_error(y_val, y_pred, squared=False)

        print(f"RMSE : {rmse}")

        mlflow.log_metric("rmse", rmse)
        mlflow.log_artifact("models/preprocessor.b", artifact_path="artifacts")


    return lr
```

## MLflow UI

If we are tracking experiments locally, we can use **mlflow ui** to access the dashboard.

```
#bash

# locally
mlflow ui 

# locally with sqlite database
mlflow ui --backend-store-uri=sqlite:///mlflow.db

# using mlflow server
mlflow server --backend-store-uri=sqlite:///mlflow.db

```

## References

- datatalks mlops zoomcamp mlflow chapter - https://github.com/DataTalksClub/mlops-zoomcamp/tree/main/02-experiment-tracking
- mlflow docs - https://mlflow.org/docs/latest/index.html
- TLC Trip Record Data - https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page