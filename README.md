# Private Preview: AutoML Model Testing

> Welcome to the PRIVATE PREVIEW of **Azure Machine Learning Automated ML Model Testing**.
> The AutoML model testing feature provides support for automated model testing.
> This is an early PREVIEW, still not announced and not supported publicly so
> it's defined as PRIVATE from that point of view, although anyone can access to it.

Automated Machine Learning, also referred to as Automated ML or AutoML, is
the process of automating the time consuming, iterative tasks of machine
learning model development. It allows data scientists, analysts, and
developers to quickly and easily build ML models with high scale, efficiency,
and productivity all while sustaining model quality.

The AutoML model testing feature allows the user to test the models which are
generated by AutoML training runs. Testing a model takes as input a test dataset
and outputs predictions and metrics for those predictions.
There are currently two supported scenarios:

1. Test the best model which is created by the main AutoML run.
   The model testing run will be started automatically at the end of the main run
   after all models have been created and the best model has been identified.
2. Test any model after the main AutoML run has completed ("on-demand" model testing).
   These test runs are started manually by the user either using the SDK or the UI.

Model testing is supported for the following task types:

- Classification
- Regression
- Forecasting

There are three ways to start a test run from a script or a notebook.
See the included notebooks for examples and further details.

1. Providing an existing test dataset when creating the `AutoMLConfig`.

    ```python
    automl_config = AutoMLConfig(task='forecasting',
                                 ...
                                 # Provide an existing test dataset
                                 test_data=test_dataset,
                                 ...
                                 forecasting_parameters=forecasting_parameters)
    ```
    Supported input test dataset types: [Dataset](https://docs.microsoft.com/en-us/python/api/azureml-core/azureml.core.dataset.dataset?view=azure-ml-py).
2. Specifying a train/test percentage split when creating the `AutoMLConfig`.
   Note, this scenario is not supported for forecasting tasks.

   ```python
   automl_config = AutoMLConfig(task = 'regression',
                                ...
                                # Specify train/test split
                                training_data=training_data,
                                test_size=0.2)
   ```
   To use a train/test split instead of providing test data directly, use the
   `test_size` parameter when creating the `AutoMLConfig`. This parameter
   must be a floating point value between 0.0 and 1.0 and specifies the
   percentage of the training dataset that should be used for the test
   dataset. For regression based tasks, random sampling is used. For
   classification tasks, stratified sampling is used. Forecasting does not
   currently support specifying a test dataset using a train/test split.

   The `test_data` and `test_size` `AutoMLConfig` parameters are mutually
   exclusive and can not be specified at the same time.

3. Use `ModelProxy` to test a model after the main AutoML run has completed
   (aka. "on-demand" model testing).

    ```python
    from azureml.train.automl.model_proxy import ModelProxy

    model_proxy = ModelProxy(best_run)
    predictions, metrics = model_proxy.test(test_data)
    ```
    Supported input test dataset types: [Dataset](https://docs.microsoft.com/en-us/python/api/azureml-core/azureml.core.dataset.dataset?view=azure-ml-py).

### Get the Predictions and Metrics Generated by the Test Run

When a remote test run is requested by passing in a value for `test_data` or
`test_size` to `AutoMLConfig`, only the best training run will have a test run
associated with it.

```python
best_run, fitted_model = remote_run.get_output()
test_run = next(best_run.get_children(type='automl.model_test'))
test_run.wait_for_completion(show_output=False, wait_post_processing=True)

# Get test metrics
test_run_metrics = test_run.get_metrics()
for name, value in test_run_metrics.items():
    print(f"{name}: {value}")

# Get test predictions
test_run_details = test_run.get_details()
test_run_predictions = Dataset.get_by_id(workspace, test_run_details['outputDatasets'][0]['identifier']['savedId'])
test_run_predictions.to_pandas_dataframe().head()
```

`ModelProxy` already returns the predictions and metrics and
does not require further processing to retrieve the outputs.

## Requirements

- To use the testing feature, AzureML SDK versions newer than or equal to 1.27 is required.

## Limitations

- The predictions.csv file which is generated by the model test run will be stored
    in the default datastore which gets created when the machine learning workspace
    is created (`workspaceblobstore`). This datastore is visible to all users with
    the same subscription. Do not use this feature if any of the information used for
    or created by the test run needs to remain private.
- Local AutoML runs do not support test runs. To use this feature, please use remote runs.
- ADB/Spark runs do not support test runs.
- Pandas DataFrames are not supported as input test datasets.
    Supported input test dataset types: [Dataset](https://docs.microsoft.com/en-us/python/api/azureml-core/azureml.core.dataset.dataset?view=azure-ml-py).
- Forecasting runs which have `enable_dnn` set to `True` do not support test runs.
- Forecasting runs do not support train/test split (`test_size`).
- The on-demand model testing feature will not work for runs created with an SDK older than 1.27.
    To use this feature, the run must have been created with SDK version >= 1.27.
- `test_data`/`test_size` is not supported at the same time as `cv_split_column_names`
- `test_data`/`test_size` is not supported at the same time as `cv_splits_indices`
- Test runs are not supported for AutoML runs which have streaming enabled.
- Model testing is not supported for image classification tasks.
- `enable_dnn=True` is not compatible with the model testing feature at this time.

## Contributing

We welcome contributions and suggestions! See `CONTRIBUTING.md` for more details.

### Issues and feedback

All forms of feedback are welcome through this repo's issues:
https://github.com/Azure/automl-devplat2-preview/issues

Please see the [contributing guidelines](CONTRIBUTING.md) for further details.

## Code of Conduct

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). Please see the [code of conduct](CODE_OF_CONDUCT.md) for details.

## Trademarks

This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft 
trademarks or logos is subject to and must follow 
[Microsoft's Trademark & Brand Guidelines](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship.
Any use of third-party trademarks or logos are subject to those third-party's policies.
