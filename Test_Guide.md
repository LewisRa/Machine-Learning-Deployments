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

# Differential Testing
# API and Integration Testing
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

- test_prediction_validation(
    field, field_value, index, expected_error, client, test_inputs_df
    )
 
- test_prediction_data_saved(client, app, test_inputs_df)
