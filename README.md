
# RoundRobinInvokeHTTP - Apache NiFi Custom Processor

## Overview
`RoundRobinInvokeHTTP` is a custom Apache NiFi processor designed to distribute HTTP requests across multiple backend endpoints **in a round-robin fashion**. 

It extends NiFi's built-in `InvokeHTTP` functionality but adds **client-side load balancing and failover** logic. If a request to a backend endpoint fails (due to connection errors or HTTP 5xx responses), the processor automatically retries the request against the next endpoint in the configured list.

This processor is particularly **useful in environments where the downstream system does not sit behind a traditional load balancer**. It provides application-level load balancing directly within your NiFi flow.

---

## Key Features
- 🔄 **Round-robin URL selection**: Cycles through a list of backend URLs for every request.  
- ♻️ **Automatic failover**: Skips to the next URL if a request fails or receives an error status code.  
- 🔐 **Full InvokeHTTP feature set**: Maintains all capabilities of the stock `InvokeHTTP` processor (dynamic headers, expression language, SSL/TLS, etc.).  
- ⚡ **Cluster-safe**: Works in NiFi clusters; each node performs independent round-robin load distribution.  
- 🧩 **Drop-in replacement**: Can be used wherever you currently use `InvokeHTTP`.  

---

## Use Case
If your downstream REST APIs or microservices are **not fronted by a load balancer** (e.g., HAProxy, NGINX, AWS ALB), this processor ensures traffic is **evenly distributed** across multiple hosts and can **gracefully handle failures** without requiring external infrastructure changes.

Example scenarios:
- Connecting to multiple internal microservice instances directly.  
- Distributing data pushes to multiple APIs across datacenters.  
- Testing performance of multiple backend servers without deploying a reverse proxy.  

---

## Project Structure

```
nifi-alb-bundle/              <- Maven parent project (packaging type: pom)
├── nifi-alb-processors/      <- Java source code for RoundRobinInvokeHTTP
│   ├── src/main/java/
│   │   └── org/apache/nifi/processors/alb/RoundRobinInvokeHTTP.java
│   └── pom.xml               <- Builds the processor JAR
├── nifi-alb-nar/             <- NAR (NiFi Archive) packaging
│   ├── src/main/resources/
│   └── pom.xml               <- Builds the .nar for deployment
└── pom.xml                   <- Parent POM
```

---

## Build Instructions

You’ll need:
- Java 8 or later
- Apache Maven 3.5+
- Apache NiFi 1.12.0+ (for runtime)

To build the processor and generate the deployable `.nar` file:

```bash
# Clone this repo
git clone https://github.com/<your-username>/RoundRobinInvokeHTTP.git
cd RoundRobinInvokeHTTP

# Build using Maven
mvn clean install
```

After a successful build, the NAR file will be generated at:

```
nifi-alb-nar/target/nifi-alb-nar-1.0.0.nar
```

---

## Deployment Instructions

1. **Copy the NAR to NiFi**:
   ```bash
   cp nifi-alb-nar/target/nifi-alb-nar-1.0.0.nar $NIFI_HOME/lib/
   ```

2. **Restart NiFi**:
   ```bash
   $NIFI_HOME/bin/nifi.sh restart
   ```

3. **Verify in UI**:
   - Go to the NiFi UI, click **Add Processor**, and search for:
     ```
     RoundRobinInvokeHTTP
     ```
   - The processor will appear alongside built-in NiFi processors.

---

## Configuration

| Property                | Description                                                          |
|-------------------------|----------------------------------------------------------------------|
| **Backend URLs**        | Comma-separated list of backend URLs. Requests are rotated in order. |
| **Max Retries**         | Number of times to retry before failing the FlowFile.                |
| **Retry on 5xx**        | Whether to fail over if HTTP 5xx responses are received.             |
| **All InvokeHTTP props**| Supports all existing properties from `InvokeHTTP`.                 |

Example `Backend URLs` value:
```
http://server1.example.com:8080/api,http://server2.example.com:8080/api
```

---

## Example Flow

```
[GenerateFlowFile] -> [RoundRobinInvokeHTTP] -> [PutFile]
```

- Each FlowFile is sent to alternating URLs from the list.  
- Failover logic ensures retrying the next URL if one is down.  

---

## Why Not Just Use a Load Balancer?
In some environments:
- External load balancers are **not allowed** (strict network/security policies).  
- The system architecture is **not yet load-balanced** but needs even distribution.  
- Developers/testers need **client-side control** over failover and retries.

This processor is designed to solve these problems without introducing external infrastructure.

---

## Contributing
Contributions are welcome! Please fork the repository, create a feature branch, and submit a Pull Request. Issues and enhancements can be submitted via GitHub Issues.

---

## License
Apache License 2.0. See [LICENSE](LICENSE) for details.
