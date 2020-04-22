# middleware-container-meter
Operator Metering object templates for capturing and calculating middleware usage on OpenShift.

## Dependencies
You will need the following dependencies to utilize this projects capabilities:

- OpenShift 4.2+
- Prometheus
- Operator Metering


## Installing Metering Objects
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

Clean up Operator Metering Objects:
```
oc delete -f ./templates/
```

## Working with Metering

Set your Metering project:
```
export METERING_NAMESPACE=openshift-metering
```

Obtain your Metering route hostname:
```
METERING_ROUTE_HOSTNAME=$(oc -n $METERING_NAMESPACE get routes metering -o json | jq -r '.status.ingress[].host')
```

Obtain your Metering reporting operator token:
```
TOKEN=$(oc -n $METERING_NAMESPACE serviceaccounts get-token reporting-operator)
```

To get report data you need to supply the Report Name (middleware-cpu-usage-report) and the Format (csv or json):
```
curl -H "Authorization: Bearer $TOKEN" -k "https://$METERING_ROUTE_HOSTNAME/api/v1/reports/get?name=[Report Name]&namespace=$METERING_NAMESPACE&format=[Format]"
```

The `middleware-cpu-usage-report` report is an hourly report and may take 2+ hours to collect data.

If you want to run a report right now for past time frames you can create an on-demand report by altering the following YAML:
```
apiVersion: metering.openshift.io/v1
kind: Report
metadata:
  name: middleware-cpu-usage-report-ondemand
  namespace: metering
  labels:
    middleware-container-meter: 'true'
spec:
  query: middleware-cpu-usage
  reportingStart: '2020-04-21T00:00:00Z'
  reportingEnd: '2020-04-22T17:00:00Z'
  runImmediately: true
  schedule:
    period: hourly
```

You will need to update the `reportingStart` and `reportingEnd` times appropriately.

The report will take a few minutes to run through the accessible data. Then you can run the command as described above to obtain the output.
```
curl -H "Authorization: Bearer $TOKEN" -k "https://$METERING_ROUTE_HOSTNAME/api/v1/reports/get?name=middleware-cpu-usage-report-ondemand&namespace=$METERING_NAMESPACE&format=csv"
pod,namespace,product,used_cores,interval_start
broker-amq-1-7m22v,fuse,Red_Hat_Fuse,0.004346,2020-04-22 17:00:00 +0000 UTC
broker-amq-1-7m22v,fuse,Red_Hat_Fuse,0.024322,2020-04-22 16:00:00 +0000 UTC
jaeger-operator-57f645966b-7psg8,fuse,Red_Hat_Fuse,0.000026,2020-04-22 17:00:00 +0000 UTC
jaeger-operator-57f645966b-7psg8,fuse,Red_Hat_Fuse,0.000090,2020-04-22 16:00:00 +0000 UTC
syndesis-db-1-wlb5j,fuse,Red_Hat_Fuse,0.002922,2020-04-22 17:00:00 +0000 UTC
syndesis-db-1-wlb5j,fuse,Red_Hat_Fuse,0.002948,2020-04-22 16:00:00 +0000 UTC
syndesis-oauthproxy-1-z4m4t,fuse,Red_Hat_Fuse,0.000799,2020-04-22 17:00:00 +0000 UTC
syndesis-oauthproxy-1-z4m4t,fuse,Red_Hat_Fuse,0.000849,2020-04-22 16:00:00 +0000 UTC
syndesis-operator-1-znws4,fuse,Red_Hat_Fuse,0.000228,2020-04-22 16:00:00 +0000 UTC
syndesis-operator-1-znws4,fuse,Red_Hat_Fuse,0.012461,2020-04-22 17:00:00 +0000 UTC
syndesis-operator-1-znws4,fuse,Red_Hat_Fuse,0.013752,2020-04-22 16:00:00 +0000 UTC
syndesis-prometheus-1-cpvwv,fuse,Red_Hat_Fuse,0.001715,2020-04-22 17:00:00 +0000 UTC
syndesis-prometheus-1-cpvwv,fuse,Red_Hat_Fuse,0.002238,2020-04-22 16:00:00 +0000 UTC
```