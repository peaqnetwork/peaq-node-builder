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

## RBAC
If you want to build RBAC, you can follow below commands

```bash
cd ../RBAC
docker run --rm -it -v $(pwd):/sources rust-stable:ubuntu-20.04 cargo +nightly contract build --manifest-path=/sources/Cargo.toml
docker run --rm -it -v $(pwd):/sources rust-stable:ubuntu-20.04 cargo +nightly contract test --manifest-path=/sources/Cargo.toml
```
