   # Machine-Learning-Deployments
"The degree of a system's observability is the degree to which it can be debugged"
## ML System Lifecycle (tests)
- Traditional tests
  - Unit tests
  - Integration tests
  - System tests
- ML specific tests
  - Data tests
  - Model tests
  - ML infrastructure tests
- Tests when the model is running in production (Shadow mode tests)
  - Skew Tests
  - Data monitoring tests
  - Prediction Monitoring tests
  
  ## Building A ML System: Key Phrases
  
  1.  The Research Environment
   - Testing different environments
   - Testing different configurations and hyperparameters
   - Testing different different data and features
  2. Development
   - Unit, integration, and acceptance tests
   - Differential tests - checks model accuracy from one version to the next
   - Benchmark tests - checks model accuracy against a simple benchmark
   - Load tests
  3. Production Environment
    - Shadow mode testing
    - Canary releases
    - Observablity & Monitoring
    - Logging & Tracing
    - Alerting
    
    
    
 ## Why Test? 
Testing is the way we show our system functionality is what we expect it to be, even as we make changes to the system
  1.  Confidence
   - Analyzing data provided by monitoring historic system behavior (past relablity)
  2. Predicting your system's future reliability 
   - Most software systems are frequently changed and teams must be able to confidently describe the change and its impact
  3. Confidence that functionality remains unchanged (version control AND MORE)
  
  ## Testing ML Systems
  Unlike traditional software which only has to tests code, ML models and system have to test code, data, and models
  
 ### Key Tesing Principles for ML
 - Use a Schema for Features
 - Model specifiction tests (model config (i.e. hyper parameters) need unit testing
 - Validate model quality
 - Test input feature code
 - Training is Reproducible (Setting Random Seed, etc)
 - Integration Test the Pipeline
 
 .....................
 ## Code Base Overview
 
  
  ```# Package Overview
package_name: gradient_boosting_model

# Data Files
training_data_file: houseprice.csv
test_data_file: test.csv

# this variable is to calculate the temporal variable
# but is dropped prior to model training.
drop_features: YrSold

pipeline_name: gb_regression
pipeline_save_file: gb_regression_output_v

# Variables
# The variable we are attempting to predict (sale price)
target: SalePrice

# Will cause syntax errors since they begin with numbers
variables_to_rename:
  1stFlrSF: FirstFlrSF
  2ndFlrSF: SecondFlrSF
  3SsnPorch: ThreeSsnPortch

features:
  - LotArea
  - OverallQual
  - YearRemodAdd
  - BsmtQual
  - BsmtFinSF1
  - TotalBsmtSF
  - FirstFlrSF
  - SecondFlrSF
  - GrLivArea
  - GarageCars
    # this one is only to calculate temporal variable:
  - YrSold

numerical_vars:
  - LotArea
  - OverallQual
  - YearRemodAdd
  - BsmtQual
  - BsmtFinSF1
  - TotalBsmtSF
  - FirstFlrSF
  - SecondFlrSF
  - GrLivArea
  - GarageCars

categorical_vars:
  - BsmtQual

temporal_vars: YearRemodAdd

# Validation
# numerical variables with NA in train set
numerical_vars_with_na:
  - LotFrontage

numerical_na_not_allowed:
  - LotArea
  - OverallQual
  - YearRemodAdd
  - BsmtFinSF1
  - TotalBsmtSF
  - FirstFlrSF
  - SecondFlrSF
  - GrLivArea
  - GarageCars
  - YrSold

# set train/test split
test_size: 0.1

# to set the random seed
random_state: 0

# The number of boosting stages to perform
n_estimators: 50

# the minimum frequency a label should have to be considered frequent
# and not be removed.
rare_label_tol: 0.01

# the minimum number of categories a variable should have in order for
# the encoder to find frequent labels
rare_label_n_categories: 5

# loss function to be optimized
loss: ls
allowed_loss_functions:
  - ls
  - huber
  ```
    
## Unit Tests
### Data Engineering 
- reduce risk of bugs in processing/fecture engineering code

### Input Data Tests 
- catch unexpected inputs, typically througha schema

A schema is a collection of rules which specify the expected values for a set of fields. Below we show a simple schema (just using a nested dictionary) for the Iris dataset.(data validation). The schema specifies the maximum and minimum values that can be taken by each variable. We can learn these values from the dataor these values may come from specific domain knowledge of the subject.

```py

# View summary statistics for our dataframe.
iris_frame.describe()

ris_schema = {
    'sepal length': {
        'range': {
            'min': 4.0,  # determined by looking at the dataframe .describe() method
            'max': 8.0
        },
        'dtype': float,
    },
    'sepal width': {
        'range': {
            'min': 1.0,
            'max': 5.0
        },
        'dtype': float,
    },
    'petal length': {
        'range': {
            'min': 1.0,
            'max': 7.0
        },
        'dtype': float,
    },
    'petal width': {
        'range': {
            'min': 0.1,
            'max': 3.0
        },
        'dtype': float,
    }
}

import unittest
import sys

class TestIrisInputData(unittest.TestCase):
    def setUp(self):
        
        # `setUp` will be run before each test, ensuring that you
        # have a new pipeline to access in your tests. See the 
        # unittest docs if you are unfamiliar with unittest.
        # https://docs.python.org/3/library/unittest.html#unittest.TestCase.setUp
        self.pipeline = SimplePipeline()
        self.pipeline.run_pipeline()
    
    def test_input_data_ranges(self):
        # get df max and min values for each column
        max_values = self.pipeline.frame.max()
        min_values = self.pipeline.frame.min()
        
        # loop over each feature (i.e. all 4 column names)
        for feature in self.pipeline.feature_names:
            
            # use unittest assertions to ensure the max/min values found in the dataset
            # are less than/greater than those expected by the schema max/min.
            self.assertTrue(max_values[feature] <= iris_schema[feature]['range']['max'])
            self.assertTrue(min_values[feature] >= iris_schema[feature]['range']['min'])
            
    def test_input_data_types(self):
        data_types = self.pipeline.frame.dtypes  # pandas dtypes method
        
        for feature in self.pipeline.feature_names:
            self.assertEqual(data_types[feature], iris_schema[feature]['dtype'])

# setup code to allow unittest to run the above tests inside the jupyter notebook.
suite = unittest.TestLoader().loadTestsFromTestCase(TestIrisInputData)
unittest.TextTestRunner(verbosity=1, stream=sys.stderr).run(suite)            
```

### Config Tests 
- reduce the risk of errors in our system configuration
### Model Quality Test
- reduce the risk sudden and gradual drops in model quality

# Integration Testing the ML API

Our Integration Tests will check functionality across our API & Model Components

## Tests before Production
- Units
- Integration Tests
- Static Analysis
- Acceptence Tests
- BDD Tests
- Benchmark Tests
- Load Tests
- UI Tests
- SenPen Tests
- Soak Tests
- Differential Tests

## Tests after Production
#### Deploy
- Shadow Mode
- Load Tests
- Config Tests
#### Release 
- Canary Release
- Monitoring 
- Feature Flagging
#### Post Release
- Logs/events
- Monitoring
- Tracing
- A/B Tests
- Auditing/Analysis
- SenPen Tests
    
# ML Monitoring 
1. Operational -is it working
   - Latencies
   - Memory size
   - CPU
2. Are the predictions accurate (Model Outputs)
3. Is the data what is expected (Model Inputs)

#### 3 Pillars of Observability
1. Logs (timestamp, payload) 
 - 3 types: plain text, json, binary (logs in protoype format)
2. Metrics
3. Distributed Tracing- tracking event logs flows through a distributed system (very few ML monitoring uses)

#### 3 components of montioring system
 1.Processing and Storage
 2. Visualization
 3. Alerting

## Alerting 
Responsive component of a monitoring system that performs actions based on changes in metric values and definitions are composed of two components a metrics based condition or threshold and an action to perform when the values fall outside of the acceptable conditions.
Examples: 
 - A feature becoming unavailable
 - Shifts in distribution of ket input values
 - Patterns specific to your model
 - Decrease in performance
3 Key Aspects of Alerting
- Notify -define situations that make sense to activate manage
- Automate - programmatic responses can be triggered based on threshold violations 
- Triage - Distinctions in alert severity depending on the scale of the problem - teams or individuals can be notified using methods appropriate to the scale of the problem. For example rising utilization of storage might warrant a ticket or an email whereas an increase in client facing error rates or unresponsiveness might require sending a page to on call staff in the middle of the night


## Model Input monitoring
## Distribution monitoring
## Monitoring Logs and Metrics
While metrics showed tendencies and patterns of a service or application logs focus on specific events.

The purpose of logs is to preserve as much information mostly technical as possible on a specific occurrence.The information in logs can then be used to investigate incidents and to help with Root Cause Analysis of false hope bugs.

# Distributed Tracing
A represenaation of a series of causally related distibuted events that encode the end to end request flow through a distributed system


