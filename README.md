# RaaS
Notes and thoughts on rollout-as-a-service. Maybe more some day

So this started this project - https://twitter.com/rbtcollins/status/1118648429113012224

There are lots of different strategies for rolling out code, with two core dimensions.
https://thenewstack.io/deployment-strategies/

The two dimensions being when you switch the expected contents on disk, and when you switch
code paths that are executing. 

For some things like system configuration this is typically a disk-content change; things
like A-B testing can be done as either a disk-content change (be that a new binary, or a
different config file), or via a dynamic check in the system (e.g. looking at an attribute
on the user).

# Hypothesis

Having an API that abstracts out the rollout process and provides pre-canned common
interaction patterns would offer enough value that either a community of contributors, or
a community of subscribers to a hosted endpoint, or both, could grow around it.



# Rationale

TODO: look into academic research in these areas.

Really good rollout strategies implementations are non-trivial:
* ML (temporal effects, change impact prediction)
* Circuit breaking when things go off the rails
* Operator affordances to override the system
* Reporting and visualisation of rollout propogation, correlation with fault rates.

Sophisticated ollout strategies are currently adhoc'd or missing in many open-source environments:
* ansible/puppet/chef/salt fleets
* OpenStack / K8s infrastructure itself
* Client devices (Linux OS versions, OS package deliveries, (open source) app stores
* Embedded and IoT scenarios
* Open Source directly distributed applications

Existing strategies are super-simple or limited in scope:
* k8s deploy/statefulset controllers cannot collaborate across a fleet of clusters (today)
* Nearly every open source project releases by uploading binaries and hoping users don't get broken

# Use case 1 - self updating client binary

The client binary would:
 - report its own health anonymously at some developer-chosen semi-randomised schedule
   This needs to report: 
   - build (e.g. linux-musl-x86_64-1.64). 
   - health - float 0 for broken, 1 for perfect
 - get back both the status of its build (active/deprecated/vulnerable)
 - and the current configurations that it can choose between, and their weights.
   e.g. `[{buildX, 1, details}, {buildY, 0.2 details}]`
 - it then chooses between the configurations and takes any action on its own.
 
 The API server would:
 - work with a time series store to store the client health reports
 - store the configurations and their current weights itself
 - adjust the desired weights itself, including rolling back if appropriate
 
New users would be directed to a redirection endpoint in the API server that
gave the most stable configuration automatically.

Timescales and volumes would auto-tune by default but could be hand-set.

# Use case 2 - Devops team running cloud infrastructure

The team would subscribe the API server to selected metrics from their existing
time-series datastore. The API server would use this to evaluate the health of
nodes x configurations. When rolling out a new configuration, a control loop
(as popularised by k8s) subscribed to the RaaS API would drive the stepping
to new nodes rather than either a) a human or b) a for-loop which only looks
at immediate errors

# Use case 3 - Application running with LaunchDarkly or https://github.com/Unleash/unleash or others

Connect RaaS to the application time-series DB - in particular to the SLO
series, and enable API integration between the two APIs. Let RaaS
scale the launch from 0 to 100 percent smoothly, watching your SLOs in realtime,
and holding the scale the moment things start to look spotty.

# Discovery with @developerjack

* Who will be writing against the API?

Case 2 and Case 3 could be quite different mental model wise - consider
presenting them differently "deployment" and "release", even if the
machinery isn't very different it may be more easily understood.

* Approaching the API design
a) domain definition
b) resource abstraction and serialisation
c) mental model / technical communication
d) REST etc

* Who are the customers - who will choose to run an instance and collaborate on building this,
  who will choose to use a hosted version?
