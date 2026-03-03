Lab 6 – REST vs gRPC Performance Comparison


PERFORMANCE RESULTS

| Method | Local (localhost) | Same-Zone (US → US) | Different Region (US → Frankfurt) |
|--------|-------------------|---------------------|------------------------------------|
| REST add (1000) | 0.79 ms | 3.21 ms | 305.92 ms |
| gRPC add (1000) | 0.10 ms | 0.79 ms | 146.98 ms |
| REST rawimg (100) | 1.55 ms | 8.94 ms | 1196.53 ms |
| gRPC rawimg (100) | 0.16 ms | 1.54 ms | 174.38 ms |
| REST dotproduct (1000) | 0.92 ms | 3.68 ms | 315.15 ms |
| gRPC dotproduct (1000) | 0.12 ms | 0.89 ms | 149.83 ms |
| REST jsonimg (100) | 7.74 ms | 27.32 ms | 1389.44 ms |
| gRPC jsonimg (100) | 4.03 ms | 12.56 ms | 186.28 ms |
| PING (Avg RTT) | 0.17 ms | 0.34 ms | 144.48 ms |

Footnote: add and dotproduct were run with 1000 reps; rawimg and jsonimg were run with 100 reps.


ANALYSIS 


The results clearly show that network latency dominates performance once requests leave the local machine. In the Local and Same-Zone tests, both REST and gRPC complete operations in under a few milliseconds, and the difference between them is small. However, in the Different Region test (US to Frankfurt), the round-trip latency increases to ~144 ms (as measured by ping), and this latency becomes the primary cost driver for every request.

For lightweight operations such as add and dotproduct, gRPC performs much closer to the measured ping RTT (e.g., 146.98 ms vs 144.48 ms), indicating minimal protocol overhead. In contrast, REST is roughly twice as slow (≈305 ms), which is consistent with REST establishing a new TCP connection for each request, adding connection setup and teardown overhead on top of the network latency. gRPC, using HTTP/2, maintains a single persistent TCP connection and multiplexes requests over it, significantly reducing per-call overhead.

The performance difference becomes even more pronounced for image-based services. REST rawImage and jsonImage over the different region take over 1 second per operation, while gRPC remains below 200 ms. This highlights two factors: (1) the inefficiency of repeatedly transmitting large payloads over high-latency links with new HTTP/1.1 connections, and (2) the additional overhead of JSON encoding (especially base64 for jsonImage). Protobuf serialization in gRPC is more compact and computationally efficient than JSON, which explains the substantially lower latency.