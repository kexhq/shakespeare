# Rodolfo browser example

This example starts a Rodolfo server on an OS assigned loopback port and
drives it with Shakespeare. It requires no changes to Rodolfo.

From the Shakespeare checkout, with Rodolfo at `../rodolfo`:

```sh
kex --source-root src --source-root ../rodolfo/src examples/rodolfo/spec/home.spec.kex
```
