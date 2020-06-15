# Test Guide

# Unit Testing

## conftest.py	
```
@pytest.fixture(scope="session")
def pipeline_inputs():
    # For larger datasets, here we would use a testing sub-sample.
    data = load_dataset(file_name=config.app_config.training_data_file)

    # Divide train and test
    X_train, X_test, y_train, y_test = train_test_split(
        data[config.model_config.features],  # predictors
        data[config.model_config.target],
        test_size=config.model_config.test_size,
        # we are setting the random seed here
        # for reproducibility
        random_state=config.model_config.random_state,
    )

    return X_train, X_test, y_train, y_test


@pytest.fixture()
def raw_training_data():
    # For larger datasets, here we would use a testing sub-sample.
    return load_dataset(file_name=config.app_config.training_data_file)


@pytest.fixture()
def sample_input_data():
    return load_dataset(file_name=config.app_config.test_data_file)
```

## test_config.py	

- test_fetch_config_structure(tmpdir):
- test_config_validation_raises_error_for_invalid_config(tmpdir)

We have a test conflict text string which is designed to mimic our config gamble.

We also have an invalid text config and we'll return to that shortly in the first test here.
from gradient_boosting_model.config.core import (
    create_and_validate_config,
    fetch_config_from_yaml,

from pydantic import ValidationError


TEST_CONFIG_TEXT 
loss: ls
allowed_loss_functions:
  - ls
  - huber

INVALID_TEST_CONFIG_TEXT 
loss: ls
allowed_loss_functions:
  - huber
  
  
test_fetch_config_structure(tmpdir):
We test a valid config ensuring that it loads correctly and that our validation runs as expected.
**test_config_validation_raises_error_for_invalid_config(tmpdir)**
**test_missing_config_field_raises_validation_error(tmpdir)**
This test makes sure that if we have any missing config fields we also raise a validation error by only passing in a test config with one field(the package name). We should  expect a validation error.(PYDANTIC AS VALIDATION ERROR)  **(Check that  field required is in error messge)** **(see if temp dir is in the system temporary directory. The base name will be pytest-NUM where NUM will be incremented with each test run. Moreover, entries older than 3 temporary directories will be removed.- https://docs.pytest.org/en/2.7.3/tmpdir.html)
## test_pipeline.py	

- test_pipeline_drops_unnecessary_features(pipeline_inputs)
- test_pipeline_transforms_temporal_features(pipeline_inputs)
## test_predict.py	

- test_prediction_quality_against_benchmark(raw_training_data, sample_input_data)
- test_prediction_quality_against_another_model(raw_training_data, sample_input_data):
## test_preprocessors.py	

- test_drop_unnecessary_features_transformer(pipeline_inputs)
- test_temporal_variable_estimator(pipeline_inputs)

## test_validation.py

- test_validate_inputs(sample_input_data)
- test_validate_inputs_identifies_errors(sample_input_data)


# API and Integration Testing

### parameterizationa allows us to try many combinations of data within the same test, see the pytest docs for details 

```
@pytest.fixture(scope='session')
def _db():
    db_url = TestingConfig.SQLALCHEMY_DATABASE_URI
    if not database_exists(db_url):
        create_database(db_url)
    # alembic can be configured through the configuration file. For testing
    # purposes 'env.py' also checks the 'ALEMBIC_DB_URI' variable first.
    engine = core.create_db_engine_from_config(config=TestingConfig())
    evars = {"ALEMBIC_DB_URI": db_url}
    with mock.patch.dict(os.environ, evars):
        core.run_migrations()

    yield engine


@pytest.fixture(scope='session')
def _db_session(_db):
    """ Create DB session for testing.
    """
    session = core.create_db_session(engine=_db)
    yield session


@pytest.fixture(scope='session')
def app(_db_session):
    app = create_app(config_object=TestingConfig(), db_session=_db_session).app
    with app.app_context():
        yield app


@pytest.fixture
def client(app):
    with app.test_client() as client:
        yield client  # Has to be yielded to access session cookies


@pytest.fixture
def test_inputs_df():
    # Load the gradient boosting test dataset which
    # is included in the model package
    test_inputs_df = load_dataset(file_name="test.csv")
    return test_inputs_df.copy(deep=True)
```

**The app fixture which calls our create app factory function from app.py file to instantiate a connection app that passing in a test configuration from the config module**

## Flask:app.test_client() as client 

Passing flask app, in client function(fixture), you can test flask app endpoints! For example:
```py
# contents from routes.py

from flask import request
import json


def configure_routes(app):

    @app.route('/')
    def hello_world():
        return 'Hello, World!'
  ```
  
  ```py
 # contents from test_routes.py
 
from flask import Flask
import json

from flask_pytest_example.handlers.routes import configure_routes


def test_base_route():
    app = Flask(__name__)
    configure_routes(app)
    client = app.test_client()
    url = '/'

    response = client.get(url)
    assert response.get_data() == b'Hello, World!'
    assert response.status_code == 200
  ```

## test_api.py
- @pytest.mark.integration
  def test_health_endpoint(client)
...
```
@pytest.mark.integration
@pytest.mark.parametrize(
    "api_endpoint, expected_no_predictions",
    (
        (
            "v1/predictions/regression",
            # test csv contains 1459 rows
            # we expect 2 rows to be filtered
            1451,
        ),
        (
            "v1/predictions/gradient",
            # we expect 8 rows to be filtered
            1457,
        ),
    ),
)
```
-  test_prediction_endpoint(
    api_endpoint, expected_no_predictions, client, test_inputs_df
    )
    
 **expected_no_predictions = expected number of predictions i.e 1451 rows for regression model and 1457 rows for the gradient boosting models**
 
 
 **response = client.post(api_endpoint, json=test_inputs_df.to_dict(orient="records"))**
This line posts an HTTP post request to our V1 predictions endpoint passing in  the test data as a JSON file (python dictionary)
then we inspect our response to ensure not only that there are no errors but that the length of the predictions matches the expected output length.
```diff
 assert response.status_code == 200
    data = json.loads(response.data)
-    assert data["errors"] is None
    assert len(data["predictions"]) == expected_no_predictions
```

**test_prediction_endpoint** is essentially an integration test for the unit test:test_validate_inputs(sample_input_data):

- test_prediction_validation(
    field, field_value, index, expected_error, client, test_inputs_df
    )

```
@pytest.mark.parametrize(
    "field, field_value, index, expected_error",
    (
        (
            "BldgType",
            1,  # expected str
            33,
            {"33": {"BldgType": ["Not a valid string."]}},
        ),
        (
            "GarageArea",  # model feature
            "abc",  # expected float
            45,
            {"45": {"GarageArea": ["Not a valid number."]}},
        ),
        (
            "CentralAir",
            np.nan,  # nan not allowed
            34,
            {"34": {"CentralAir": ["Field may not be null."]}},
        ),
        ("LotArea", "", 2, {"2": {"LotArea": ["Not a valid integer."]}}),
    ),
)
```
For example: This code is specifying that at index 33 we're setting the BldgType = 1 and then based on that, when we call post with that test input data frame. Thus, we expect that the data we get back matches this expected error {"33": {"BldgType": ["Not a valid string."]} 
 
 **test_prediction_validation** is essentially an integration test for the unit test:test_validate_inputs_identifies_errors(sample_input_data):


- test_prediction_data_saved(client, app, test_inputs_df) (shadow mode deployment testing)

## test_back_to_back_models.py ( Differential Testing)
- @pytest.mark.differential
  def test_model_prediction_differentials(client)
 
 Using testing-and-monitoring-ml-deployments/packages/ml_api/differential_tests/compare.py, this test compares the regresssion and gradient boosting models.  We just pass in the first 10 rows as the two models' validation differs which means they filter out a slightly different number of rows which would cause the differential tests to fail. The tolerence level can be set to the level of variation desired.      

  

## test_persistence.py
- @pytest.mark.parametrize(
        "model_type, model,",
        (
            (ModelType.GRADIENT_BOOSTING, GradientBoostingModelPredictions),
            (ModelType.LASSO, LassoModelPredictions),
        ),
        )
        def test_data_access(model_type, model, test_inputs_df)
        
        

