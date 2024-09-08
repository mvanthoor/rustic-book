# Downloads

## Latest pre-compiled binaries

- [Rustic Alpha 3.0.0](https://github.com/mvanthoor/rustic/releases/download/alpha-3.0.0/rustic-alpha-3.0.0.0.zip)
- [Rustic Alpha 2](https://github.com/mvanthoor/rustic/releases/download/alpha-2/rustic-alpha-2-bin.zip)
- [Rustic Alpha 1.1](https://github.com/mvanthoor/rustic/releases/download/alpha-1.1/rustic-alpha-1.1-bin.zip)

## Other versions

All versions of the engine for which pre-compiled binaries where released
can be downloaded from the GitHub Releases page.

[Rustic Releases](https://github.com/mvanthoor/rustic/releases)

Browse through the page to find the release you are looking for; you can
download the binaries from the assets. It is possible that a release has no
pre-built binary; this is the case if it is just a maintenance release,
which updates either the libraries to the newest version at that point, or
fixes something in the code to adhere to the newest version of Rust. Such
releases have no engine bugfixes or changes to the chess code, so the
latest binary from that series can be used.

If you do want to use the latest version of any series, you can download
the source code ZIP file and unpack it. Then, in a terminal, go to the
location where you unzipped the source code and from the directory where
the cargo.toml file is, run the following command:

```bash,ignore
cargo build --release
```

The new executable will be built in the ./target/release directory. If you
need more information about the build process, see the chapter [Building
Rustic](../back_matter/build.md) in the back matter of this book.