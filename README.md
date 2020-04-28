# Machine-Learning-Deployments

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
   - Differential tests
   - Benchmark tests
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
    
    
  
