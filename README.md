# middleware-container-meter
Operator Metering object templates for capturing and calculating middleware usage on OpenShift.

## Dependencies
You will need the following dependencies to utilize this projects capabilities:

- OpenShift 3.11+
- Prometheus
- Operator Metering


## Installing Operator Metering Objects
The following are the steps needed to create the necessary operator metering objects (datasource, query, report).

Login to the OpenShift cluster:
```
oc login
```

Switch to the metering project:
```
oc project metering
```

Create the Operator Metering Objects:
```
oc create -f ./templates/
```