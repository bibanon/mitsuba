# Architecture

Welcome to the document describing Mitsuba's architecture.
If you want to familiarize yourself with the code base, then this is a correct place!
After you will be familiar with our code base, please follow [contributing guide](../../CONTRIBUTING.md)

### `static`, `src/templates`, `src/web`

These files are responsible for Mitsuba's frontend and API.
As for templating engine we use [Handlebars](https://handlebarsjs.com)

### `src/archiver`

Core archiver of Mitsuba.

### `src/db.rs`, `src/models.rs`, `sqlx-data.json`

Interactions with Mitsuba's database.
There's also `migrations` folder in the root of the project, which is responsible for Mitsuba's database schema.

### `src/http.rs`

Mitsuba's HTTP client, we are using reqwest library.

### `src/metrics.rs`

Metrics for [Grafana](https://grafana.com)

### `src/object_storage.rs`

This file is responsible for interactions with Amazon S3, we utilize `rust-s3` crate.

### `src/main.rs`

Mitsuba's CLI administration.

### `src/util.rs`

Helpful utilities for Mitsuba.
