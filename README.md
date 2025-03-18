# AI BWRAP

This repository provides scripts to call ai tools with bwarp to do every basic
sandboxing.

This is useful to ensure that the software will not access intentionally or
by user error data that it should not see.

Wrapping the tools with `brwap` is by no means bullet-proof.
It increases security but gives not guarantees.
You still have to worry about open ports on the **loopback device**,
in most cases **D-BUS access** and things that might be shared in
`/run/user/<yourid>` or whatever needs to be shared to make the
application of your choice run.

## Supported Tools

- cusor.com (editor)
  - verified on debian-bookworm-amd46

## Installation

This wrapper collection requires a Linux installation. Required packages are
`bash` and `bwarp`

## Contributions

Are welcome! :)
