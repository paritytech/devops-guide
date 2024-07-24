# RPC Overview

An RPC node is a server that allows external applications to interact with the Polkadot chain. This interaction is facilitated through a set of standardized APIs which enable the execution of various operations and retrieval of blockchain data. Here are some key points about an RPC node in Polkadot:

Functions of an RPC Node:
*Querying Blockchain Data*: RPC nodes can be used to fetch information from the blockchain, such as block details, transaction histories, account balances, and other state information.
*Submitting Transactions*: External applications can use RPC nodes to broadcast transactions to the network. This includes sending tokens, executing smart contracts, or any other transaction type supported by Polkadot.
*Subscription to Events*: RPC nodes can provide real-time updates on blockchain events, such as new blocks being added, changes in account balances, or other relevant blockchain events.
*Chain State and Metadata*: RPC nodes allow applications to query the current state of the blockchain, as well as metadata about the runtime environment, including the version of the Polkadot runtime, the list of available RPC methods, and more.

### Type of RPC Nodes

There are two common types of RPC nodes: *archive* nodes and *pruned* nodes, which differ in data storage and access.

- **Archive Nodes**:
  - Store the entire blockchain history, providing full state access for tasks requiring historical data, such as analytics and audits.
  - Require substantial disk space.
  - **Important Flag**: `--state-pruning archive --blocks-pruning archive` (Ensures the node retains all historical data)

- **Pruned Nodes**:
  - Retain only recent data, offering limited historical access but using significantly less storage.
  - Suitable for most day-to-day operations and network security.
  - **Important Flag**: `--state-pruning 1000` (Only keeps last 1000 of finalized blocks)


While archive nodes are essential for applications needing comprehensive historical data, pruned nodes are sufficient for wallets, validators, and other typical uses.


### Important Flags for Running an RPC Node

1. **`--rpc-methods`**: Defines which RPC methods are available.
   - Example: `--rpc-methods=Unsafe`
   - Options: `Auto`, `Safe`, `Unsafe`

2. **`--rpc-cors`**: Specifies the Cross-Origin Resource Sharing (CORS) settings.
   - Example: `--rpc-cors=all` (Allow all origins)
   - Example: `--rpc-cors=http://localhost:3000` (Allow specific origin)

3. **`--rpc-port`**: Sets the port on which the RPC server listens.
   - Example: `--rpc-port=9933`

4. **`--rpc-external`**: Allows RPC connections from external sources.
   - Example: `--rpc-external`

5. **`--rpc-rate-limit`**: RPC rate limiting (calls/minute) for each connection..
   - Example: `--rpc-rate-limit=100`

6. **`--unsafe-rpc-external`**: Listen to all RPC interfaces.
   - Example: `--unsafe-rpc-external`


### Securing the WS Port

Securing the WebSocket (WS) port is essential to prevent unauthorized access and potential security threats such as data breaches, man-in-the-middle attacks, and denial-of-service attacks. It is also necessary to access the node from the Polkadot-js UI.

#### Tools for Securing WebSocket Connections

A list of most common tools available that supports securing websocket:

1. **NGINX**: A popular web server and reverse proxy that can be configured to handle WebSocket connections securely.
   - **Documentation**: [NGINX WebSocket Support](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)

2. **HAProxy**: A high-performance TCP/HTTP load balancer that can manage WebSocket connections securely.
   - **Documentation**: [HAProxy WebSocket Support](http://www.haproxy.org/#docs)

3. **Traefik**: A modern reverse proxy and load balancer that supports WebSocket connections and integrates well with various security features.
   - **Documentation**: [Traefik WebSocket Support](https://doc.traefik.io/traefik/providers/websocket/)

4. **Cloudflare**: Provides WebSocket support along with security features like DDoS protection and WAF.
   - **Documentation**: [Cloudflare WebSocket Support](https://developers.cloudflare.com/using-cloudflare/website-performance/websockets/)

5. **AWS Application Load Balancer (ALB)**: Supports WebSocket connections and integrates with AWS WAF for additional security.
   - **Documentation**: [AWS ALB WebSocket Support](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html)

6. **Caddy**: A web server with automatic HTTPS that can also handle WebSocket connections securely.
   - **Documentation**: [Caddy WebSocket Support](https://caddyserver.com/docs/reverse-proxy#websocket)

### How-to Guides

This section provides a deployment guide for an RPC node via Ansible and Kubernetes.
