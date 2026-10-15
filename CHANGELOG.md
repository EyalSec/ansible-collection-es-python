# Changelog

## 1.0.0 (2026-10-15)

First release.

- Role `eyalsec.es_python.es_python`: installs es-python for one user on each
  host, with one API key for the whole fleet or a one-time install secret for
  a single machine. Idempotent; the credential travels on standard input and
  never appears in a log.
