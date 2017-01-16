# heroku-buildpack-rust

[![Build Status](https://travis-ci.org/intrepion/heroku-buildpack-rust.svg?branch=master)](https://travis-ci.org/intrepion/heroku-buildpack-rust)

**Features:**

* Cached [rustup.sh](https://rustup.rs), the Rust toolchain installer.
* Configurable Rust version selection inside of the `Cargo.toml`, or by specifying the `$RUST_VERSION` environment variable.

## Configuration

We currently (ab)use the `cargo`'s "target" feature to set the version desired.
Unfortunately because of this there are sometimes (harmless) `cargo` warnings
about an unused value in the `toml` file.

Example:

## Instructions

```bash
heroku create --buildpack https://github.com/intrepion/heroku-buildpack-rust
```

## Example App

Make a `Cargo.toml` file:

```toml
[package]
name = "intrepion-heroku-rust-iron"
version = "0.1.0"
authors = ["Oliver Forral <intrepion@gmail.com>"]

[dependencies]
iron = "*"

```

In `src/main.rs` let's use a simple [iron](http://ironframework.io/) demo:

```rust
extern crate iron;

use iron::prelude::*;
use iron::status;
use std::env;

fn main() {
    fn hello_world(_: &mut Request) -> IronResult<Response> {
        Ok(Response::with((status::Ok, "Hello World!")))
    }

    let url = format!("0.0.0.0:{}", env::var("PORT").unwrap());

    println!("Binding on {:?}", url);
    Iron::new(hello_world).http(&url[..]).unwrap();
    println!("Bound on {:?}", url);
}
```

Now the following steps:

```bash
git add src/main.rs Cargo.toml && \
git commit -m "Init" && \
git push heroku master
```

Heroku should then build your application. Finally, you *may* need to start your
application's `web` dyno with:

```bash
heroku ps:scale web=1
```

Now you can visit the url given in the output
and see your application!

## Testing

If you have Docker, you can test this buildpack by doing the following:

```bash
make
```

The `Makefile` defines how to pull down the testrunner and build the appropriate
docker container, then test the buildpack.
