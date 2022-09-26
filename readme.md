# packetcrypt_rs
PacketCrypt implementation in Rust

## How does this differ from the offical packetcrypt_rs release?
This version of packetcrypt_rs writes the ann miner statistics to a log in InfluxDB Line Protocol format.

This log can be consumed by Telegraf and fed into InfluxDB.

## What data is written to the logs?
1. Encryptions (ke/s)
2. Bandwidth (Mb/s)
3. Goodrate (per pool)
4. Anns accept (per pool)
5. Anns reject (per pool)
6. Anns overload (per pool)

## How do I implement this?
Create an empty file in `/var/log` named `packetcrypt_stats.log`.

Add the following to your `telegraf.conf` file

    [[inputs.tail]]
        files = ["/var/log/packetcrypt_stats.log"]
        watch_method = "inotify"
        data_format = "influx"

Restart Telegraf

    sudo systemctl restart telegraf

If you already have packetcrypt_rs installed, remove the directory (you can leave the miner running)

    rm -rf packetcrypt_rs

Clone and build the repository

    git clone https://github.com/3rror404/packetcrypt_rs
    cd packetcrypt_rs
    cargo build --release

Start/restart the miner

(Optional)
Create a cron job to wipe the log at midnight everyday

    00 00 * * * echo -n >/var/log/packetcrypt_stats.log


# Original Readme

## What exists
PacketCrypt mining is made up of 6 distinct components:
* Master - provides work and config files to everyone for a given pool
* Announcement Miner - generates announcements (data which uses bandwidth)
* Announcement Handler - consumes announcements, checks validity and provides to block miner
* Block Miner - uses announcements for for mining blocks
* Block Handler - takes shares from Block Miner and validates them
* Paymaker - takes messages from Block Handler and Announcement Handler to
decide which miners the pool should pay

The Master, Announcement Handler, Block Handler, and Paymaker must be operated
by the mining pool, the Announcement Miner and Block Miner can be operated by 3rd
parties.

This codebase currently provides the *Announcement Handler* and *Announcement Miner* components.
All of the others can be found in
[the C PacketCrypt project](https://github.com/cjdelisle/PacketCrypt).

## Install
First install rust if you haven't, see: [rustup](https://rustup.rs/)

    git clone https://github.com/cjdelisle/packetcrypt_rs
    cd packetcrypt_rs
    cargo build --release

## Install Guide with CLANG build

Step 1: Copy and run the following command

    sudo apt-get update && sudo apt-get install gcc git curl make clang && curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh && mkdir ~/packet && cd ~/packet && git clone https://github.com/cjdelisle/packetcrypt_rs && cd packetcrypt_rs

Step 1.5(Optional). If you were using a gcc build then please copy and run the following command

    cargo clean

Step 2: Copy and run the following command

    CC=clang cargo build --release

## Mine announcements

* `./target/release/packetcrypt ann <pool url> --paymentaddr <your PKT addr>`

For more information `./target/release/packetcrypt help ann`

## Run an Announcement Handler
If you're running a pool, you can use the Rust announcement handler as follows:
* `./target/release/packetcrypt ah -C /path/to/pool.toml`

See [pool.example.toml](https://github.com/cjdelisle/packetcrypt_rs/blob/master/pool.example.toml)
for information about what should be in your pool.toml file.

For more information `./target/release/packetcrypt help ah`

## Env vars
* `RUST_LOG=packetcrypt=debug` for better logging
* `RUST_BACKTRACE=1` for backtraces on errors (including non-critical ones)

## Memory leak detection
To run with memory leak detection, build with `cargo build --features leak_detect` and while
it is running send a SIGUSR1 signal, this will cause it to write out all of it's long lived memory
to a file.

## Jemalloc
You may achieve better performance by building with `cargo build --release --features jemalloc`

## License

LGPL-2.1 or LGPL-3.0, at your option
