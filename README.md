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
 
  
  
    
    
  
