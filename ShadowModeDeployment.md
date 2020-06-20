### Deployment vs Release
Deployments need not be exposed to customers while releases do. For example, the difference between shadow mode and canary release is that shadow mode has no effect on customers while in canary releases,  you release your system yo a limited subset to users.

# Shadow Mode Deployment
Shadow mode is needed because it is extreme difficult to completely replicate the production environment in the testing environment because of several reasons including: 
- seasonality changes
- customer trends
- market forces
- external systems
- changes to the API
- contracts
- data structures

#### How Shadow Mode Works 

At the application level, the incoming prediction requests are performed by our current model and that same prediction requests is also
sent to our shadow model where the prediction is saved but not served back to the customer and all the logic for handling this is done in code by triggering it through a feature flag.However you do need to think about your data model if you're going to capture predictions and pass system to disk either in a database or in your logs.You need a way to distinguish between those shadow and life predictions so that you can effectively run queries as needed.

A more complex way to do a shadow deployment is at the infrastructure level. Shadow prediction requests are forked at the infrastructure level and traffic is routed to two different versions of our API or potentially two different API endpoints. Doing this in a micro services environment can be challenging and tends to require a more mature dev ops setup. If you haven't considered that possibility reasons to use an infrastructure level shadow deployment include your application your machine learning application being a black box that you're simply unable to change. You may have created a prototype in a slower language like Python and you're looking to test it before you go ahead with a rewrite in a faster language like C or C++ or your feature flagging system may be so riddled with technical debt that adding an additional flag to your application is actually less appealing than making infrastructure changes.

####  There are many data checks on some statistical tests that we can implement during shuttle deployment to highlight if there are differences between the data that we use to train the model and live data
- Values Checks - Allowed values
- Values Checks - Value range
- Values Checks - Missing values
- Distribution Checks - Basic statistics (Mean, median, SD, Max, Min)
- Distribution Checks - Statistical tests between trained and live data (numeriCal variables)
  - If the variables are normally distributed (t-tests or anova)
  - If the variables are NOTnormally distributed (t-tests or anova). We could use non parametric tests like Kruskal-Wallis
- Distribution Checks - Statistical tests between trained and live data (categorical variables)
  -ChiSquare
- Distribution Checks - Statistical tests between trained and live data (sparse variables)
  - Most values are zeros (Bag of words)
  - Non-zero values : % training vs live data (ChiSquare)
- Model Performance - Statistics
  - Accuracy, Percison and recall, ROC-AUC
  - MSE, RMSE, MAEM ,....
  - Predictions distribution
- Model Performance - Bespoke population
  - Model data slices where performance is especially unportant ot where model might perform poorly. Use understanding of the data to identify data slices of interest. Compare model metrics for data slices against the metric for your data set. 
  
 #### In app.py under testing-and-monitoring-ml-deployments/packages/ml_api/api/
  ```py
  
    # Setup database
    init_database(flask_app, config=config_object, db_session=db_session)
  ```
  
  #### Setting up the db uses the Persistence directory. We are using PostGres. 
  ## Core.py
  
  ```py
  import logging
import os

import alembic.config
from flask import Flask
from sqlalchemy import create_engine
from sqlalchemy.engine import Engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import scoped_session, sessionmaker
from sqlalchemy_utils import database_exists, create_database

from api.config import Config, ROOT

_logger = logging.getLogger('mlapi')

# Base class for SQLAlchemy models
Base = declarative_base()


def create_db_engine_from_config(*, config: Config) -> Engine:
    """The Engine is the starting point for any SQLAlchemy application.
    It’s “home base” for the actual database and its DBAPI, delivered to the SQLAlchemy
    application through a connection pool and a Dialect, which describes how to talk to
    a specific kind of database / DBAPI combination.
    """

    db_url = config.SQLALCHEMY_DATABASE_URI
    if not database_exists(db_url):
        create_database(db_url)
    engine = create_engine(db_url)

    _logger.info(f"creating DB conn with URI: {db_url}")
    return engine


def create_db_session(*, engine: Engine) -> scoped_session:
    """Broadly speaking, the Session establishes all conversations with the database.
     It represents a “holding zone” for all the objects which you’ve loaded or
     associated with it during its lifespan.
     """
    return scoped_session(sessionmaker(autocommit=False, autoflush=False, bind=engine))


def init_database(app: Flask, config: Config, db_session=None) -> None:
    """Connect to the database and attach DB session to the app."""

    if not db_session:
        engine = create_db_engine_from_config(config=config)
        db_session = create_db_session(engine=engine)

    app.db_session = db_session

    @app.teardown_appcontext
    def shutdown_session(exception=None):
        db_session.remove()


def run_migrations():
    """Run the DB migrations prior to the tests."""

    # alembic looks for the migrations in the current
    # directory so we change to the correct directory.
    os.chdir(str(ROOT))
    alembicArgs = ["--raiseerr", "upgrade", "head"]
    alembic.config.main(argv=alembicArgs)
  ```
You can work with the core (base model) which work with SQL more directly or you can use the object reational mapping abstraction layer which allows us to define our database and columns in code(what we are using here)

### Sessions:
The session establishes all conversations with the database and it's a regular Python class which can be directly instantiated in any database queries persisted within the session. **It seems to be like closing down sql at the end of the work day and it clears all temp tables, for example.** Session.begin() begins a transaction on this session and session.close() closes current session by clearing all items and ending any transaction in progress. 


## Models.py
Classes *LassoModelPredictions* and *GradientBoostingModelPredictions* represents a database tables of predictions. *LassoModelPredictions* for the live trained model and *GradientBoostingModelPredictions* for the shadow model. Both use base class from core.py(sqlalchemy)
```py
from sqlalchemy import Column, String, DateTime, Integer
from sqlalchemy.dialects.postgresql import JSONB
from sqlalchemy.sql import func

from api.persistence.core import Base


class LassoModelPredictions(Base):
    __tablename__ = "regression_model_predictions"
    id = Column(Integer, primary_key=True)
    user_id = Column(String(36), nullable=False)
    datetime_captured = Column(
        DateTime(timezone=True), server_default=func.now(), index=True
    )
    model_version = Column(String(36), nullable=False)
    inputs = Column(JSONB)
    outputs = Column(JSONB)


class GradientBoostingModelPredictions(Base):
    __tablename__ = "gradient_boosting_model_predictions"
    id = Column(Integer, primary_key=True)
    user_id = Column(String(36), nullable=False)
    datetime_captured = Column(
        DateTime(timezone=True), server_default=func.now(), index=True
    )
    model_version = Column(String(36), nullable=False)
    inputs = Column(JSONB)
    outputs = Column(JSONB)
```

## Data_access.py

```py
import enum
import json
import logging
import typing as t

import numpy as np
import pandas as pd
from regression_model.predict import make_prediction as make_live_prediction
from sqlalchemy.orm.session import Session

from api.persistence.models import (
    LassoModelPredictions,
    GradientBoostingModelPredictions,
)
from gradient_boosting_model.predict import make_prediction as make_shadow_prediction

_logger = logging.getLogger('mlapi')


SECONDARY_VARIABLES_TO_RENAME = {
    "FirstFlrSF": "1stFlrSF",
    "SecondFlrSF": "2ndFlrSF",
    "ThreeSsnPortch": "3SsnPorch",
}


class ModelType(enum.Enum):
    LASSO = "lasso"
    GRADIENT_BOOSTING = "gradient_boosting"


class PredictionResult(t.NamedTuple):
    errors: t.Any
    predictions: np.array
    model_version: str


MODEL_PREDICTION_MAP = {
    ModelType.GRADIENT_BOOSTING: make_shadow_prediction,
    ModelType.LASSO: make_live_prediction,
}


class PredictionPersistence:
    def __init__(self, *, db_session: Session, user_id: str = None) -> None:
        self.db_session = db_session
        if not user_id:
            # in reality, here we would use something like a UUID for anonymous users
            # and if we had user logins, we would record the user ID.
            self.user_id = "007"

    def make_save_predictions(
        self, *, db_model: ModelType, input_data: t.List
    ) -> PredictionResult:
        """Get the prediction from a given model and persist it."""
        # Access the model prediction function via mapping
        if db_model == ModelType.LASSO:
            # we have to rename a few of the columns for backwards
            # compatibility with the regression model package.
            live_frame = pd.DataFrame(input_data)
            input_data = live_frame.rename(
                columns=SECONDARY_VARIABLES_TO_RENAME
            ).to_dict(orient="records")

        result = MODEL_PREDICTION_MAP[db_model](input_data=input_data)
        errors = None
        try:
            errors = result["errors"]
        except KeyError:
            # regression model `make_prediction` does not include errors
            pass

        prediction_result = PredictionResult(
            errors=errors,
            predictions=result.get("predictions").tolist() if not errors else None,
            model_version=result.get("version"),
        )

        if prediction_result.errors:
            return prediction_result

        self.save_predictions(
            inputs=input_data, prediction_result=prediction_result, db_model=db_model
        )

        return prediction_result

    def save_predictions(
        self,
        *,
        inputs: t.List,
        prediction_result: PredictionResult,
        db_model: ModelType,
    ) -> None:
        """Persist model predictions to storage."""
        if db_model == db_model.LASSO:
            prediction_data = LassoModelPredictions(
                user_id=self.user_id,
                model_version=prediction_result.model_version,
                inputs=json.dumps(inputs),
                outputs=json.dumps(prediction_result.predictions),
            )
        else:
            prediction_data = GradientBoostingModelPredictions(
                user_id=self.user_id,
                model_version=prediction_result.model_version,
                inputs=json.dumps(inputs),
                outputs=json.dumps(prediction_result.predictions),
            )

        self.db_session.add(prediction_data)
        self.db_session.commit()
        _logger.debug(f"saved data for model: {db_model}")
```
### Changes to Config module under

```py
class Config:
    DEBUG = False
    TESTING = False
    ENV = os.getenv("FLASK_ENV", "production")
    SERVER_PORT = int(os.getenv("SERVER_PORT", 5000))
    SERVER_HOST = os.getenv("SERVER_HOST", "0.0.0.0")
    LOGGING_LEVEL = os.getenv("LOGGING_LEVEL", logging.INFO)
    SHADOW_MODE_ACTIVE = os.getenv('SHADOW_MODE_ACTIVE', True)
    #SQLALCHEMY_DATABASE_URI = (
       #f"postgresql+psycopg2://{os.getenv('DB_USER')}:"
        #f"{os.getenv('DB_PASSWORD')}@{os.getenv('DB_HOST')}/{os.getenv('DB_NAME')}"
    )
```
## Database Migrations
 - Make/ delete a new table
 - Add/ remove columns
 - Modify existing data in a table
 
 We can migration to a new revision of table or migrate back using migration script. 
 
 #### Database Migration Libraries
 - Alembic
 - Django Migrations
 - Yoyo Database Migrations
 - Migrate
 - SQLAlchemy Migrate
 - SQL Migration Runner
 
 **https://www.youtube.com/watch?v=36yw8VC3KU8&t=414s 21:00**
 ## The Alembic Migration Environment

The structure of this environment, including some generated migration scripts, looks like:

```
yourproject/
    alembic/
        env.py
        README
        script.py.mako
        versions/
            3512b954651e_add_account.py
            2b1ae634e5cd_add_order_id.py
            3adcc9a56557_rename_username_field.py
```
The directory includes these directories/files:

yourproject - this is the root of your application’s source code, or some directory within it.

alembic - this directory lives within your application’s source tree and is the home of the migration environment. It can be named anything, and a project that uses multiple databases may even have more than one.

env.py - This is a Python script that is run whenever the alembic migration tool is invoked. At the very least, it contains instructions to configure and generate a SQLAlchemy engine, procure a connection from that engine along with a transaction, and then invoke the migration engine, using the connection as a source of database connectivity.

The env.py script is part of the generated environment so that the way migrations run is entirely customizable. The exact specifics of how to connect are here, as well as the specifics of how the migration environment are invoked. The script can be modified so that multiple engines can be operated upon, custom arguments can be passed into the migration environment, application-specific libraries and models can be loaded in and made available. Alembic includes a set of initialization templates which feature different varieties of env.py for different use cases.

script.py.mako - This is a Mako template file which is used to generate new migration scripts. Whatever is here is used to generate new files within versions/. This is scriptable so that the structure of each migration file can be controlled, including standard imports to be within each, as well as changes to the structure of the upgrade() and downgrade() functions. For example, the multidb environment allows for multiple functions to be generated using a naming scheme upgrade_engine1(), upgrade_engine2().

**https://alembic.sqlalchemy.org/en/latest/tutorial.html **
 
 #### Creating an Alembic Environment
 ```
 alembic init alembic
 ```
 #### Configuring alembic.ini
 ![](https://github.com/LewisRa/Machine-Learning-Deployments/blob/master/markdownImages/alembicConfig.PNG)
 #### Creating a Migration
 ```
 alembic revision -m "create organization table"
  ```
  Output: Generatring h/migrations/versions/1975ea83b712_create_organization_table.py...done
  
  This command creates a python file.: 
  ![](https://github.com/LewisRa/Machine-Learning-Deployments/blob/master/markdownImages/alembicCreate.PNG)
  Don't forget to add organization

#### For example,  we're adding a new organization table and the concept of organizations into our database 

1. Add the organization table into our database 


![](https://github.com/LewisRa/Machine-Learning-Deployments/blob/master/markdownImages/alembicCreate2.PNG)

2. Define a relationship between the existing groups table and the new
organization table.


![](https://github.com/LewisRa/Machine-Learning-Deployments/blob/master/markdownImages/alembicModify.PNG)


3. Define for all of our existing groups  to an organization that they belong to so that no
group is allowed to be without an organization.


![](https://github.com/LewisRa/Machine-Learning-Deployments/blob/master/markdownImages/alembicModify3.PNG)

- 3.1
Since we're using the organization model and the  group model, it might be tempting to import from our
existing models in our code like above but that is a big mistake. 

Since the organization model and the group model may change over time right those are not
necessarily static and this migration
assumes that those models look a certain
way it 

Because importing the current organization and group modelassumes that they have certain
columns and in two years from now those columns might not actually exist in those models and we always want our migration or migration script should work for years to come. We should be able
to two years from now revert this migration if we want to. In order to do that, we define
our models inside of this migration and this will ensure that we can run this migration script in two or three years

![](https://github.com/LewisRa/Machine-Learning-Deployments/blob/master/markdownImages/alembicModify4.PNG)
