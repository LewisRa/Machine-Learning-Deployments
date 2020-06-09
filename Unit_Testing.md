 # Unit Testing a Production ML Model
## tox.ini


In testing-and-monitoring-ml-deployments/packages/gradient_boosting_model/ run: 
```
tox 
tox > toxOutput.txt
tox -r - the -r is a flag to rebuild our virtual environments, including redownloading module via pip install.
```
```diff
[tox]
- envlist = unit_tests,typechecks,stylechecks
skipsdist = True


-[testenv]
install_command = pip install {opts} {packages}
deps =
    -rtest_requirements.txt

passenv =
     KAGGLE_USERNAME
     KAGGLE_KEY

setenv =
  PYTHONPATH=.

commands=
    kaggle competitions download -c house-prices-advanced-regression-techniques -p gradient_boosting_model/datasets/
    unzip -o gradient_boosting_model/datasets/house-prices-advanced-regression-techniques.zip -d gradient_boosting_model/datasets
    mv gradient_boosting_model/datasets/train.csv gradient_boosting_model/datasets/houseprice.csv
-    python gradient_boosting_model/train_pipeline.py
- pytest \
-     -s \
-     -vv \
-          {posargs:tests/}


-[testenv:unit_tests]
envdir = {toxworkdir}/unit_tests
deps =
     {[testenv]deps}

setenv =
  PYTHONPATH=.

commands =
     python gradient_boosting_model/train_pipeline.py
-     pytest \
-           -s \
-           -vv \
-           {posargs:tests/}


-[testenv:typechecks]
envdir = {toxworkdir}/unit_tests

deps =
     {[testenv:unit_tests]deps}

-commands = {posargs:mypy gradient_boosting_model}


-[testenv:stylechecks]
envdir = {toxworkdir}/unit_tests

deps =
     {[testenv:unit_tests]deps}

-commands = {posargs:flake8 gradient_boosting_model tests}


[flake8]
exclude = .git,env
max-line-length = 90
```

### Run train_pipeline without tox and pytest
```

pip install .
....
python3
x=3 
locals()  will give you a dictionary of local variables--> x:3
globals() will give you a dictionary of global variables--> x:3
dir() will give you the list of in scope variables--> x
```

## DataManagement.py
```
import pandas as pd
import joblib
from sklearn.pipeline import Pipeline

from gradient_boosting_model.config.core import config, DATASET_DIR, TRAINED_MODEL_DIR
from gradient_boosting_model import __version__ as _version

import logging
import typing as t


_logger = logging.getLogger(__name__)


def load_dataset(*, file_name: str) -> pd.DataFrame:
    dataframe = pd.read_csv(f"{DATASET_DIR}/{file_name}")

    # rename variables beginning with numbers to avoid syntax errors later
    transformed = dataframe.rename(columns=config.model_config.variables_to_rename)
    return transformed


def save_pipeline(*, pipeline_to_persist: Pipeline) -> None:
    """Persist the pipeline.
    Saves the versioned model, and overwrites any previous
    saved models. This ensures that when the package is
    published, there is only one trained model that can be
    called, and we know exactly how it was built.
    """

    # Prepare versioned save file name
    save_file_name = f"{config.app_config.pipeline_save_file}{_version}.pkl"
    save_path = TRAINED_MODEL_DIR / save_file_name

    remove_old_pipelines(files_to_keep=[save_file_name])
    joblib.dump(pipeline_to_persist, save_path)
    _logger.info(f"saved pipeline: {save_file_name}")


def load_pipeline(*, file_name: str) -> Pipeline:
    """Load a persisted pipeline."""

    file_path = TRAINED_MODEL_DIR / file_name
    trained_model = joblib.load(filename=file_path)
    return trained_model


def remove_old_pipelines(*, files_to_keep: t.List[str]) -> None:
    """
    Remove old model pipelines.
    This is to ensure there is a simple one-to-one
    mapping between the package version and the model
    version to be imported and used by other applications.
    """
    do_not_delete = files_to_keep + ["__init__.py"]
    for model_file in TRAINED_MODEL_DIR.iterdir():
        if model_file.name not in do_not_delete:
            model_file.unlink()
 ```
 #### load_dataset
- packages/gradient_boosting_model/gradient_boosting_model/train_pipeline.py
- packages/gradient_boosting_model/tests/conftest.py
- packages/ml_api/scripts/populate_database.py
- packages/ml_api/tests/conftest.py
- packages/ml_api/tests/test_back_to_back_models.py
- packages/ml_api/tests/test_api.py
 #### save_pipeline used in testing-and-monitoring-ml-deployments/packages/gradient_boosting_model/gradient_boosting_model/train_pipeline.py 
 #### load_pipeline used in packages/gradient_boosting_model/gradient_boosting_model/predict.py
 #### remove_old_pipelines used in save_pipeline function

## Validation.py
```
import typing as t

from gradient_boosting_model.config.core import config

import numpy as np
import pandas as pd
from marshmallow import fields, Schema, ValidationError


class HouseDataInputSchema(Schema):
    Alley = fields.Str(allow_none=True)
    BedroomAbvGr = fields.Integer()
    BldgType = fields.Str()
    BsmtCond = fields.Str(allow_none=True)
    BsmtExposure = fields.Str(allow_none=True)
    BsmtFinSF1 = fields.Float(allow_none=True)
    BsmtFinSF2 = fields.Float(allow_none=True)
    BsmtFinType1 = fields.Str(allow_none=True)
    BsmtFinType2 = fields.Str(allow_none=True)
    BsmtFullBath = fields.Float(allow_none=True)
    BsmtHalfBath = fields.Float(allow_none=True)
    BsmtQual = fields.Str(allow_none=True)
    BsmtUnfSF = fields.Float()
    CentralAir = fields.Str()
    Condition1 = fields.Str()
    Condition2 = fields.Str()
    Electrical = fields.Str(allow_none=True)
    EnclosedPorch = fields.Integer()
    ExterCond = fields.Str()
    ExterQual = fields.Str()
    Exterior1st = fields.Str(allow_none=True)
    Exterior2nd = fields.Str(allow_none=True)
    Fence = fields.Str(allow_none=True)
    FireplaceQu = fields.Str(allow_none=True)
    Fireplaces = fields.Integer()
    Foundation = fields.Str()
    FullBath = fields.Integer()
    Functional = fields.Str(allow_none=True)
    GarageArea = fields.Float()
    GarageCars = fields.Float()
    GarageCond = fields.Str(allow_none=True)
    GarageFinish = fields.Str(allow_none=True)
    GarageQual = fields.Str(allow_none=True)
    GarageType = fields.Str(allow_none=True)
    GarageYrBlt = fields.Float(allow_none=True)
    GrLivArea = fields.Integer()
    HalfBath = fields.Integer()
    Heating = fields.Str()
    HeatingQC = fields.Str()
    HouseStyle = fields.Str()
    Id = fields.Integer()
    KitchenAbvGr = fields.Integer()
    KitchenQual = fields.Str(allow_none=True)
    LandContour = fields.Str()
    LandSlope = fields.Str()
    LotArea = fields.Integer()
    LotConfig = fields.Str()
    LotFrontage = fields.Float(allow_none=True)
    LotShape = fields.Str()
    LowQualFinSF = fields.Integer()
    MSSubClass = fields.Integer()
    MSZoning = fields.Str(allow_none=True)
    MasVnrArea = fields.Float(allow_none=True)
    MasVnrType = fields.Str(allow_none=True)
    MiscFeature = fields.Str(allow_none=True)
    MiscVal = fields.Integer()
    MoSold = fields.Integer()
    Neighborhood = fields.Str()
    OpenPorchSF = fields.Integer()
    OverallCond = fields.Integer()
    OverallQual = fields.Integer()
    PavedDrive = fields.Str()
    PoolArea = fields.Integer()
    PoolQC = fields.Str(allow_none=True)
    RoofMatl = fields.Str()
    RoofStyle = fields.Str()
    SaleCondition = fields.Str()
    SaleType = fields.Str(allow_none=True)
    ScreenPorch = fields.Integer()
    Street = fields.Str()
    TotRmsAbvGrd = fields.Integer()
    TotalBsmtSF = fields.Float()
    Utilities = fields.Str(allow_none=True)
    WoodDeckSF = fields.Integer()
    YearBuilt = fields.Integer()
    YearRemodAdd = fields.Integer()
    YrSold = fields.Integer()
    FirstFlrSF = fields.Integer()
    SecondFlrSF = fields.Integer()
    ThreeSsnPortch = fields.Integer()


def drop_na_inputs(*, input_data: pd.DataFrame) -> pd.DataFrame:
    """Check model inputs for na values and filter."""
    validated_data = input_data.copy()
    if input_data[config.model_config.numerical_na_not_allowed].isnull().any().any():
        validated_data = validated_data.dropna(
            axis=0, subset=config.model_config.numerical_na_not_allowed
        )

    return validated_data


def validate_inputs(
    *, input_data: pd.DataFrame
) -> t.Tuple[pd.DataFrame, t.Optional[dict]]:
    """Check model inputs for unprocessable values."""

    # convert syntax error field names (beginning with numbers)
    input_data.rename(columns=config.model_config.variables_to_rename, inplace=True)
    validated_data = drop_na_inputs(input_data=input_data)

    # set many=True to allow passing in a list
    schema = HouseDataInputSchema(many=True)
    errors = None

    try:
        # replace numpy nans so that Marshmallow can validate
        schema.load(validated_data.replace({np.nan: None}).to_dict(orient="records"))
    except ValidationError as exc:
        errors = exc.messages

    return validated_data, errors
```

### testing-and-monitoring-ml-deployments/packages/gradient_boosting_model/gradient_boosting_model/config/core.py /
### testing-and-monitoring-ml-deployments/packages/gradient_boosting_model/gradient_boosting_model/config.yml
#### Validate_inputs
- packages/gradient_boosting_model/gradient_boosting_model/predict.py
- testing-and-monitoring-ml-deployments/packages/gradient_boosting_model/tests/test_validation.py
- packages/gradient_boosting_model/tests/test_pipeline.py

#### drop_na_inputs not used? 

## preprocessor.py 

#### http://www.ashukumar27.io/sklearn_pipelines/

## errors.py
---


## 1. Build Pipeline - testing-and-monitoring-ml-deployments/packages/gradient_boosting_model/gradient_boosting_model/pipeline.py

- numerical_imputer
  - variables=config.model_config.numerical_vars

- categorical_imputer
  - variables=config.model_config.categorical_vars

- temporal_variable (preprocessor.py) ##Calculates the time difference between 2 temporal variables. **Subtract the differences in years from YearRemodAdd (Remodel date) and YrSold(Feature to be dropped) and replace data in YearRemodAdd with those differences **
   - variables=config.model_config.temporal_vars,
   - reference_variable=config.model_config.drop_features,

- rare_label_encoder ##RareLabelCategoricalEncoder() groups infrequent categories altogether into one new category called ‘Rare’ or a different string indicated by the user. (from feature_engine.categorical_encoders import RareLabelCategoricalEncoder)
  - tol=config.model_config.rare_label_tol,
  - n_categories=config.model_config.rare_label_n_categories,
  - variables=config.model_config.categorical_vars,

- categorical_encoder

- drop_features
  - variables_to_drop=config.model_config.drop_features

- gb_model

---
# Testing

## conftest.py	
## test_config.py	

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
## test_predict.py	
## test_preprocessors.py	
## test_validation.py

###
