# tr_init

Initialization code for tr! macro for easy usage and small binary size.

## Introduction

There is a very nice [`tr`](https://crates.io/crates/tr) crate with the `tr!` macro that can be used to localize Rust applications.

The macro uses `gettext` approach which I like more than the `fluent` one. I know that `fluent` has many advantages but I like that with `gettext` I've more readable source code and I can write it quicker. Your experience may vary.

The only problem with `tr` crate is that if you want to embed translations into resulting binary and auto-select correct one, the initialization code for that is pretty long (at version 0.1.3 at time of writing).

This crate makes the initialization of `tr` macro as easy as possible.

## Features

- Autodetection of user's locale.

- The resulting code size impact is very small. Instead of `locale_config` it uses simplified code from it, as `locale_config` depends on `regex` crate which is a few megabytes of size after compilation.

- The localization files are embedded in the binary.

- It search the local folder for the localization files first and then it looks to the embedded ones. That way the localization files can be added or modified without recompilation.

- It works great with `cargo-i18n`.

## Usage

_Note. This instruction uses `cargo-i18` tool. Please also read https://github.com/kellpossible/cargo-i18n for more information._ 

### Steps

Install required tools:

```shell script
cargo install xtr
cargo install cargo-i18n
```

Add the following to your `Cargo.toml` dependencies:

```toml
tr_init = "0.1"
rust-embed = "5"
```

Create an `i18n.toml` file in the root directory of your crate:
 
 ```toml
fallback_language = "en"

[gettext]
target_languages = ["en", "pl"]
output_dir = "i18n"
```
 
Run `cargo i18n` tool:

```shell script
cargo i18n
``` 

It scans the code and creates and updates the localization file. You can run it every time you want to update your localization files or compile `po` files.

Minimal example:

```rust
use tr_init::{tr_init, tr};

#[derive(rust_embed::RustEmbed)]
#[folder = "i18n/mo"]
struct Translations;

fn main() {
    tr_init!("locale", Translations);

    println!("{}", tr!("Hello, world!"));
}
```

The files from `i18n/mo` folder are embedded in the executable file. The `tr_init!` macro looks for:
- `locale/{lang}/{module_name}.mo` in the file system
- `{lang}/{module_name}.mo` in the embedded `Translations` struct  

## TODO

It currently works with `.mo` files. It would be nice if it could work directly with `.po` files without the need for the compilation. That way newly added localization files could be easily edited with any text editor.  
