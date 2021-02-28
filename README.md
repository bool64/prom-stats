# Prometheus Adapter for Contextualized Stats Tracker

This library provides context-driven stats tracker implementation.

[![Build Status](https://github.com/bool64/prom-stats/workflows/test-unit/badge.svg)](https://github.com/bool64/prom-stats/actions?query=branch%3Amaster+workflow%3Atest-unit)
[![Coverage Status](https://codecov.io/gh/bool64/prom-stats/branch/master/graph/badge.svg)](https://codecov.io/gh/bool64/prom-stats)
[![GoDevDoc](https://img.shields.io/badge/dev-doc-00ADD8?logo=go)](https://pkg.go.dev/github.com/bool64/prom-stats)
[![time tracker](https://wakatime.com/badge/github/bool64/prom-stats.svg)](https://wakatime.com/badge/github/bool64/prom-stats)
![Code lines](https://sloc.xyz/github/bool64/prom-stats/?category=code)
![Comments](https://sloc.xyz/github/bool64/prom-stats/?category=comments)

## Features

* Context-driven labels control.
* Zero allocation.
* A simple interface with variadic number of key-value pairs for labels.
* Easily mockable interface free from 3rd party dependencies.

## Example

```go
// Bring your own Prometheus registry.
registry := prometheus.NewRegistry()
tr := prom.Tracker{
    Registry: registry,
}

// Add custom Prometheus configuration where necessary.
tr.DeclareHistogram("my_latency_seconds", prometheus.HistogramOpts{
    Buckets: []float64{1e-4, 1e-3, 1e-2, 1e-1, 1, 10, 100},
})

ctx := context.Background()

// Add labels to context.
ctx = stats.AddKeysAndValues(ctx, "ctx-label", "ctx-value0")

// Override label values.
ctx = stats.AddKeysAndValues(ctx, "ctx-label", "ctx-value1")

// Collect stats with last mile labels.
tr.Add(ctx, "my_count", 1,
    "some-label", "some-value",
)

tr.Add(ctx, "my_latency_seconds", 1.23)

tr.Set(ctx, "temperature", 33.3)
```

## Performance

Sample benchmark result with go1.16.
```
goos: darwin
goarch: amd64
pkg: github.com/bool64/stats/prom-stats
cpu: Intel(R) Core(TM) i7-8559U CPU @ 2.70GHz
```
```
name             time/op
Tracker_Add-8    635ns ± 3%
RawPrometheus-8  472ns ± 2%

name             alloc/op
Tracker_Add-8    0.00B     
RawPrometheus-8   336B ± 0%

name             allocs/op
Tracker_Add-8     0.00     
RawPrometheus-8   2.00 ± 0%
```

## Caveats

Prometheus client does not support metrics with same name and different label sets. 
If you add a label to context, make sure you have it in all cases, at least with an empty value `""`.

## Versioning

This project adheres to [Semantic Versioning](https://semver.org/#semantic-versioning-200).

Before version `1.0.0`, breaking changes are tagged with `MINOR` bump, features and fixes are tagged with `PATCH` bump.
After version `1.0.0`, breaking changes are tagged with `MAJOR` bump.
