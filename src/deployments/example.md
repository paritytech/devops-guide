# Example Deployments

## Development Testnet

Below is an example configuration of a private or public testnet. Where one provider is used and hosts are spread over two locations/zones within that provider.

| Function      | Provider(s) | Count | Locations |
| ------------- | ----------- | ----- | --------- |
| Bootnodes     | 1           | 2     | 2         |
| Validator     | 1           | 4     | 2         |
| RPC           | 1           | 2     | 2         |
| Load Balancer | 1           | 1     | 2         |

## Parachain Network

Below is a minimal configuration example for a production parachain. This example uses two providers with four locations, which should all be well physically distributed. Adding more locations within the existing providers and a third provider is recommended for a highly available configuration.

| Function      | Provider(s) | Count | Locations |
| ------------- | ----------- | ----- | --------- |
| Bootnodes     | 2           | 4     | 4         |
| Validator     | 2           | 8     | 4         |
| RPC           | 2           | 8     | 4         |
| Load Balancer | 2           | 1     | 2         |
