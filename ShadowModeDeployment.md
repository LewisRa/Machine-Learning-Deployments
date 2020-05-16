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
# ML Monitioring
