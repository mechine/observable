# prometheus http api

[官网](https://prometheus.io/docs/prometheus/latest/querying/api/#querying-metada)

### ## 获取元数据

#### Querying metric metadata

It returns metadata about metrics currently scrapped from targets. However, it does not provide any target information. This is considered **experimental** and might change in the future.

```
GET /api/v1/metadata
```

URL query parameters:

* `limit=<number>`: Maximum number of metrics to return.
* `metric=<string>`: A metric name to filter metadata for. All metric metadata is retrieved if left empty.

#### Finding series by label matchers <a href="#finding-series-by-label-matchers" id="finding-series-by-label-matchers"></a>

The following endpoint returns the list of time series that match a certain label set.

```
GET /api/v1/series
POST /api/v1/series
```

URL query parameters:

* `match[]=<series_selector>`: Repeated series selector argument that selects the series to return. At least one `match[]` argument must be provided.
* `start=<rfc3339 | unix_timestamp>`: Start timestamp.
* `end=<rfc3339 | unix_timestamp>`: End timestamp.

You can URL-encode these parameters directly in the request body by using the `POST` method and `Content-Type: application/x-www-form-urlencoded` header. This is useful when specifying a large or dynamic number of series selectors that may breach server-side URL character limits.

The `data` section of the query result consists of a list of objects that contain the label name/value pairs which identify each series.

#### Getting label names <a href="#getting-label-names" id="getting-label-names"></a>

The following endpoint returns a list of label names:

```
GET /api/v1/labels
POST /api/v1/labels
```

URL query parameters:

* `start=<rfc3339 | unix_timestamp>`: Start timestamp. Optional.
* `end=<rfc3339 | unix_timestamp>`: End timestamp. Optional.
* `match[]=<series_selector>`: Repeated series selector argument that selects the series from which to read the label names. Optional.

The `data` section of the JSON response is a list of string label names.

#### Querying label values <a href="#querying-label-values" id="querying-label-values"></a>

The following endpoint returns a list of label values for a provided label name:

```
GET /api/v1/label/<label_name>/values
```

URL query parameters:

* `start=<rfc3339 | unix_timestamp>`: Start timestamp. Optional.
* `end=<rfc3339 | unix_timestamp>`: End timestamp. Optional.
* `match[]=<series_selector>`: Repeated series selector argument that selects the series from which to read the label values. Optional.

The `data` section of the JSON response is a list of string label values.

### ## 查询数据

#### Instant queries <a href="#instant-queries" id="instant-queries"></a>

The following endpoint evaluates an instant query at a single point in time:

```
GET /api/v1/query
POST /api/v1/query
```

URL query parameters:

* `query=<string>`: Prometheus expression query string.
* `time=<rfc3339 | unix_timestamp>`: Evaluation timestamp. Optional.
* `timeout=<duration>`: Evaluation timeout. Optional. Defaults to and is capped by the value of the `-query.timeout` flag.

The current server time is used if the `time` parameter is omitted.

You can URL-encode these parameters directly in the request body by using the `POST` method and `Content-Type: application/x-www-form-urlencoded` header. This is useful when specifying a large query that may breach server-side URL character limits.

#### Range queries <a href="#range-queries" id="range-queries"></a>

The following endpoint evaluates an expression query over a range of time:

```
GET /api/v1/query_range
POST /api/v1/query_range
```

URL query parameters:

* `query=<string>`: Prometheus expression query string.
* `start=<rfc3339 | unix_timestamp>`: Start timestamp, inclusive.
* `end=<rfc3339 | unix_timestamp>`: End timestamp, inclusive.
* `step=<duration | float>`: Query resolution step width in `duration` format or float number of seconds.
* `timeout=<duration>`: Evaluation timeout. Optional. Defaults to and is capped by the value of the `-query.timeout` flag.

You can URL-encode these parameters directly in the request body by using the `POST` method and `Content-Type: application/x-www-form-urlencoded` header. This is useful when specifying a large query that may breach server-side URL character limits.

