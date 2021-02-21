# RPC Tracing

Hyperledger Besu integrates with the [open-telemetry](https://open-telemetry.io)
project to integrate tracing reporting.

This allows to report all JSON-RPC traffic as traces.

To try out this example, start the Open Telemetry Collector and the Zipkin
service with:

`$> docker-compose up`

Start besu with:

`$> OTEL_RESOURCE_ATTRIBUTES="service.name=besu-dev"
OTEL_EXPORTER_OTLP_METRIC_INSECURE=true OTEL_EXPORTER_OTLP_SPAN_INSECURE=true
./gradlew run --args="--network=dev --rpc-http-enabled --metrics-enabled
--metrics-protocol=opentelemetry"`

Try interacting with the JSON-RPC API. Here is a simple example using cURL:

`$> curl -X POST --data
'{"jsonrpc":"2.0","method":"web3_clientVersion","params":[],"id":53}'
http://localhost:8545`

Open the Zipkin UI by browsing to http://localhost:9411/

You will be able to see the detail of your traces.

References:
* [OpenTelemetry Environment Variable
Specification](https://github.com/open-telemetry/opentelemetry-specification/blob/master/specification/sdk-environment-variables.md)


## Trace RPC API Notes

This document outlines major differences for `trace_replayBlockTransactions` 
compared to other implementations.

## `stateDiff` 

No major differences were observed in the `stateDiff` field.

## `trace`

Besu reports `gasUsed` after applying the effects of gas refunds.  Future
implementations of Besu might track gas refunds separately.

## `vmTrace`

### Returned Memory from Calls

In the `vmTrace` `ope.ex.mem` fields Besu only reports actual data returned
from a `RETURN` opcode.  Other implementations return the contents of the 
reserved output space for the call operations.  Note two major differences:

1. Besu reports `null` when a call operation ends because of a `STOP`,  `HALT`, 
   `REVERT`, running out of instructions, or any exceptional halts.
2. When a `RETURN` operation returns data of a different length than the space
   reserved by the call only the data passed to the `RETURN` operation is 
   reported.  Other implementations will include pre-existing memory data or 
   trim the returned data.

### Precompiled Contracts Calls

Besu reports only the actual cost of the precompiled contract call in the 
`cost` field. 

### Out of Gas 

Besu reports the operation that causes out fo gas exceptions, including 
calculated gas cost.  The operation is not executed so no `ex` values are 
reported.
