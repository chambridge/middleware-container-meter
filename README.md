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
export METERING_ROUTE_HOSTNAME=$(oc -n $METERING_NAMESPACE get routes metering -o json | jq -r '.status.ingress[].host')
```

Obtain your Metering reporting operator token:
```
export TOKEN=$(oc -n $METERING_NAMESPACE serviceaccounts get-token reporting-operator)
```

To get report data you need to supply the Report Name (middleware-cpu-usage-report) and the Format (csv or json):
```
curl -H "Authorization: Bearer $TOKEN" -k "https://$METERING_ROUTE_HOSTNAME/api/v1/reports/get?name=[Report Name]&namespace=$METERING_NAMESPACE&format=[Format]"
```

The `sw-cpu-usage-product-report`,`sw-cpu-usage-product-integrations-report` reports are hourly reports and may take 2+ hours to collect data.

If you want to run a report right now for past time frames you can create an on-demand report by altering the following YAML:

*Full product list:*
```
apiVersion: metering.openshift.io/v1
kind: Report
metadata:
  name: sw-cpu-usage-product-report-ondemand
  namespace: openshift-metering
  labels:
    subscription-watch: 'true'
spec:
  query: sw-cpu-usage-product
  reportingStart: '2020-05-01T00:00:00Z'
  reportingEnd: '2020-05-02T17:00:00Z'
  runImmediately: true
  schedule:
    period: hourly
```

*Filtered by integrations product list:*
```
apiVersion: metering.openshift.io/v1
kind: Report
metadata:
  name: sw-cpu-usage-product-integrations-report-ondemand
  namespace: openshift-metering
  labels:
    subscription-watch: 'true'
spec:
  query: sw-cpu-usage-product-integrations
  reportingStart: '2020-05-01T00:00:00Z'
  reportingEnd: '2020-05-02T17:00:00Z'
  runImmediately: true
  schedule:
    period: hourly
```

You will need to update the `reportingStart` and `reportingEnd` times appropriately.

The report will take a few minutes to run through the accessible data. Then you can run the command as described above to obtain the output.

*Full product list:*
curl -H "Authorization: Bearer $TOKEN" -k "https://$METERING_ROUTE_HOSTNAME/api/v1/reports/get?name=sw-cpu-usage-product-report-ondemand&namespace=$METERING_NAMESPACE&format=csv"
pod,namespace,product,used_cores,interval_start
broker-amq-1-hhjch,fuse,Red_Hat_Fuse,0.004952,2020-05-02 17:00:00 +0000 UTC
broker-amq-1-hhjch,fuse,Red_Hat_Fuse,0.005043,2020-05-02 16:00:00 +0000 UTC
camel-k-operator-674f659774-kbh2m,integrations,Red_Hat_Fuse,0.000450,2020-05-02 17:00:00 +0000 UTC
camel-k-operator-674f659774-kbh2m,integrations,Red_Hat_Fuse,0.000533,2020-05-02 16:00:00 +0000 UTC
gen-load-1-78b8565b76-rhbcf,integrations,Red_Hat_Fuse,0.194420,2020-05-02 17:00:00 +0000 UTC
gen-load-1-78b8565b76-rhbcf,integrations,Red_Hat_Fuse,0.279935,2020-05-02 16:00:00 +0000 UTC
gen-load-10-687c64f48d-rmp99,integrations,Red_Hat_Fuse,0.942859,2020-05-02 17:00:00 +0000 UTC
gen-load-10-687c64f48d-rmp99,integrations,Red_Hat_Fuse,0.946686,2020-05-02 16:00:00 +0000 UTC
gen-load-11-6b4856dc47-kwgzh,integrations,Red_Hat_Fuse,0.939554,2020-05-02 17:00:00 +0000 UTC
gen-load-11-6b4856dc47-kwgzh,integrations,Red_Hat_Fuse,0.944814,2020-05-02 16:00:00 +0000 UTC
gen-load-12-7467dfd4fd-7qcl4,integrations,Red_Hat_Fuse,0.169220,2020-05-02 17:00:00 +0000 UTC
gen-load-12-7467dfd4fd-7qcl4,integrations,Red_Hat_Fuse,0.288170,2020-05-02 16:00:00 +0000 UTC
gen-load-13-68d8b7fc7c-njmfc,integrations,Red_Hat_Fuse,0.173621,2020-05-02 17:00:00 +0000 UTC
gen-load-13-68d8b7fc7c-njmfc,integrations,Red_Hat_Fuse,0.283341,2020-05-02 16:00:00 +0000 UTC
gen-load-14-6894959576-4hqht,integrations,Red_Hat_Fuse,0.981594,2020-05-02 17:00:00 +0000 UTC
gen-load-14-6894959576-4hqht,integrations,Red_Hat_Fuse,0.982727,2020-05-02 16:00:00 +0000 UTC
gen-load-15-956c48967-7dmv7,integrations,Red_Hat_Fuse,0.188789,2020-05-02 17:00:00 +0000 UTC
gen-load-15-956c48967-7dmv7,integrations,Red_Hat_Fuse,0.271612,2020-05-02 16:00:00 +0000 UTC
gen-load-16-9c766dfbb-9sw9l,integrations,Red_Hat_Fuse,0.161736,2020-05-02 17:00:00 +0000 UTC
gen-load-16-9c766dfbb-9sw9l,integrations,Red_Hat_Fuse,0.291478,2020-05-02 16:00:00 +0000 UTC
gen-load-17-69b8b6fdcc-lsn4p,integrations,Red_Hat_Fuse,0.198019,2020-05-02 17:00:00 +0000 UTC
gen-load-17-69b8b6fdcc-lsn4p,integrations,Red_Hat_Fuse,0.270358,2020-05-02 16:00:00 +0000 UTC
gen-load-2-7f6475665f-f2jsx,integrations,Red_Hat_Fuse,0.224147,2020-05-02 17:00:00 +0000 UTC
gen-load-2-7f6475665f-f2jsx,integrations,Red_Hat_Fuse,0.275930,2020-05-02 16:00:00 +0000 UTC
gen-load-3-6d578d869d-dcfcd,integrations,Red_Hat_Fuse,0.198322,2020-05-02 17:00:00 +0000 UTC
gen-load-3-6d578d869d-dcfcd,integrations,Red_Hat_Fuse,0.274101,2020-05-02 16:00:00 +0000 UTC
gen-load-4-7dc6665657-7rwz2,integrations,Red_Hat_Fuse,0.191659,2020-05-02 17:00:00 +0000 UTC
gen-load-4-7dc6665657-7rwz2,integrations,Red_Hat_Fuse,0.284684,2020-05-02 16:00:00 +0000 UTC
gen-load-5-bdf9645dd-tt5wx,integrations,Red_Hat_Fuse,0.192700,2020-05-02 17:00:00 +0000 UTC
gen-load-5-bdf9645dd-tt5wx,integrations,Red_Hat_Fuse,0.289853,2020-05-02 16:00:00 +0000 UTC
gen-load-6-6d94d44979-ncjkf,integrations,Red_Hat_Fuse,0.167334,2020-05-02 17:00:00 +0000 UTC
gen-load-6-6d94d44979-ncjkf,integrations,Red_Hat_Fuse,0.273196,2020-05-02 16:00:00 +0000 UTC
gen-load-7-7dbd54f9d-9btmd,integrations,Red_Hat_Fuse,0.896190,2020-05-02 17:00:00 +0000 UTC
gen-load-7-7dbd54f9d-9btmd,integrations,Red_Hat_Fuse,0.899747,2020-05-02 16:00:00 +0000 UTC
gen-load-8-5888f9c7bf-xcpx8,integrations,Red_Hat_Fuse,0.943069,2020-05-02 17:00:00 +0000 UTC
gen-load-8-5888f9c7bf-xcpx8,integrations,Red_Hat_Fuse,0.944591,2020-05-02 16:00:00 +0000 UTC
gen-load-8566946f4-9pgkc,integrations,Red_Hat_Fuse,0.157401,2020-05-02 17:00:00 +0000 UTC
gen-load-8566946f4-9pgkc,integrations,Red_Hat_Fuse,0.278761,2020-05-02 16:00:00 +0000 UTC
gen-load-9-58fc67fcbd-bpngp,integrations,Red_Hat_Fuse,0.897682,2020-05-02 17:00:00 +0000 UTC
gen-load-9-58fc67fcbd-bpngp,integrations,Red_Hat_Fuse,0.897756,2020-05-02 16:00:00 +0000 UTC
syndesis-db-1-mbnp8,fuse,Red_Hat_Fuse,0.004577,2020-05-02 17:00:00 +0000 UTC
syndesis-db-1-mbnp8,fuse,Red_Hat_Fuse,0.013675,2020-05-02 16:00:00 +0000 UTC
syndesis-meta-1-d7lnj,fuse,Red_Hat_Fuse,0.000447,2020-05-02 17:00:00 +0000 UTC
syndesis-meta-1-d7lnj,fuse,Red_Hat_Fuse,0.000500,2020-05-02 16:00:00 +0000 UTC
syndesis-oauthproxy-1-vgxqk,fuse,Red_Hat_Fuse,0.000687,2020-05-02 17:00:00 +0000 UTC
syndesis-oauthproxy-1-vgxqk,fuse,Red_Hat_Fuse,0.000721,2020-05-02 16:00:00 +0000 UTC
syndesis-operator-7849d69b65-fdr8h,fuse,Red_Hat_Fuse,0.013318,2020-05-02 17:00:00 +0000 UTC
syndesis-operator-7849d69b65-fdr8h,fuse,Red_Hat_Fuse,0.015151,2020-05-02 16:00:00 +0000 UTC
syndesis-prometheus-1-9sp2h,fuse,Red_Hat_Fuse,0.002605,2020-05-02 17:00:00 +0000 UTC
syndesis-prometheus-1-9sp2h,fuse,Red_Hat_Fuse,0.002964,2020-05-02 16:00:00 +0000 UTC
syndesis-server-1-88rkd,fuse,Red_Hat_Fuse,0.019133,2020-05-02 17:00:00 +0000 UTC
syndesis-server-1-88rkd,fuse,Red_Hat_Fuse,0.026651,2020-05-02 16:00:00 +0000 UTC
syndesis-ui-1-6fkjs,fuse,Red_Hat_Fuse,0.000054,2020-05-02 17:00:00 +0000 UTC
syndesis-ui-1-6fkjs,fuse,Red_Hat_Fuse,0.000056,2020-05-02 16:00:00 +0000 UTC

*Filtered by integrations product list:*
```
curl -H "Authorization: Bearer $TOKEN" -k "https://$METERING_ROUTE_HOSTNAME/api/v1/reports/get?name=sw-cpu-usage-product-integrations-report-ondemand&namespace=$METERING_NAMESPACE&format=csv"
pod,namespace,product,used_cores,interval_start
gen-load-1-78b8565b76-rhbcf,integrations,Red_Hat_Fuse,0.194420,2020-05-02 17:00:00 +0000 UTC
gen-load-1-78b8565b76-rhbcf,integrations,Red_Hat_Fuse,0.279935,2020-05-02 16:00:00 +0000 UTC
gen-load-10-687c64f48d-rmp99,integrations,Red_Hat_Fuse,0.942859,2020-05-02 17:00:00 +0000 UTC
gen-load-10-687c64f48d-rmp99,integrations,Red_Hat_Fuse,0.946686,2020-05-02 16:00:00 +0000 UTC
gen-load-11-6b4856dc47-kwgzh,integrations,Red_Hat_Fuse,0.939554,2020-05-02 17:00:00 +0000 UTC
gen-load-11-6b4856dc47-kwgzh,integrations,Red_Hat_Fuse,0.944814,2020-05-02 16:00:00 +0000 UTC
gen-load-12-7467dfd4fd-7qcl4,integrations,Red_Hat_Fuse,0.169220,2020-05-02 17:00:00 +0000 UTC
gen-load-12-7467dfd4fd-7qcl4,integrations,Red_Hat_Fuse,0.288170,2020-05-02 16:00:00 +0000 UTC
gen-load-13-68d8b7fc7c-njmfc,integrations,Red_Hat_Fuse,0.173621,2020-05-02 17:00:00 +0000 UTC
gen-load-13-68d8b7fc7c-njmfc,integrations,Red_Hat_Fuse,0.283341,2020-05-02 16:00:00 +0000 UTC
gen-load-14-6894959576-4hqht,integrations,Red_Hat_Fuse,0.981594,2020-05-02 17:00:00 +0000 UTC
gen-load-14-6894959576-4hqht,integrations,Red_Hat_Fuse,0.982727,2020-05-02 16:00:00 +0000 UTC
gen-load-15-956c48967-7dmv7,integrations,Red_Hat_Fuse,0.188789,2020-05-02 17:00:00 +0000 UTC
gen-load-15-956c48967-7dmv7,integrations,Red_Hat_Fuse,0.271612,2020-05-02 16:00:00 +0000 UTC
gen-load-16-9c766dfbb-9sw9l,integrations,Red_Hat_Fuse,0.161736,2020-05-02 17:00:00 +0000 UTC
gen-load-16-9c766dfbb-9sw9l,integrations,Red_Hat_Fuse,0.291478,2020-05-02 16:00:00 +0000 UTC
gen-load-17-69b8b6fdcc-lsn4p,integrations,Red_Hat_Fuse,0.198019,2020-05-02 17:00:00 +0000 UTC
gen-load-17-69b8b6fdcc-lsn4p,integrations,Red_Hat_Fuse,0.270358,2020-05-02 16:00:00 +0000 UTC
gen-load-2-7f6475665f-f2jsx,integrations,Red_Hat_Fuse,0.224147,2020-05-02 17:00:00 +0000 UTC
gen-load-2-7f6475665f-f2jsx,integrations,Red_Hat_Fuse,0.275930,2020-05-02 16:00:00 +0000 UTC
gen-load-3-6d578d869d-dcfcd,integrations,Red_Hat_Fuse,0.198322,2020-05-02 17:00:00 +0000 UTC
gen-load-3-6d578d869d-dcfcd,integrations,Red_Hat_Fuse,0.274101,2020-05-02 16:00:00 +0000 UTC
gen-load-4-7dc6665657-7rwz2,integrations,Red_Hat_Fuse,0.191659,2020-05-02 17:00:00 +0000 UTC
gen-load-4-7dc6665657-7rwz2,integrations,Red_Hat_Fuse,0.284684,2020-05-02 16:00:00 +0000 UTC
gen-load-5-bdf9645dd-tt5wx,integrations,Red_Hat_Fuse,0.192700,2020-05-02 17:00:00 +0000 UTC
gen-load-5-bdf9645dd-tt5wx,integrations,Red_Hat_Fuse,0.289853,2020-05-02 16:00:00 +0000 UTC
gen-load-6-6d94d44979-ncjkf,integrations,Red_Hat_Fuse,0.167334,2020-05-02 17:00:00 +0000 UTC
gen-load-6-6d94d44979-ncjkf,integrations,Red_Hat_Fuse,0.273196,2020-05-02 16:00:00 +0000 UTC
gen-load-7-7dbd54f9d-9btmd,integrations,Red_Hat_Fuse,0.896190,2020-05-02 17:00:00 +0000 UTC
gen-load-7-7dbd54f9d-9btmd,integrations,Red_Hat_Fuse,0.899747,2020-05-02 16:00:00 +0000 UTC
gen-load-8-5888f9c7bf-xcpx8,integrations,Red_Hat_Fuse,0.943069,2020-05-02 17:00:00 +0000 UTC
gen-load-8-5888f9c7bf-xcpx8,integrations,Red_Hat_Fuse,0.944591,2020-05-02 16:00:00 +0000 UTC
gen-load-8566946f4-9pgkc,integrations,Red_Hat_Fuse,0.157401,2020-05-02 17:00:00 +0000 UTC
gen-load-8566946f4-9pgkc,integrations,Red_Hat_Fuse,0.278761,2020-05-02 16:00:00 +0000 UTC
gen-load-9-58fc67fcbd-bpngp,integrations,Red_Hat_Fuse,0.897682,2020-05-02 17:00:00 +0000 UTC
gen-load-9-58fc67fcbd-bpngp,integrations,Red_Hat_Fuse,0.897756,2020-05-02 16:00:00 +0000 UTC
```