---
marp: true
---

<!--
theme: gaia
_class: lead
style: |
  code {
    font-weight: 700;
    color: inherit;
    background-color: inherit;
  }
  code.language-rust {
    color: #ddd;
  }
  code.language-text, code.language-toml {
    font-weight: 700;
    color: #ddd;
  }
-->

# Publishing a Rust crate
2021/08
Eric Seppanen

---

## Crate names

The crates.io / Cargo ecosystem has a flat namespace.

- First one to claim a name owns it!

---

## Cargo.toml metadata

There are a number of fields that should be added to `Cargo.toml` for published crates:

- `description`
- `repository`
- `license`
- `keywords`
- `categories`

See the [Cargo Book][cargo_manifest] for more details.

---

## Libraries & Binaries

Published crates are usually libraries, but they can be used to distribute binaries as well.

Example:
```text
$ cargo install ripgrep
```

*Note: docs.rs doesn't host documentation for binary crates.*

---

## Documentation

All crates published to crates.io will automatically get docs hosted on docs.rs.

Documentation should include:
- Top-level docs in `lib.rs`
- A `README.md` file (usually a copy of the top-level docs).
- Documentation for every public type, function, trait, etc.
- Module docs can also have detailed explanations.

---

## Documentation: best practices

- Use `#![warn(missing_docs)]`
- Add examples to all documentation (doc-tests).
- Document the reasons an error may be returned.
- Document any panics.
- If any `unsafe` functions are public, document the invariants that are expected.

See the [Rust Book][book_documentation] for details.

--- 

## Syncing `README.md` and top module docs

Three options:

- manual copy/paste (not great because of `//!`)
- [`cargo readme`]
- use [`doc = include_str!`] to import README as the module docs.


---

## doc tests

Examples in rust documentation will be compiled and run by `cargo test` or `cargo test --doc`.

Examples that aren't supposed to work can be annotated with `ignore`, `no_run`, `compile_fail`, or `should_panic`.

See the [rustdoc book][doctests] for more details.


---

## Semantic Versioning

Rust crates follow semantic versioning:

```text
1.0.0
1.0.1   # Patch
1.1.0   # Minor version
2.0.0   # Major version
```

---

### Semantic Versioning: allowed changes

Examples of major "breaking" changes:

- adding public fields to structs
- removing public items

Examples of minor changes:

- adding a new public item
- loosening generic bounds

See the [Cargo Book][cargo_semver] for more details.

--- 

## Cargo and SemVer

Downstream crates that depend on your crate will probably assume that minor versions will work with no code changes.

```toml
[dependencies]
some_crate = "0.1.2" # May be upgraded to 0.1.9
other_crate = "= 1.2.3" # Pinned to an exact version
```

Most library crates don't commit `Cargo.lock`, and even if they did, in many cases Cargo will upgrade anyway.

---

## Semantic Versioning for 0.x.y

"Development" releases are treated differently.

```text
0.1.0
0.1.1   # Treated like a minor release
0.2.0   # Treated like a major release
```

---

## Minimum Supported Rust Version

Crates usually document an **MSRV** to indicate which Rust version is required to build.

Many crates require a fairly recent Rust build (~1 year old).

---

## Publishing a crate release

Be careful! Once published, you can't un-publish a release.

Make a checklist of things to do before publishing.

Use tooling like `cargo release` to automate some steps:
- `Cargo.toml` version is updated and committed.
- git tag(s) are created.
- tags are pushed to the upstream repo.
- publish to crates.io.

---

## Pre-publishing checklist

- Clean repo directory (including `Cargo.lock`)
- Build and test on stable, nightly, msrv.
- Run `cargo fmt`.
- Run clippy.
- Run `cargo doc` and fix any warnings.
- Make sure documentation is up to date, including `README.md`.
- If optional features exist, build and test with various feature combinations.

---

## Pre-publishing checklist, continued

- Check if your dependencies have major updates; decide whether to upgrade.
- Check if your dependencies have security vulnerabilities (`cargo audit`).
- grep for `unimplemented!`, `todo!`, `XXX`, `FIXME`, `println`, `dbg!`...
- check code coverage (`cargo tarpaulin`?)

---

## Cargo yank

`cargo yank` will tag a release as "yanked", causing dependent builds to try to select another version.

It does not remove the release data, and any builds that are already using that version will continue to use it.

---

## Useful attributes

```rust
// Throw warnings if any public items are undocumented.
#![warn(missing_docs)]
```

```rust
// Don't allow `unsafe` in this crate.
#![forbid(unsafe_code)]
```
```rust
// Don't allow implicit truncation of integers,
// e.g. `foo as u16`
#![warn(clippy::cast_possible_truncation)]
```

---

## Check the license of dependencies

```text
$ cargo license
Apache-2.0 OR BSL-1.0 (1): ryu
Apache-2.0 OR MIT (19): base64, float-ord, half, heck, hex, itoa, 
proc-macro-crate, proc-macro2, quote, rustversion, serde, serde_cbor, 
serde_derive, serde_json, syn, toml, unicode-segmentation, unicode-xid, 
version_check
MIT (8): cddl-cat, escape8259, nom, ntest, ntest_proc_macro_helper, 
ntest_test_cases, ntest_timeout, strum_macros
MIT OR Unlicense (1): memchr
```

---

## Optional crate features

Crate features must only be additive (adding modules, functions, types).

They should not e.g. change the number of parameters to a function.

If you have optional features, run tests with different feature sets enabled.

---

[cargo_semver]: https://doc.rust-lang.org/cargo/reference/semver.html

[cargo_manifest]: (https://doc.rust-lang.org/cargo/reference/manifest.html)

[doctests]: https://doc.rust-lang.org/rustdoc/documentation-tests.html

[book_documentation]: https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html?making-useful-documentation-comments

[`cargo readme`]: https://github.com/livioribeiro/cargo-readme

[`doc = include_str!`]: https://blog.rust-lang.org/2021/07/29/Rust-1.54.0.html#attributes-can-invoke-function-like-macros
