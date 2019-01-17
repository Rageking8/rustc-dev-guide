# Request for stabilization

Once an unstable feature has been well-tested with no outstanding
concern, anyone may push for its stabilization. It involves the
following steps.

- Documentation PRs
- Write a stabilization report
- FCP
- Stabilization PR

## Documentation PRs

If any documentation for this feature exists, it should be
in the [`Unstable Book`], located at [`src/doc/unstable-book`].
If it exists, the page for the feature gate should be removed.

If there was documentation there, integrating it into the
existing documentation is needed.

If there wasn't documentation there, it needs to be added.

Places that may need updated documentation:

- [The Reference]: This must be updated, in full detail.
- [The Book]: This may or may not need updating, depends.
    If you're not sure, please open an issue on this repository
    and it can be discussed.
- standard library documentation: As needed. Language features
    often don't need this, but if it's a feature that changes
    how good examples are written, such as when `?` was added
    to the language, updating examples is important.
- [Rust by Example]: As needed.

Prepare PRs to update documentations invovling this new feature
for  repositories mentioned above. Maintainers of these repositories
will keep these PRs open until the whole stabilization process
has completed. Meanwhile, we can proceed to the next step.

## Write a stabilization report

Find the tracking issue of the feature, and create a short
stabilization report. Essentially this would be a brief summary
of the feature plus some links to test cases showing it works
as expected, along with a list of edge cases that came up and
and were considered. This is a minimal "due diligence" that
we do before stabilizing.

The report should contain:

- A summary, showing examples (e.g. code snippets) what is
  enabled by this feature.
- Links to test cases in our test suite regarding this feature
  and describe the feature's behavior on encountering edge cases.
- Links to the documentations (the PRs we have made in the
  previous steps).
- Any other relevant information(Examples of such reports can
  be found in rust-lang/rust#44494 and rust-lang/rust#28237).
- The resolutions of any unresolved questions if the stabilization
  is for an RFC.

## FCP

If any member of the team responsible for tracking this
feature agrees with stabilizing this feature, they will
start the FCP (final-comment-period) process by commenting

```bash
@rfcbot fcp merge
```

The rest of the team members will review the proposal. If the final
decision is to stabilize, we proceed to do the actual code modification.

## Stabilization PR

Once we have decided to stabilize a feature, we need to have
a PR that actually makes that stabilization happen. These kinds
of PRs are a great way to get involved in Rust, as they take
you on a little tour through the source code.

Here is a general guide to how to stabilize a feature --
every feature is different, of course, so some features may
require steps beyond what this guide talks about.

Note: Before we stabilize any feature, it's the rule that it
should appear in the documentation.

### Updating the feature-gate listing

There is a central listing of feature-gates in
[`src/libsyntax/feature_gate.rs`]. Search for the `declare_features!`
macro. There should be an entry for the feature you are aiming
to stabilize, something like (this example is taken from
[rust-lang/rust#32409]:

```rust,ignore
// pub(restricted) visibilities (RFC 1422)
(active, pub_restricted, "1.9.0", Some(32409)),
```

The above line should be moved down to the area for "accepted"
features, declared below in a separate call to `declare_features!`.
When it is done, it should look like:

```rust,ignore
// pub(restricted) visibilities (RFC 1422)
(accepted, pub_restricted, "1.31.0", Some(32409)),
// note that we changed this
```

Note that, the version number is updated to be the version number
of the stable release where this feature will appear. This can be
found by consulting [the forge](https://forge.rust-lang.org/), which will guide
you the next stable release number. You want to add 1 to that,
because the code that lands today will become go into beta on that
date, and then become stable after that. So, at the time of this
writing, the next stable release (i.e. what is currently beta) was
1.30.0, hence I wrote 1.31.0 above.

### Removing existing uses of the feature-gate

Next search for the feature string (in this case, `pub_restricted`)
in the codebase to find where it appears. Change uses of
`#![feature(XXX)]` from the `libstd` and any rustc crates to be
`#![cfg_attr(stage0, feature(XXX))]`. This includes the feature-gate
only for stage0, which is built using the current beta (this is
needed because the feature is still unstable in the current beta).

Also, remove those strings from any tests. If there are tests
specifically targeting the feature-gate (i.e., testing that the
feature-gate is required to use the feature, but nothing else),
simply remove the test.

### Do not require the feature-gate to use the feature

Most importantly, remove the code which flags an error if the
feature-gate is not present (since the feature is now considered
stable). If the feature can be detected because it employs some
new syntax, then a common place for that code to be is in the
same `feature_gate.rs`. For example, you might see code like this:

```rust,ignore
gate_feature_post!(&self, pub_restricted, span,
 "`pub(restricted)` syntax is experimental");
```

This `gate_feature_post!` macro prints an error if the
`pub_restricted` feature is not enabled. It is not needed
now that `#[pub_restricted]` is stable.

For more subtle features, you may find code like this:

```rust,ignore
if self.tcx.sess.features.borrow().pub_restricted { /* XXX */ }
```

This `pub_restricted` field (obviously named after the feature)
would ordinarily be false if the feature flag is not present
and true if it is. So transform the code to assume that the field
is true. In this case, that would mean removing the `if` and
leaving just the `/* XXX */`.

```rust,ignore
if self.tcx.sess.features.borrow().pub_restricted { /* XXX */ }
becomes
/* XXX */

if self.tcx.sess.features.borrow().pub_restricted && something { /* XXX */ }
 becomes
if something { /* XXX */ }
```

[rust-lang/rust#32409]:https://github.com/rust-lang/rust/issues/32409
[src/libsyntax/feature_gate.rs]:https://doc.rust-lang.org/nightly/nightly-rustc/syntax/feature_gate/index.html
[The Reference]: https://github.com/rust-lang-nursery/reference
[The Book]: https://github.com/rust-lang/book
[Rust by Example]: https://github.com/rust-lang/rust-by-example
[`Unstable Book`]: https://doc.rust-lang.org/unstable-book/index.html
[`src/doc/unstable-book`]: https://github.com/rust-lang/rust/tree/master/src/doc/unstable-book