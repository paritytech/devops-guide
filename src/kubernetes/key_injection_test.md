# Polkadot Key Injection

## Chart Settings:

Ensure `allowUnsafeRpcMethods: true` is set in the helm chart values.

## Example Key Injection Values:

Add example keys, in this case ``//Testing/babe`` (sr25519) and ``//Testing//gran`` (ed25519) on chain `polkadot-dev`:

```
  keys: 
    - type: "gran"
      scheme: "ed25519"
      seed: "//Testing//gran"
    - type: "babe"
      scheme: "sr25519"
      seed: "//Testing"
      extraDerivation: //babe
```

## Get public keys:

 ``` 
 $ polkadot key inspect //Testing//babe --network polkadot
  Secret Key URI `//Testing//babe` is account:
  Network ID:        polkadot 
 Secret seed:       0x35993b5ca38b5909a26dbc15b736dede3bdf233ce63380f6ae62cb0ca096483b
  Public key (hex):  0x28c63dd7bb87238daf363a539da87d3c67d66a199e24f7751432e4f2363e8147
  Account ID:        0x28c63dd7bb87238daf363a539da87d3c67d66a199e24f7751432e4f2363e8147
  Public key (SS58): 1vToiVGY8q2kuG5XaN4JXrydgmkvoNMPpbwb9fPU8uQuRM5
  SS58 Address:      1vToiVGY8q2kuG5XaN4JXrydgmkvoNMPpbwb9fPU8uQuRM5
  ```

```
  $ polkadot key inspect //Testing//gran --network polkadot --scheme ed25519
   Secret Key URI `//Testing//gran` is account:
  Network ID:        polkadot 
  Secret seed:       0xbfc22ae1e901dbd25057747b315481cf8a470261ea81892a9aa31be1391a0dcb
  Public key (hex):  0x7819b7ddc959b749a5da604c3a2d7d3e0097760be603d596f19d116d6207ad38
  Account ID:        0x7819b7ddc959b749a5da604c3a2d7d3e0097760be603d596f19d116d6207ad38
  Public key (SS58): 13iUPPkgWezjEpxFyfCKczLkrsBtJfwMWYhjDRwvtCKp1v3s
  SS58 Address:      13iUPPkgWezjEpxFyfCKczLkrsBtJfwMWYhjDRwvtCKp1v3s
  ```

  - Babe key: 0x28c63dd7bb87238daf363a539da87d3c67d66a199e24f7751432e4f2363e8147
 - Grandpa Key: 0x7819b7ddc959b749a5da604c3a2d7d3e0097760be603d596f19d116d6207ad38

## Verify keys exist

```
 $ yarn run:api rpc.author.hasKey 0x28c63dd7bb87238daf363a539da87d3c67d66a199e24f7751432e4f2363e8147 babe
   { 
      "hasKey": true
   }
```

```
   $ yarn run:api rpc.author.hasKey 0x7819b7ddc959b749a5da604c3a2d7d3e0097760be603d596f19d116d6207ad38 gran
   {
     "hasKey": true
   }
```

