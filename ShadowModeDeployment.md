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
  
  #### Setting up the db uses the Persistence directory
  #### Using core.py
  
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
