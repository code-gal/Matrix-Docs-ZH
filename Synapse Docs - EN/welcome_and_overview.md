# Introduction

Welcome to the documentation repository for Synapse, a
[Matrix](https://matrix.org) homeserver implementation developed by Element.

## Installing and using Synapse

This documentation covers topics for **installation**, **configuration** and
**maintenance** of your Synapse process:

* Learn how to [install](Synapse%20Docs%20-%20EN/setup/installation.md) and
  [configure](Synapse%20Docs%20-%20EN/usage/configuration/config_documentation.md) your own instance, perhaps with [Single
  Sign-On](usage/configuration/user_authentication/index.html).

* See how to [upgrade](Synapse%20Docs%20-%20EN/upgrade.md) between Synapse versions.

* Administer your instance using the [Admin
  API](usage/administration/admin_api/index.html), installing [pluggable
  modules](modules/index.html), or by accessing the [manhole](Synapse%20Docs%20-%20EN/manhole.md).

* Learn how to [read log lines](Synapse%20Docs%20-%20EN/usage/administration/request_log.md), configure
  [logging](Synapse%20Docs%20-%20EN/usage/configuration/logging_sample_config.md) or set up [](Synapse%20Docs%20-%20EN/structured_logging.md).

* Scale Synapse through additional [worker processes](Synapse%20Docs%20-%20EN/workers.md).

* Set up [monitoring and metrics](Synapse%20Docs%20-%20EN/metrics-howto.md) to keep an eye on your
  Synapse instance's performance.

## Developing on Synapse

Contributions are welcome! Synapse is primarily written in
[Python](https://python.org). As a developer, you may be interested in the
following documentation:

* Read the [Contributing Guide](Synapse%20Docs%20-%20EN/development/contributing_guide.md). It is meant
  to walk new contributors through the process of developing and submitting a
  change to the Synapse codebase (which is [hosted on
  GitHub](https://github.com/element-hq/synapse)).

* Set up your [](Synapse%20Docs%20-%20EN/development/contributing_guide.md#2-what-do-i-need), then learn
  how to [lint](Synapse%20Docs%20-%20EN/development/contributing_guide.md#run-the-linters) and
  [test](Synapse%20Docs%20-%20EN/development/contributing_guide.md#8-test-test-test) your code.

* Look at [the issue tracker](https://github.com/element-hq/synapse/issues) for
  bugs to fix or features to add. If you're new, it may be best to start with
  those labeled [good first
  issue](https://github.com/element-hq/synapse/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22).

* Understand [how Synapse is
  built](development/internal_documentation/index.html), how to [](Synapse%20Docs%20-%20EN/development/database_schema.md), learn about
  [federation](Synapse%20Docs%20-%20EN/federate.md) and how to [](Synapse%20Docs%20-%20EN/federate.md#running-a-demo-federation-of-synapses) for development.

* We like to keep our `git` history clean. [Learn](Synapse%20Docs%20-%20EN/development/git.md) how to
  do so!

* And finally, contribute to this documentation! The source for which is
  [located here](https://github.com/element-hq/synapse/tree/develop/docs).

## Reporting a security vulnerability

If you've found a security issue in Synapse or any other Element project,
please report it to us in accordance with our [Security Disclosure
Policy](https://element.io/security/security-disclosure-policy). Thank you!
