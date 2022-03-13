# About
Ubuntu 20.04 based build environment for peaq-network-node

# To build
```bash
pushd build-image && \
chmod +x *.sh && \
sh build-container.sh && \
popd
```

# To use

Extract the contents of root home folder, so we can mount it later to save on build times
```bash
./extract-root-contents.sh
```

Change into the peaq-network-node directory and start the build container, the following command assumes, this repo and peaq-network-node are siblings on the file system

```bash
docker run -it --rm -v $PWD:/work -v $PWD/../peaq-network-node-docker-builder/dev-env/root:/root rust-stable:ubuntu-20.04 /bin/bash
cd /work
cargo build --release
```
or you can use below commands
```bash
cd ../peaq-network-node
docker run --rm -it -v $(pwd):/sources rust-stable:ubuntu-20.04 cargo build --release --manifest-path=/sources/Cargo.toml
```

The built binary will be on the host machine, under peaq-network-node/target/release/peaq-node

### Tracing node
If we want to build the node which supports the EVM tracing module, we have to follow below commands. The runtime module with EVM tracing module is in the `target_runtime` folder after the compilation.

```bash
cd ../peaq-network-node

# Build runtime module
docker run --rm -it --env CARGO_TARGET_DIR="/sources/target_runtime" -v $(pwd):/sources rust-stable:ubuntu-20.04 cargo build --release -p peaq-node-runtime --features "std aura evm-tracing" --manifest-path=/sources/Cargo.toml
# Build node
docker run --rm -it -v $(pwd):/sources rust-stable:ubuntu-20.04 cargo build --release --manifest-path=/sources/Cargo.toml
```

After building the runtime module and node, we can start a node by replacing the runtime module with EVM tracing feature.
```bash
# Copy the runtime module
mkdir -p wasm
cp target_runtime/release/wbuild/peaq-node-runtime/peaq_node_runtime.wasm wasm


# Run the tracing node
./target/release/peaq-node \
--dev \
--tmp \
--ws-external \
--rpc-external \
--ethapi=debug,trace,txpool \
--wasm-runtime-overrides wasm \
--execution Wasm
```

## RBAC
If you want to build RBAC, you can follow below commands

```bash
cd ../RBAC
docker run --rm -it -v $(pwd):/sources rust-stable:ubuntu-20.04 cargo +nightly contract build --manifest-path=/sources/Cargo.toml
docker run --rm -it -v $(pwd):/sources rust-stable:ubuntu-20.04 cargo +nightly contract test --manifest-path=/sources/Cargo.toml
```
