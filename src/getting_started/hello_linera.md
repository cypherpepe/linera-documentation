# Hello, Linera

This section is about running a local development network, then compiling and
deploying your first application from scratch.

By the end of this section, you'll have a
[microchain](../core_concepts/microchains.md) running locally and a working
application that can be queried using GraphQL.

## Starting the Local Network

The first step is to start your local development network. A development network
consists of a numbers of [validators](../advanced_topics/validators.md), each of
which consist of an ingress proxy (aka a "load balancer") and a number of
workers (aka "physical shards").

To start a local network, run the following command:

```bash
linera net up
```

This will start a validator with the default number of shards and create a
temporary directory storing the entire network state.

## Storing the Wallet

The `linera net up` command also prints two statements which can be used to
configure your terminal with a new wallet for this local network:

```bash
Linera net directory available at: /var/folders/3d/406tbklx3zx2p3_hzzpfqdbc0000gn/T/.tmpvJ6lJI
To configure your Linera client for this network, run:

export LINERA_WALLET="/var/folders/3d/406tbklx3zx2p3_hzzpfqdbc0000gn/T/.tmpvJ6lJI/wallet_0.json"
export LINERA_STORAGE="rocksdb:/var/folders/3d/406tbklx3zx2p3_hzzpfqdbc0000gn/T/.tmpvJ6lJI/linera.db"
```

The wallet is only valid for the lifetime of a single network. Every time your
local network is restarted, the wallet needs to be reconfigured.

## Interacting with the Network

The main way of interacting with the network and deploying applications is using
the `linera` client.

To check that the network is working, you can synchronize the balance for your
[default chain](../core_concepts/wallet.md) with the rest of the network.

```bash
linera sync-balance
```

You should see an output of `10`.

## Building an Example Application

Applications running on Linera are [Wasm](https://webassembly.org/) bytecode.
Each validator and client has a built-in Wasm virtual machine (VM) which can
execute bytecode.

Let's build the `counter` application from the `examples/` subdirectory:

```bash
cd examples/counter && cargo build --release
```

> Note: This will automatically build Wasm, not native code.

## Creating the Application

You can publish the bytecode and create an application using it on your local
network using the `linera` client's `publish-and-create` command and provide:

1. The location of the contract bytecode
2. The location of the service bytecode
3. The JSON encoded initialization arguments

```bash
linera publish-and-create \
  ../target/wasm32-unknown-unknown/release/counter_{contract,service}.wasm \
  --json-argument "42"
```

Congratulations! You've published your first application on Linera!

## Querying your Application

Now let's query your application to get the current counter value. To do that,
we need to use the client running in
[_service_ mode](../core_concepts/node_service.md). This will expose a bunch of
APIs locally which we can use to interact with applications on the network.

```bash
linera service
```

<!-- TODO: add graphiql image here -->

Navigate to `http://localhost:8080` in your browser to access the GraphiQL, the
[GraphQL](https://graphql.org) IDE. We'll look at this in more detail in a
[later section](../core_concepts/wallet.md#graphql); for now, list the
applications deployed on your default chain e476… by running:

```gql
query {
  applications(
    chainId: "e476187f6ddfeb9d588c7b45d3df334d5501d6499b3f9ad5595cae86cce16a65"
  ) {
    id
    description
    link
  }
}
```

Since we've only deployed one application, the results returned have a single
entry.

At the bottom of the returned JSON there is a field `link`. To interact with
your application copy and paste the link into a new browser tab.

Finally, to query the counter value, run:

```gql
query {
  value
}
```

This will return a value of `42`, which is the initialization argument we
specified when deploying our application.