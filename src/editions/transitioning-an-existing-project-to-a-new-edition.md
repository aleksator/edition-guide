# Transitioning an existing project to a new edition

New editions might change the way you write Rust – they add new syntax,
language, and library features, and also remove features. For example, `try`,
`async`, and `await` are keywords in Rust 2018, but not Rust 2015. If you
have a project that's using Rust 2015, and you'd like to use Rust 2018 for it
instead, there's a few steps that you need to take.

> It's our intention that the migration to new editions is as smooth an
> experience as possible. If it's difficult for you to upgrade to Rust 2018,
> we consider that a bug. If you run into problems with this process, please
> [file a bug](https://github.com/rust-lang/rust/issues/new). Thank you!

Here's an example. Imagine we have a crate that has this code in
`src/lib.rs`:

```rust
trait Foo {
    fn foo(&self, Box<Foo>);
}
```

This code uses an anonymous parameter, that `Box<Foo>`. This is [not
supported in Rust 2018](../rust-2018/trait-system/no-anon-params.html), and
so this would fail to compile. Let's get this code up to date!

## Updating your code to be compatible with the new edition

Your code may or may not use features that are incompatible with the new
edition. In order to help transition to Rust 2018, we've included a new
subcommand with Cargo. To start, let's run it:

```console
> cargo fix --edition
```

This will check your code, and automatically fix any issues that it can.
Let's look at `src/lib.rs` again:

```rust
trait Foo {
    fn foo(&self, _: Box<Foo>);
}
```

It's re-written our code to introduce a parameter name for that trait object.
In this case, since it had no name, `cargo fix` will replace it with `_`,
which is conventional for unusued variables.

`cargo fix` is still pretty new, and so it can't always fix your code automatically.
If `cargo fix` can't fix something, it will print the warning that it cannot fix
to the console. If you see one of these warnings, you'll have to update your code
manually. See the corresponding section of this guide for help, and if you have
problems, please seek help at the [user's forums](https://users.rust-lang.org/).

Keep running `cargo fix --edition` until you have no more warnings.

Congrats! Your code is now valid in both Rust 2015 and Rust 2018!

## Enabling the new edition to use new features

In order to use some new features, you must explicitly opt in to the new
edition. Once you're ready to commit, change your `Cargo.toml` to add the new
`edition` key/value pair. For example:

```toml
[package]
name = "foo"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]
edition = "2018"
```

If there's no `edition` key, Cargo will default to Rust 2015. But in this case,
we've chosen `2018`, and so our code is compiling with Rust 2018!

## Writing idiomatic code in a new edition

Editions are not only about new features and removing old ones. In any programming
language, idioms change over time, and Rust is no exception. While old code
will continue to compile, it might be written with different idioms today.

Our sample code contains an outdated idiom. Here it is again:

```rust
trait Foo {
    fn foo(&self, _: Box<Foo>);
}
```

In Rust 2018, it's considered idiomatic to use the [`dyn`
keyword](../rust-2018/trait-system/dyn-trait-for-trait-objects.html) for
trait objects.

We can ask `cargo fix` to fix it:

```console
$ cargo fix --edition-idioms
```
> The `--edition-idioms` flag applies only to the "current crate" if you want
> to run it against a workspace is necessary to use a workaround with
> `RUSTFLAGS` in order to execute it in all the workspace members.
>
> ```shell
> $ RUSTFLAGS='-Wrust_2018_idioms' cargo fix --all
> ```

Afterwards, `src/lib.rs` looks like this:

```rust
trait Foo {
    fn foo(&self, _: Box<dyn Foo>);
}
```

We're now more idiomatic, and we didn't have to fix our code manually!

As before, `cargo fix` may not be able to automatically update our code.
If `cargo fix` can't fix something, it will print a warning to the console,
and you'll have to fix it manually.

Enjoy the new edition!