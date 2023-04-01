# Embedding YACE in your application

It is possible to embed YACE into an external Go application. This mode might be useful to you if you would like to scrape on demand or run in a stateless manner.

The entrypoint to use YACE as a library is the `UpdateMetrics` func in [update.go](./pkg/exporter.go#L35) which requires,
- `config`: this is the struct representation of the configuration defined in [Top Level Configuration](#top-level-configuration)
- `registry`: any prometheus compatible registry where scraped AWS metrics will be written
- `metricsPerQuery`: controls the same behavior defined by the CLI flag `metrics-per-query`
- `labelsSnakeCase`: controls the same behavior defined by the CLI flag `labels-snake-case`
- `cloudwatchSemaphore`/`tagSemaphore`: adjusts the concurrency of requests as defined by [Requests concurrency](#requests-concurrency). Pass in a different length channel to adjust behavior
- `cache`
  - Any implementation of the [SessionCache Interface](./pkg/session/sessions.go#L41)
  - `session.NewSessionCache(config, <fips value>)` would be the default
  - `<fips value>` is defined by the `fips` CLI flag
- `observedMetricLabels`
  - Prometheus requires that all metrics exported with the same key have the same labels
  - This map will track all labels observed and ensure they are exported on all metrics with the same key in the provided `registry`
  - You should provide the same instance of this map if you intend to re-use the `registry` between calls
- `logger`
  - Any implementation of the [Logger Interface](./pkg/logging/logger.go#L13)
  - `logging.NewLogger(logrus.StandardLogger())` is an acceptable default

The update definition also includes an exported slice of [Metrics](./pkg/exporter.go#L18) which includes AWS API call metrics. These can be registered with the provided `registry` if you want them
included in the AWS scrape results. If you are using multiple instances of `registry` it might make more sense to register these metrics in the application using YACE as a library to better
track them over the lifetime of the application.
