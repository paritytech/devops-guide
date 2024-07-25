# Subkey
`subkey` is a cli tool for generating substrate based account. It can also be used for signing and verifying messages, but in this doc we only cover account creation and node key (libp2p) generation.

# Installation
In order to install the tool, please reference the substrate documentation: https://docs.substrate.io/reference/command-line-tools/subkey/#installation

# Generate Substrate Based Accounts
As mentioned earlier, subkey can be used to generate accounts for different substrate chains and parachains, including polkadot, kosama, moonbeam, etc.

By default, `substrate generate` command generate an account for substrate network. By adding `--network <network-name>`, one could generate accounts for other networks. 

```
> subkey generate
Secret phrase:       claim year obvious artist also float royal vague industry tackle husband prize
  Network ID:        substrate
  Secret seed:       0x4548be7dc2fa89b3e5600f3933d69ffac4d06e0ac6222ccb9a17fe3cb49ebd9d
  Public key (hex):  0x10938f06b4265ac2bef65bc5d3e3d8ce20b12bc07d2d45ca1463ba2f1a47a078
  Account ID:        0x10938f06b4265ac2bef65bc5d3e3d8ce20b12bc07d2d45ca1463ba2f1a47a078
  Public key (SS58): 5CSSTUxjWVAWSLDxigjsyAF2PXLCznf1FSUhbDooTwaaWQM1
  SS58 Address:      5CSSTUxjWVAWSLDxigjsyAF2PXLCznf1FSUhbDooTwaaWQM1

> substrate generate --network polkadot
Secret phrase:       spell subject sleep celery cigar term neck tube blossom inflict session baby
  Network ID:        polkadot
  Secret seed:       0x50893f76f50855f9ca891dfa55ef64f51ad744f3d643f3f8046ef13b6b249baa
  Public key (hex):  0x847ace070b6bb193659379f1c64e8f3dda721ba545bfaee6fbb717df84ef247f
  Account ID:        0x847ace070b6bb193659379f1c64e8f3dda721ba545bfaee6fbb717df84ef247f
  Public key (SS58): 13zhoUzf3rqj5uzgFAuquf7DrgofKrUHBjUemarXS78G4Bon
  SS58 Address:      13zhoUzf3rqj5uzgFAuquf7DrgofKrUHBjUemarXS78G4Bon
```

- The seed phrase must be saved and stored in a safe place to be able to recover your account information (public and private keys).
- The `SS58 Address` is the account address on different network (aka wallet address).
- The `Network ID` field indicate the network name that this account belongs to.

# Retrieve Accounts From the Seed Phrase
`subkey inspect` subcommand can be used to retrieve the account information from the seed phrase:

```
> subkey inspect "claim year obvious artist also float royal vague industry tackle husband prize"
Secret phrase:       claim year obvious artist also float royal vague industry tackle husband prize
  Network ID:        substrate
  Secret seed:       0x4548be7dc2fa89b3e5600f3933d69ffac4d06e0ac6222ccb9a17fe3cb49ebd9d
  Public key (hex):  0x10938f06b4265ac2bef65bc5d3e3d8ce20b12bc07d2d45ca1463ba2f1a47a078
  Account ID:        0x10938f06b4265ac2bef65bc5d3e3d8ce20b12bc07d2d45ca1463ba2f1a47a078
  Public key (SS58): 5CSSTUxjWVAWSLDxigjsyAF2PXLCznf1FSUhbDooTwaaWQM1
  SS58 Address:      5CSSTUxjWVAWSLDxigjsyAF2PXLCznf1FSUhbDooTwaaWQM1

> subkey inspect --network polkadot "spell subject sleep celery cigar term neck tube blossom inflict session baby"
Secret phrase:       spell subject sleep celery cigar term neck tube blossom inflict session baby
  Network ID:        polkadot
  Secret seed:       0x50893f76f50855f9ca891dfa55ef64f51ad744f3d643f3f8046ef13b6b249baa
  Public key (hex):  0x847ace070b6bb193659379f1c64e8f3dda721ba545bfaee6fbb717df84ef247f
  Account ID:        0x847ace070b6bb193659379f1c64e8f3dda721ba545bfaee6fbb717df84ef247f
  Public key (SS58): 13zhoUzf3rqj5uzgFAuquf7DrgofKrUHBjUemarXS78G4Bon
  SS58 Address:      13zhoUzf3rqj5uzgFAuquf7DrgofKrUHBjUemarXS78G4Bon
```

# Derived Accounts (Child Accounts)
The `subkey` programs supports creation of sub accounts which are derived from the exact same seed phrase, so that there'd be no need for additional seed phrases for new accounts. One use case is when you want to create multiple accounts for different validators/collators, without needing to create and store many seed phrases.

To derive a child key pair, you add two slashes (//), a derivation path, and an index after the secret phrase associated with its parent key. Because you derive child key pairs and addresses from keys that have been previously generated, you use the subkey inspect command. For example:

```
> subkey inspect "claim year obvious artist also float royal vague industry tackle husband prize//validator-1//beefy"
  Network ID:        substrate
  Secret seed:       0xb60b7b15155936780cd83063e169bc8402276b253a1d75fa678cc547dd530eb2
  Public key (hex):  0x269c50d4065b4bf58839df4cfc16a11b367b80fbf00d21a96da863d8659e720a
  Account ID:        0x269c50d4065b4bf58839df4cfc16a11b367b80fbf00d21a96da863d8659e720a
  Public key (SS58): 5CwL7TsLnqQKakTfweh5aZ27fe9GHySP86YwbHtLUK3p649g
  SS58 Address:      5CwL7TsLnqQKakTfweh5aZ27fe9GHySP86YwbHtLUK3p649g


> subkey inspect "claim year obvious artist also float royal vague industry tackle husband prize//validator-1//babe"
  Network ID:        substrate
  Secret seed:       0xb31ee2df80a708917b62683579164f625be935a4c3760b24cd3f60f630e0ee2e
  Public key (hex):  0x00493534063762b76cb4ffad8336e57af312c02a6155a871600040de6f1d936a
  Account ID:        0x00493534063762b76cb4ffad8336e57af312c02a6155a871600040de6f1d936a
  Public key (SS58): 5C55c1YcFWQV6XdFCQVpG52K1SjgZqWKDvg7uNe9PFX4Hc9x
  SS58 Address:      5C55c1YcFWQV6XdFCQVpG52K1SjgZqWKDvg7uNe9PFX4Hc9x
```

**Important Note:** Both seed phrase and derivation paths need to be stored to be able to arrive at the key for an address. Only use custom derivation paths if you are comfortable with your knowledge of this topic.

# Generate Node Keys (`libp2p` Keys)
Use the `subkey generate-node-key` command to generate random public and private keys for peer-to-peer (`libp2p`) communication between Substrate nodes. The public key is the peer identifier that is used in chain specification files or as a command-line argument to identify a node participating in the blockchain network. In most cases, you run this command with a command-line option to save the private key to a file.

To generate a random key pair for peer-to-peer communication and save the secret key in a file, run a command similar to the following:
```
> subkey generate-node-key --file ../generated-node-key
12D3KooWMUsb8yizAaQiFRPtuqJTK9c7CSektH5YFV3MGxHGJbnV
```
This command displays the peer identifier for the node key in the terminal and the private key is saved in the generated-node-key file. In this example, the saved key in the parent directory instead of the current working directory.
```
12D3KooWHALHfL7dDBiGTt4JTEAvCbDWts8zHwvcPvJXDF9fxue7
```

Use the `subkey inspect-node-key` command to display the peer identifier for the node that corresponds with the node key in the specified file name. Before using this command, you should have previously used the `subkey generate-node-key` command and saved the key to a file.
```
> subkey inspect-node-key --file <file-name>
```
