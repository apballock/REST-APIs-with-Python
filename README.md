# REST APIs with Python

A three-notebook sprint covering the full lifecycle of API integration in Python — from foundational concepts to authenticated requests, data transformation, and a live interactive dashboard combining two independent data sources. Every chart in this project reflects real-time data pulled from production APIs at the moment of execution.

---

## Notebook 1 — REST API Fundamentals

**File:** [`notebook_01_rest_fundamentals.ipynb`](notebook_01_rest_fundamentals.ipynb)

REST (Representational State Transfer) is the architectural standard behind virtually every modern data integration — from weather apps to Power BI connectors to database pipelines. Understanding how HTTP methods and status codes work is a prerequisite for any role that touches external data.

This notebook establishes the core mental model: an API is a contract between two systems. The client sends a request specifying a resource and an action (GET to retrieve, POST to create, PUT to replace, PATCH to update, DELETE to remove), and the server responds with a status code indicating the outcome — 2xx for success, 4xx for client errors, 5xx for server failures.

The practical work used the Open-Meteo API, a production-grade meteorological data source, to make the first live GET requests. Rather than concatenating parameters directly into the URL, requests were built using the `params` argument — the standard approach in production code, which handles character encoding automatically and keeps URLs readable.

Error handling was implemented from the start using `raise_for_status()` combined with `try/except` blocks scoped to specific exception types. This pattern — distinguishing between HTTP errors, connection failures, and timeouts — is what separates scripts that work in development from code that holds up in production.

---

## Notebook 2 — Authentication & Data Processing

**File:** [`notebook_02_authentication_and_data.ipynb`](notebook_02_authentication_and_data.ipynb)

Most production APIs require authentication. This notebook covers API key management using OpenWeatherMap, with a deliberate focus on credential security: keys are stored in a `.env` file, loaded at runtime via `python-dotenv`, and excluded from version control through `.gitignore`. Hardcoding credentials into source code is one of the most common and costly mistakes in data engineering — this setup eliminates that risk by design.

The OpenWeatherMap response is deeply nested JSON, which required structured parsing to extract only the fields relevant to analysis: temperature, feels-like, humidity, pressure, weather description, and wind speed. This parsing logic was encapsulated in a dedicated function, keeping the data collection loop clean and the transformation logic testable independently.

With a working fetch-and-parse function in place, the notebook iterated over six European cities — Barcelona, Madrid, Lisbon, Paris, Berlin, and Amsterdam — loading each response into a list and assembling the full dataset into a pandas DataFrame in a single pass. The result was a clean, analysis-ready table of live weather data with one row per city.

A summary analysis identified the hottest and coldest cities, the most humid location, and the strongest wind at the moment of execution — grounding the technical work in a concrete analytical output.

---

## Notebook 3 — European Weather Dashboard

**File:** [`notebook_03_dashboard.ipynb`](notebook_03_dashboard.ipynb)

The final project combines two independent APIs — OpenWeatherMap (authenticated) and Open-Meteo (open) — into a single analytical dataset, then visualizes it through an interactive Plotly dashboard.

Merging data from multiple sources on a shared key (`city`) is a pattern that appears constantly in real data work: consolidating vendor feeds, joining internal and external datasets, cross-validating figures from different providers. Here it served a concrete analytical purpose: comparing temperature readings from two independent meteorological sources side by side, which is a lightweight form of data validation. Both APIs returned consistent values across all six cities, confirming the reliability of each source independently.

The dashboard covers four dimensions simultaneously — temperature, humidity, wind speed, and atmospheric pressure — in a 2×2 subplot layout. A separate scatter plot maps the relationship between temperature and humidity across cities, with bubble size encoding wind speed, giving a multi-variable view in a single chart. The final visualization is a live choropleth map of Europe with city markers sized and colored by temperature, where hovering over any point surfaces the full weather profile for that location.

The distinction between the two APIs used in this project reflects a real architectural decision: Open-Meteo is suitable for high-volume, unauthenticated use cases where rate limits and key management would add unnecessary friction. OpenWeatherMap offers richer data fields and commercial SLA guarantees, at the cost of credential management. Choosing between them depends on the use case, not on which is technically superior.

---

## Key Concepts

**Authentication and credential security** are non-negotiable in production. API keys stored in source code are a security incident waiting to happen — `.env` files combined with `.gitignore` enforce separation between code and secrets from the first commit.

**Structured error handling** is what makes API integrations reliable. Catching specific exception types (`HTTPError`, `ConnectionError`, `Timeout`) allows the code to respond appropriately to different failure modes rather than crashing or silently returning bad data.

**Combining multiple sources** on a shared key is one of the most common patterns in data engineering. The merge in Notebook 3 is simple by design — but the underlying logic of joining, validating, and reconciling data from independent providers scales directly to more complex production workflows.

**Plotly for stakeholder-facing output** — static charts belong in exploratory analysis. When the deliverable is something another person will interact with, interactivity is not a nice-to-have; it is what allows a stakeholder to answer their own follow-up questions without requesting a new export.
