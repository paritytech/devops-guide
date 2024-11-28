# Guide: Deploying an RPC node

This guide demonstrates the deployment of an RPC node via Helm chart in Kubernetes.


## Preparations

Parity maintains a helm GitHub repo @ [https://github.com/paritytech/helm-charts](https://github.com/paritytech/helm-charts) - Inside this repo is the [node](https://github.com/paritytech/helm-charts/tree/main/charts/node) chart which can be used for deploying your Substrate/Polkadot binary.

All variables are documented clearly in the [README.md](https://github.com/paritytech/helm-charts/blob/main/charts/node/README.md) and there’s an example [local-rococo](https://github.com/paritytech/helm-charts/tree/main/charts/node/examples/local-rococo) which you can start working from.


#### Helm Deployment


Binary images are available at Dockerhub. Parity maintains both [Polkadot](https://hub.docker.com/r/parity/polkadot) and [Polkadot-Parachain](https://hub.docker.com/r/parity/polkadot-parachain) images from the [polkadot-sdk](https://github.com/paritytech/polkadot-sdk) repo.

When running an RPC node, a sidecar called [ws-health-exporter](https://github.com/paritytech/scripts/tree/master/dockerfiles/ws-health-exporter) is good to have to allow a readiness endpoint for Kubernetes.


```bash
helm repo add parity https://paritytech.github.io/helm-charts/
helm -n NAMESPACE upgrade --install RELEASE_NAME -f FILE_NAME.yml parity/node
```

#### Relaychain Helm Values

```
image:
  repository: parity/polkadot
  tag: v1.14.0
node:
  replicas: 1
  chain: westend
  role: full
  chainData:
    chainPath: westend2
    volumeSize: 600Gi
    storageClass: ssd-csi
    database: paritydb
    pruning: archive
  chainKeystore:
    mountInMemory:
      enabled: true
  flags:
    - "--rpc-max-connections 5000"
  perNodeServices:
    relayP2pService:
      enabled: true
      type: NodePort
    setPublicAddressToExternalIp:
      enabled: true
  persistGeneratedNodeKey: true
  enableOffchainIndexing: true
  serviceMonitor:
    enabled: true
  tracing:
    enabled: false
  resources:
    requests:
      cpu: 500m
      memory: 2Gi
    limits:
      cpu: 4
      memory: 7Gi

# RPC Endpoint
ingress:
  enabled: false
  annotations:
    kubernetes.io/ingress.class: TODO
    external-dns.alpha.kubernetes.io/target: TODO
    cert-manager.io/cluster-issuer: TODO
  host: parachain.example.com
  tls:
    - secretName: parachain.example.com
      hosts:
        - parachain.example.com

extraContainers:
- name: ws-health-exporter
  image: docker.io/paritytech/ws-health-exporter:0a2e6e9b-20230412
  env:
  - name: WSHE_NODE_RPC_URLS
    value: "ws://127.0.0.1:9944"
  - name: WSHE_NODE_MAX_UNSYNCHRONIZED_BLOCK_DRIFT
    value: "4"
  - name: WSHE_NODE_MIN_PEERS
    value: "2"
  resources:
    requests:
      cpu: 10m
      memory: 32M
    limits:
      cpu: 100m
      memory: 64M
  readinessProbe:
    httpGet:
      path: /health/readiness
      port: 8001
```

#### Parachains Helm Values

```
  image:
    repository: parity/polkadot-parachain
    tag: 1.14.0
  node:
    chain: bridge-hub-westend
    command: polkadot-parachain
    replicas: 1
    role: full
    chainData:
      database: paritydb
      pruning: archive
      volumeSize: 50Gi
      storageClass: ssd-csi
    chainKeystore:
      mountInMemory:
        enabled: true
    isParachain: true
    collatorRelayChain:
      chain: westend
      chainData:
        database: paritydb
        pruning: 1000
        storageClass: ssd-csi
        volumeSize: 150Gi
        chainPath: westend2
      chainKeystore:
        mountInMemory:
          enabled: true
    flags:
      - "--rpc-max-connections 1000"
    perNodeServices:
      relayP2pService:
        enabled: true
        type: NodePort
      paraP2pService:
        enabled: true
        type: NodePort
      setPublicAddressToExternalIp:
        enabled: true
    resources:
      requests:
        cpu: 1
        memory: 2Gi
      limits:
        memory: 4Gi
        
# RPC Endpoint
ingress:
  enabled: false
  annotations:
    kubernetes.io/ingress.class: TODO
    external-dns.alpha.kubernetes.io/target: TODO
    cert-manager.io/cluster-issuer: TODO
  host: parachain.example.com
  tls:
    - secretName: parachain.example.com
      hosts:
        - parachain.example.com
```

### Best Practices

* The advantage of running in Kubernetes is the number of replicas can easily be added behind an ingress. 
* A backup is ideally used due to easily scale up new RPC nodes
* Enabling serviceMonitor to enable monitoring of the RPC nodes