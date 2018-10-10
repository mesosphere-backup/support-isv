# Overview
DC/OS Spinnaker is an automated service that makes it easy to deploy and manage Spinnaker on DC/OS.

The Spinnaker service is a micro service composition, a good overview on the micro services involved can be found [here](https://www.spinnaker.io/reference/architecture/).

## General Spinnaker Triage Steps

1. Collect a DC/OS log bundle
2. Collect Scheduler logs - stderr & stdout.
3. Collect Node Logs - stderr & stdout.
Pro Tip: ​dcos task log --all <task-id> > all.log

## General framework troubleshooting
https://docs.mesosphere.com/services/ops-guide/troubleshooting/

## Spinnaker Specific

All Spinnaker services except `redis` and `deck` are springboot applications which write logs to stdout. In the service config section of the Spinnaker service debug level log output can be enabled for the springboot based services. When doing config changes for one of the spinnaker services it stdout log should be observed.
