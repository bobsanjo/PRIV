# Production Operation

**Author:** Akshay Pundle

## Overview

This explainer reviews the features available in the Protected Audience services and defines the key components for production readiness, so that these services are operable at the desired 
reliability rate. Our goal is to build a set of mitigations and plans so that when outages occur, they can be gracefully and quickly addressed, rather than attempting to prevent outages or build “perfect” servers.

## Design principles

These pillars are presented in alphabetical order, rather than order of importance:

* [Change management](#change_management)
* [Developer experience](#developer-experience)
* [Incident management](#incident-management)
* [Reliability](#reliability)
* [Resources](#resources)

### Change management
 
We will offer management tools to address service outages, which typically occur when there is a system change.

*   **Permissions and roles:** to control who can make changes and when.
*   **Server release process:** covers how a new release candidate is created and how the candidate is tested to certify that it is ready and has no regressions. This also includes the ability to run "canary" jobs to test out a new release at a small scale before it goes to full production and understand how rollbacks will work.
*   **Lifecycle management:** releases will not be supported forever, so we need to work out what commitments there will be and how old releases will be deprecated. This includes the handling of breaking versus non-breaking changes.

### Developer experience

We will create a positive experience for developers by responding to open issues and offering a well-built service with the ability to contribute code.

*   **Repository structure:** the open source repositories for the servers should be consistent and integrated with tooling (for example, for code indexing and search, continuous integration, continuous build, and so on).
*   **Community contribution policies:** these should be consistent between servers, especially where they share shared repositories for common components.
*   **Debugging tools:** these are especially tricky to build for Protected Audience services because of the deliberate limitations of what the servers expose about their running state. We plan to publish separate explainers (for example, [Debugging protected audience API services][1]) on this topic.

### Incident management

We will offer playbooks for service operators and define emergency procedures.

*   **Support structures:** when there is an outage, how can server operators report it for triage and remediation.
*   **Playbooks:** for server operators there should be a well-defined set of common alerts and how/when to mitigate them.
*   **Pre-planned mitigations:** a set of responses that are expected to be useful. For example, to roll back to a previous server version or to restore corrupt data.

### Reliability

We will create a system that meets reliability requirements.

*   **Reliability ranges:** the server operators are responsible for deciding the reliability targets that they are aiming for (for example, uptime). The servers should be able to meet a range of different reliabilities. This is important because often extremely high reliability comes with significant costs.
*   **Monitoring and alerting:** the ability to detect when servers fail, or stray from reliability targets, is necessary. Without this information server operators cannot tell if the servers are performing correctly. See [Monitoring protected audience API services][2] for details.
*   **Risk management:** past outages will be tracked and documented together with changes that come out of them. This is used to both track system health over time and to make sure that known bugs are corrected.
*   **Graceful degradation:** when servers are overwhelmed they should be able to shed traffic in a predictable way (while also alerting operators that they are in this state, as above).
*   **Scale testing:** it is useful to know where the limits of the servers are. For example, how much traffic can one single instance handle before it is overwhelmed.

### Resources

We will provide information as to what resources the server will use and how best to set it up to use it efficiently.

*   **Sensible default deployment configurations:** to ensure that common mistakes are avoided.
*   **Ongoing benchmarking:** to prevent performance regressions.
*   **Cost prediction:** to estimate the Cloud costs required to run certain workloads.


[1]: https://github.com/privacysandbox/fledge-docs/blob/main/debugging_protected_audience_api_services.md
[2]: https://github.com/privacysandbox/fledge-docs/blob/main/monitoring_protected_audience_api_services.md
