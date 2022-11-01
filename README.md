# clap_complete_nushell

Generates [Nushell](https://github.com/nushell/nushell) completions for [`clap`](https://github.com/clap-rs/clap) based CLIs

[![Crates.io](https://img.shields.io/crates/v/clap_complete_nushell)](https://crates.io/crates/clap_complete_nushell)
[![Crates.io](https://img.shields.io/crates/d/clap_complete_nushell)](https://crates.io/crates/clap_complete_nushell)
[![License](https://img.shields.io/github/license/nibon7/clap_complete_nushell)](LICENSE)
[![docs.rs](https://img.shields.io/docsrs/clap_complete_nushell)](https://docs.rs/clap_complete_nushell)
[![Build Status](https://img.shields.io/github/workflow/status/nibon7/clap_complete_nushell/CI/master)](https://github.com/nibon7/clap_complete_nushell/actions/workflows/ci.yaml?query=branch%3Amaster)

## Examples

### myapp.rs

```rust
use clap::{Arg, ArgAction, Command};
use clap_complete::generate;
use clap_complete_nushell::Nushell;
use std::io;

fn main() {
    let mut cmd = Command::new("myapp")
        .version("3.0")
        .propagate_version(true)
        .about("Tests completions")
        .arg(
            Arg::new("file")
                .value_hint(clap::ValueHint::FilePath)
                .help("some input file"),
        )
        .arg(
            Arg::new("config")
                .action(ArgAction::Count)
                .help("some config file")
                .short('c')
                .visible_short_alias('C')
                .long("config")
                .visible_alias("conf"),
        )
        .arg(Arg::new("choice").value_parser(["first", "second"]))
        .subcommand(
            Command::new("test").about("tests things").arg(
                Arg::new("case")
                    .long("case")
                    .action(ArgAction::Set)
                    .help("the case to test"),
            ),
        )
        .subcommand(
            Command::new("some_cmd")
                .about("top level subcommand")
                .subcommand(
                    Command::new("sub_cmd").about("sub-subcommand").arg(
                        Arg::new("config")
                            .long("config")
                            .action(ArgAction::Set)
                            .value_parser([clap::builder::PossibleValue::new(
                                "Lest quotes aren't escaped.",
                            )])
                            .help("the other case to test"),
                    ),
                ),
        );

    generate(Nushell, &mut cmd, "myapp", &mut io::stdout());
}
```


### myapp.nu

```nu
module completions {

  def "nu-complete myapp choice" [] {
    [ "first" "second" ]
  }

  # Tests completions
  export extern myapp [
    file?: string	# some input file
    --config(-c)	# some config file
    --conf	# some config file
    -C	# some config file
    choice?: string@"nu-complete myapp choice"
    --version(-V)	# Print version information
  ]

  # tests things
  export extern "myapp test" [
    --case: string	# the case to test
    --version(-V)	# Print version information
  ]

  # top level subcommand
  export extern "myapp some_cmd" [
    --version(-V)	# Print version information
  ]

  def "nu-complete myapp some_cmd sub_cmd config" [] {
    [ "Lest quotes aren't escaped." ]
  }

  # sub-subcommand
  export extern "myapp some_cmd sub_cmd" [
    --config: string@"nu-complete myapp some_cmd sub_cmd config"	# the other case to test
    --version(-V)	# Print version information
  ]

}

use completions *
```
