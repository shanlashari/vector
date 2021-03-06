---
delivery_guarantee: "at_least_once"
description: "The Vector `aws_cloudwatch_metrics` sink streams `metric` events to Amazon Web Service's CloudWatch Metrics service via the `PutMetricData` API endpoint."
event_types: ["metric"]
issues_url: https://github.com/timberio/vector/issues?q=is%3Aopen+is%3Aissue+label%3A%22sink%3A+aws_cloudwatch_metrics%22
operating_systems: ["linux","macos","windows"]
sidebar_label: "aws_cloudwatch_metrics|[\"metric\"]"
source_url: https://github.com/timberio/vector/tree/master/src/sinks/aws_cloudwatch_metrics.rs
status: "beta"
title: "AWS Cloudwatch Metrics Sink"
unsupported_operating_systems: []
---

The Vector `aws_cloudwatch_metrics` sink [streams](#streaming) [`metric`][docs.data-model.metric] events to [Amazon Web Service's CloudWatch Metrics service][urls.aws_cw_metrics] via the [`PutMetricData` API endpoint](https://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/API_PutMetricData.html).

<!--
     THIS FILE IS AUTOGENERATED!

     To make changes please edit the template located at:

     website/docs/reference/sinks/aws_cloudwatch_metrics.md.erb
-->

## Configuration

import Tabs from '@theme/Tabs';

<Tabs
  block={true}
  defaultValue="common"
  values={[
    { label: 'Common', value: 'common', },
    { label: 'Advanced', value: 'advanced', },
  ]
}>

import TabItem from '@theme/TabItem';

<TabItem value="common">

import CodeHeader from '@site/src/components/CodeHeader';

<CodeHeader fileName="vector.toml" learnMoreUrl="/docs/setup/configuration/"/ >

```toml
[sinks.my_sink_id]
  namespace = "service" # example
  region = "us-east-1" # example
```

</TabItem>
<TabItem value="advanced">

<CodeHeader fileName="vector.toml" learnMoreUrl="/docs/setup/configuration/" />

```toml
[sinks.my_sink_id]
  # REQUIRED
  namespace = "service" # example
  region = "us-east-1" # example

  # OPTIONAL
  type = "aws_cloudwatch_metrics" # no default, must be: "aws_cloudwatch_metrics" (if supplied)
  inputs = ["my-source-id"] # example, no default
  healthcheck = true # default
```

</TabItem>

</Tabs>

## Options

import Fields from '@site/src/components/Fields';

import Field from '@site/src/components/Field';

<Fields filters={true}>


<Field
  common={false}
  defaultValue={true}
  enumValues={null}
  examples={[true,false]}
  name={"healthcheck"}
  path={null}
  relevantWhen={null}
  required={false}
  templateable={false}
  type={"bool"}
  unit={null}
  >

### healthcheck

Enables/disables the sink healthcheck upon start. See [Health Checks](#health-checks) for more info.


</Field>


<Field
  common={true}
  defaultValue={null}
  enumValues={null}
  examples={["service"]}
  name={"namespace"}
  path={null}
  relevantWhen={null}
  required={true}
  templateable={false}
  type={"string"}
  unit={null}
  >

### namespace

A [namespace](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch_concepts.html#Namespace) that will isolate different metrics from each other.


</Field>


<Field
  common={true}
  defaultValue={null}
  enumValues={null}
  examples={["us-east-1"]}
  name={"region"}
  path={null}
  relevantWhen={null}
  required={true}
  templateable={false}
  type={"string"}
  unit={null}
  >

### region

The [AWS region][urls.aws_cw_metrics_regions] of the target CloudWatch stream resides.


</Field>


</Fields>

## How It Works

### Environment Variables

Environment variables are supported through all of Vector's configuration.
Simply add `${MY_ENV_VAR}` in your Vector configuration file and the variable
will be replaced before being evaluated.

You can learn more in the [Environment Variables][docs.configuration#environment-variables]
section.

### Health Checks

Health checks ensure that the downstream service is accessible and ready to
accept data. This check is performed upon sink initialization.
If the health check fails an error will be logged and Vector will proceed to
start.

#### Require Health Checks

If you'd like to exit immediately upon a health check failure, you can
pass the `--require-healthy` flag:

```bash
vector --config /etc/vector/vector.toml --require-healthy
```

#### Disable Health Checks

If you'd like to disable health checks for this sink you can set the
`healthcheck` option to `false`.

### Metric Types

CloudWatch Metrics types are organized not by their semantics, but by storage properties:
- Statistic Sets
- Data Points

In Vector only the latter is used to allow lossless statistics calculations on CloudWatch side.

The following matrix outlines how Vector metric types are mapped into CloudWatch metrics types.

| Vector Metrics | CloudWatch Metrics |
|----------------|--------------------|
| Counter        | Data Point         |
| Gauge          | Data Point         |
| Gauge Delta [1]| Data Point         |
| Histogram      | Data Point         |
| Set            | N/A                |

1. Gauge values are persisted between flushes. On Vector start up each gauge is assumed to have
zero (0.0) value, that can be updated explicitly by the consequent absolute (not delta) gauge
observation, or by delta increments/decrements. Delta gauges are considered an advanced feature
useful in distributed setting, however it should be used with care.

### Streaming

The `aws_cloudwatch_metrics` sink streams data on a real-time
event-by-event basis. It does not batch data.


[docs.configuration#environment-variables]: /docs/setup/configuration/#environment-variables
[docs.data-model.metric]: /docs/about/data-model/metric/
[urls.aws_cw_metrics]: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/working_with_metrics.html
[urls.aws_cw_metrics_regions]: https://docs.aws.amazon.com/general/latest/gr/rande.html#cw_region
