+++
title = "Explore"
type = "docs"
[menu.docs]
name = "Explore"
identifier = "explore"
parent = "features"
weight = 5
+++

# Introduction

One of the major new features of Grafana 6.0 is the new query-focused Explore workflow for troubleshooting and/or for data exploration.

Grafana's dashboard UI is all about building dashboards for visualization. Explore strips away all the dashboard and panel options so that you can focus on the query. Iterate until you have a working query and then think about building a dashboard.

For infrastructure monitoring and incident response, you no longer need to switch to other tools to debug what went wrong. Explore allows you to dig deeper into your metrics and logs to find the cause. Grafana's new logging datasource, [Loki](https://github.com/grafana/loki) is tightly integrated into Explore and allows you to correlate metrics and logs by viewing them side-by-side. This creates a new debugging workflow where you can:

1. Receive an alert
2. Drill down and examine metrics
3. Drill down again and search logs related to the metric and time interval (and in the future, distributed traces).

If you just want to explore your data and do not want to create a dashboard then Explore makes this much easier. Explore will show the results as both a graph and a table enabling you to see trends in the data and more detail at the same time (if the datasource supports both graph and table data).

## Turning the Explore Feature On

Explore will be officially released in Grafana 6.0. It is however already in the latest nightly builds of Grafana and can be turned using a feature flag in the config file. Restart Grafana after making the config file change.

```ini
[explore]
# Enable the Explore section
enabled = true
```

Or if using docker:

```bash
docker pull grafana/grafana:master
docker run --name grafana -p 3000:3000 -e "GF_EXPLORE_ENABLED=true" grafana/grafana:master
```

## How to Start Exploring

There is a new Explore icon on the menu bar to the left. This opens a new empty Explore tab.

{{< docs-imagebox img="/img/docs/v60/explore_menu.png" class="docs-image--no-shadow" caption="Screenshot of the new Explore Icon" >}}

If you want to start with an existing query in a panel then choose the Explore option from the Panel menu. This opens an Explore tab with the query from the panel and allows you to tweak or iterate in the query outside of your dashboard.

{{< docs-imagebox img="/img/docs/v60/explore_panel_menu.png" class="docs-image--no-shadow" caption="Screenshot of the new Explore option in the panel menu" >}}

Choose your datasource in the dropdown in the top left. Prometheus has a custom Explore implementation, the other datasources (for now) use their standard query editor.

The query field is where you can write your query and explore your data. There are three buttons beside the query field, a clear button (X), an add query button (+) and the remove query button (-). Just like the normal query editor, you can add and remove multiple queries.

## Split and Compare

The Split feature is an easy way to compare graphs and tables side-by-side or to look at related data together on one page. Click the split button to duplicate the current query and split the page into two side-by-side queries. It is possible to select another datasource for the new query which for example, allows you to compare the same query for two different servers or to compare the staging environment to the production environment.

{{< docs-imagebox img="/img/docs/v60/explore_split.png" class="docs-image--no-shadow" caption="Screenshot of the new Explore option in the panel menu" >}}

You can close the newly created query by clicking on the Close Split button.

## Prometheus-specific Features

The first version of Explore features a custom querying experience for Prometheus. When a query is executed, it actually executes two queries, a normal Prometheus query for the graph and an Instant Query for the table. An Instant Query returns the last value for each time series which shows a good summary of the data shown in the graph.

### Metrics Explorer

On the left-hand side of the query field is a `Metrics` button, clicking on this opens the Metric Explorer. This shows a hierarchical menu with metrics grouped by their prefix. For example, all the Alert Manager metrics will be grouped under the `alertmanager` prefix. This is a good starting point if you just want to explore which metrics are available.

{{< docs-imagebox img="/img/docs/v60/explore_metric_explorer.png" class="docs-image--no-shadow" caption="Screenshot of the new Explore option in the panel menu" >}}

### Query Field

The Query field supports autocomplete for metric names, function and works mostly the same way as the standard Prometheus query editor. Press the enter key to execute a query.

The autocomplete menu can be trigger by pressing Ctrl + Space. The Autocomplete menu contains a new History section with a list of recently executed queries.

Suggestions can appear under the query field - click on them to update your query with the suggested change.

- For counters (monotonously increasing metrics), a rate function will be suggested.
- For buckets, a histogram function will be suggested.
- For recording rules, possible to expand the rules.

### Table Filters

Click on the filter button <span title="Filter for label" class="logs-label__icon fa fa-search-plus"></span> in a labels column in the Table panel to add filters to the query expression. This works with multiple queries too - the filter will be added for all the queries.

## Logs Integration - Loki-specific Features

For Grafana 6.0, the first log integration is for the new open source log aggregation system from Grafana Labs - [Loki](https://github.com/grafana/loki). Loki is designed to be very cost effective, as it does not index the contents of the logs, but rather a set of labels for each log stream. The logs from Loki are queried in a similar way to querying with label selectors in Prometheus. It uses labels to group log streams which can be made to match up with your Prometheus labels. Read more about Loki [here]((https://github.com/grafana/loki)) or the Grafana Labs hosted variant: [Grafana Cloud Logs](https://grafana.com/loki).

If you switch from a Prometheus query to a logs query (you can do a split first to have your metrics and logs side by side) then it will keep the labels from your query that exist in the logs and use those to query the log streams. For example, the following Prometheus query:

`grafana_alerting_active_alerts{job="grafana"}`

after switching to the Logs datasource, the query changes to:

`{job="grafana"}`

This will return a chunk of logs in the selected time range that can be grepped/text searched.

### Log Queries

A log query consists of two parts: **log stream selector**, and a **search expression**. For performance reasons you need to start by choosing a log stream by selecting a log label.

The Logs Explorer (the `Log labels` button) next to the query field shows a list of labels of available log streams. An alternative way to write a query is to use the query field's autocomplete - you start by typing a left curly brace `{` and the autocomplete menu will suggest a list of labels. Press the `enter` key to execute the query.

Once the result is returned, the log panel shows a list of log rows and a bar chart where the x-axis shows the time and the y-axis shows the frequency/count.

{{< docs-imagebox img="/img/docs/v60/explore_loki.png" class="docs-image--no-shadow" caption="Explore Loki Log Streams" >}}

#### Log Stream Selector

For the label part of the query expression, wrap it in curly braces `{}` and then use the key value syntax for selecting labels. Multiple label expressions are separated by a comma:

`{app="mysql",name="mysql-backup"}`

The following label matching operators are currently supported:

- `=` exactly equal.
- `!=` not equal.
- `=~` regex-match.
- `!~` do not regex-match.

Examples:

- `{name=~"mysql.+"}`
- `{name!~"mysql.+"}`

The [same rules that apply for Prometheus Label Selectors](https://prometheus.io/docs/prometheus/latest/querying/basics/#instant-vector-selectors) apply for Loki Log Stream Selectors.

Another way to add a label selector, is in the table section, clicking on the **Filter** button beside a label will add the label to the query expression. This even works for multiple queries and will the label selector to each query.

#### Search Expression

After writing the Log Stream Selector, you can filter the results further by writing a search expression. The search expression can be just text or a regex expression.

Example queries:

- `{job="mysql"} error`
- `{name="kafka"} tsdb-ops.*io:2003`
- `{instance=~"kafka-[23]",name="kafka"} kafka.server:type=ReplicaManager`

### Deduping

Log data can be very repetitive and Explore can help by hiding duplicate log lines. There are a few different deduplication algorithms that you can use:

- `exact` Exact matches are done on the whole line, except for date fields.
- `numbers` Matches on the line after stripping out numbers (durations, IP addresses etc.).
- `signature` The most aggressive deduping - strips all letters and numbers, and matches on the remaining whitespace and punctuation.

### Timestamp, Local time and Labels

There are some other check boxes under the logging graph apart from the Deduping options.

- Timestamp: shows/hides the Timestamp column
- Local time: shows/hides the Local time column
- Labels: shows/hides the label filters column