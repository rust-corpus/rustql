# RustQL: a Tool for Semantic Querying of Rust Code.

RustQL is a prototype of Niko Matsakis'
[idea](http://smallcultfollowing.com/babysteps/blog/2017/02/17/project-idea-datalog-output-from-rustc/)
to use Datalog for understanding how Rust programmers write code. The
initial version was developed by Nicolas Winkler. The project is
currently maintained by the [Programming
Methodology](http://www.pm.inf.ethz.ch/) group at ETH Zurich.

The tool consists of three parts:

1.  Extractor – a Rust compiler plugin that extracts the information
    about the crate during compilation.
2.  Linker – a program that merges the information about different
    crates into a single database.
3.  Query Engine – a program that takes a Datalog like query and
    evaluates it on the database. The engine is based on the
    [Datafrog](https://crates.io/crates/datafrog).

## Extractor

Build the extractor and prepare the environment:

```bash
cd rustql-extractor
cargo build --release
source prepare_env.sh
```

Extract information about the crate and its dependencies by compiling
it:

```bash
cd <some-crate>
cargo clean
cargo build
```

This will emit information about the compiled crates at `~/.rustql/crates/`.

## Linker

To create the database, compile the linker and run it:

```bash
cd rustql-linker
cargo run --release
```

This will create a file `database.db` in the current directory.

## Query Engine

When you run a query engine, it will enter a loop in which you can
provide paths to queries to you want to execute:

```bash
cd ../rustql-query/
env RUST_BACKTRACE=1 cargo run --release
```

Example queries to try out:

```plain
samples/same_type.rql
samples/thief.rql
samples/unsafe.rql
```

## Getting Started

This tutorial will show how to download most popular 500 crates and run
a simple query over them.

### Check Out Submodules


```bash
git submodule update --init
```

### Set Up Vagrant

Note: If you are running on a server, you probably want to allocate a
larger VM by editing `Vagrantfile`.

Install [Vagrant](https://www.vagrantup.com/) and start the VM:

```bash
vagrant up
vagrant ssh
```

All remaining steps should be done inside the VM from the directory
`/vagrant` that is mapped to the directory on the host in which the
repository was cloned.

### Download Top 500 Crate Metadata

```bash
cd top-crates/
cargo run
cd ..
```

### Run Extractor

```
cd rustql-extractor/
cargo build --release
cd ..
mkdir -p data/cache
python3 bin/run_extractor.py
rm -f /vagrant/data/crates/rustql_dummy_*
```

### Link

```bash
cd rustql-linker
export EXTRACTOR_TARGET_DIR=/vagrant/data/crates/
cargo run --release
cd ..
```

### Start Query Engine

```bash
cd rustql-query/
RUST_BACKTRACE=1 cargo run --release
cd ..
```
