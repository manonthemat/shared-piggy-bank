# Shared Piggy Bank

This repo houses a simple Plutus Contract that acts as a shared piggy bank.

## Setting up

### Cabal+Nix build

Set up your machine to build things with `Nix`, following the [Plutus README](https://github.com/input-output-hk/plutus/blob/master/README.adoc) (make sure to set up the binary cache!).

To enter a development environment, simply open a terminal on the project's root and use `nix-shell` to get a bash shell:

```
$ nix-shell
```

Otherwise, you can use [direnv](https://github.com/direnv/direnv) which allows you to use your preferred shell. Once installed, just run:

```
$ echo "use nix" > .envrc # Or manually add "use nix" in .envrc if you already have one
$ direnv allow
```

and you'll have a working development environment for now and the future whenever you enter this directory.

The build should not take too long if you correctly set up the binary cache. If it starts building GHC, stop and setup the binary cache.

Afterwards, the command `cabal build server` from the terminal should work (if `cabal` couldn't resolve the dependencies, run `cabal update` and then `cabal build`).

## The Plutus Application Backend (PAB)

With the PAB we can serve and interact with contracts over a web API.
You can read more about the PAB here: [PAB Architecture](https://github.com/input-output-hk/plutus/blob/master/plutus-pab/ARCHITECTURE.adoc).

Here, the PAB is configured with one contract, the `PiggyBank` contract from `./src/Plutus/Contracts/PiggyBank.hs`.

Here's an example of running and interacting with this contract via the API. For this it will help if you have `jq` installed.

1. Compile the contract and build the server:

```
cabal build server
```

2. Run the PAB binary:

```
cabal exec -- server
````

This will then start up the server on port 9080.

3. Check what contracts are present:

```
curl -s http://localhost:9080/api/contract/definitions | jq
```

You should receive a list of contracts and the endpoints that can be called on them, and the arguments
required for those endpoints.

We're interested in the `PiggyBankContract`.

#### Put and empty

After starting the PAB, you may want to run `./run.sh` to send some requests to the server.

First, we activate four distinct wallets. These wallets come with some ADA tokens.
Next, two wallets put 99999900000 lovelaces (1 million lovelaces = 1 ADA) into the piggy bank.

Next, wallet 3 is trying to empty the piggy bank and claim the ADA in it.
However, the data sent along the request is wrong, so in the log of the server we will see that the transaction didn't validate.

We're being very nice here, because that attempt actually didn't cost wallet 3 anything.
A strong selling point for Cardano is that we can check quite a bit of information before data ever hits the chain.
Anyone who spent hundreds of USD on a few occassions on a failed ETH transaction will appreciate this.

Finally, wallet 4 creates a correct request and empties the piggy bank.

Note that you can verify the balances by looking at the log of the server when exiting by pressing return.
