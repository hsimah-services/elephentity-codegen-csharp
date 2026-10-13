# elephentity-codegen-csharp

The C# builder for [Elephentity](https://github.com/hsimah-services/elephentity): the compiled spec in,
C# source out.

**This is the scaffold.** `eleph-gen-csharp` speaks the builder protocol, validates its configuration
and answers `describe`. It emits no files yet; the emitters (entities, enums, contracts, catalogue and
dependency-injection wiring) are the next step. Implemented in Rust, like every Elephentity builder.

It reads one JSON request on stdin and writes one JSON response on stdout, and never touches the
filesystem: it returns paths and bodies, and signing and writing happen in
[elephentity-codegen](https://github.com/hsimah-services/elephentity-codegen) after it exits.

## Installing it

No PHP or Composer is needed. Pin a release in `eleph.json`, then fetch it:

```bash
eleph install            # downloads eleph-gen-csharp@0.1.0 into tools/builders/
eleph install --check    # fails if it is missing, another version, or modified
```

From this repository: `cargo install --path . --locked`, or `cargo build --release --locked` and use
`bin/eleph-gen-csharp`, which picks `target/release` then `target/debug`. With Composer, `composer
require --dev elephentity/codegen-csharp` provides `vendor/bin/eleph-gen-csharp` (build it with cargo).

## Configuring it

```json
{
  "spec": "spec",
  "codegen": "eleph-codegen@0.2.0",
  "targets": {
    "csharp": {
      "builder": "eleph-gen-csharp@0.1.0",
      "output": "Generated",
      "namespace": "Hsimah.Entity",
      "typeNamespace": "Hsimah.Type",
      "check": ["dotnet", "run", "--project", "tests/Contracts", "--", "eleph-check"]
    }
  }
}
```

| Key | Required | |
|---|---|---|
| `builder` | yes | The program; `name@version` pins a release for `eleph install`. |
| `output` | yes | Where the tree is written. Its own directory, relative to the project. |
| `namespace` | yes | The C# namespace of the generated entities: dot-separated identifiers, none a reserved word. |
| `typeNamespace` | yes | The C# namespace of the declared types. |
| `check` | no | The command that runs the contract check, as an array. |

Every problem with the configuration is reported at once, as errors in a normal response, and no file
is written.

### `check`

Only the project knows where its contract-test project is, so it says so here, and the builder echoes
the command back in `describe` as `provides.check`. `eleph check` then runs it, with the project as the
working directory, and reads one `{"severity", "message"}` JSON object per line of its output
(`ContractCheck.TryRunConsole` in `Elephentity.Runtime.Testing` prints exactly that for `-- eleph-check`).
A target with no `check` is reported by `eleph check` as having no checker; it never passes silently.

## Working on it

```bash
cargo fmt --check
cargo clippy --locked --all-targets -- -D warnings
cargo test --locked
echo '{"elephentity":1,"irVersion":"1.2","request":"describe","target":"csharp"}' | cargo run -q
```

- `rust/ir.rs` is a byte-identical copy of the PHP builder's, recorded in [SOURCE.md](SOURCE.md) and
  checked by `tests/ir_copy.rs`.
- `tests/protocol.rs` runs the real binary against the five request fixtures in `tests/fixtures/golden`,
  copied from `elephentity-codegen-php`.
- The version gate is the envelope (`1`) and the IR (`1.2`), declared in `rust/main.rs` and again in every
  other program. A mismatch is refused in both directions.

## Releasing

Releasing is the owner's call: an annotated `v*` tag runs `release.yml`, which uses
`elephentity-codegen`'s reusable workflow to build and publish four archives and `SHA256SUMS`
(`docs/RELEASING.md` there is the contract `eleph install` relies on).

MIT licensed.
