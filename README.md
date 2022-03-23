## Firehose Kubernetes Manifests

Those here are a edited Kubernetes manifest used by StreamingFast on its production cluster serving Ethereum Mainnet,
Ethereum Ropsten, BSC Mainnet, Polygon Mainnet, NEAR Mainnet and NEAR Testnet.

It's a trimmed down edited version of our files. Some elements have been replaced so they need to be changed
be you. The elements to change are listed below

**Important** The manifest you will find it are provided as a reference to create your own version of it. Indeed,
they will probably not apply directly (although it should be quite easy to make them work mostly directly). Indeed,
some elements specifically `PersistentVolumeClaim` have some dependencies on the cluster to be configured correctly,
for example `common/yaml/StorageClass.gcpssd-lazy.yaml`.

### Namespaces

Each chain specific set (e.g. [ethereum/mainnet](./ethereum/mainnet)) has an hard-coded namespace that follows the
pattern `<protocol>-<network>` where usually the protocol is a shorthen version of the full one.

Change the `namespace` value with a search/replace if you want to use something else.

### Replacements

- `<sfeth_container_image>` Should be the Docker image you will use to run `sfeth` binary (the Docker image should contain `sfeth` as well as the Geth instrumented binary for the chain).
- `<oneBlocksStoragePath>` Should points to a location where single block are saved for further processing by the merger (which cleans them after creating a bundle).
- `<mergedBlocksStoragePath>` Should points to location where merged bundles of 100 blocks will be stored permanently.
- `<blocksIndexesStoragePath>` Should points to a location where blocks indexes are populated and stored permanently.
- `<irreversibilityIndexesStoragePath>` hould points to a location where irreversibility indexes (to determine if a block is final or not rapidly) are populated and stored permanently.

For the storage paths, how you gonna split depends a bit on what you are going to do deploy. When using a cloud provider directly,
we suggest using a single bucket and in each bucket, element are namespaced using the deployed namespace then a folder, so for
`eth-mainnet` namespace, storage paths could look like:

- `<oneBlocksStoragePath>` => `gs://acme-blocks-us/eth-mainnet/one-blocks`
- `<mergedBlocksStoragePath>` => `gs://acme-blocks-us/eth-mainnet/merged-blocks`
- `<blocksIndexesStoragePath>` => `gs://acme-blocks-us/eth-mainnet/indexes/fitlers`
- `<irreversibilityIndexesStoragePath>` => `gs://acme-blocks-us/eth-mainnet/indexes/irr`

Replacement can be made against any DSN accepted by our storage abstraction library
https://github.com/streamingfast/dstore.

- AWS S3 - `s3://[bucket]/path?region=us-east-1` or `s3://<ip>:<port>/<bucket>?region=none&insecure=true` (any compatible storage solution with S3 interface works)
- Google Storage `gs://[bucket]/path`
- Azure Blob Storage `az://[account].[container]/path`
- Local file systems `file:///` prefix (including virtual of fused-based)

Authentication must be configured per pod to access the storage location, this is is highly dependant on the picked
solution. We use `ServiceAccount` and GKE mapping to manages permissions of our individual pods.

### Versions

We version most of the resources you will see relate to a particular network. The actual current version is not really important
and could all be renamed to `v1`. We versionned them like so we are able to later deploy a brand new set in the same namespace
without collision.