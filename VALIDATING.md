To generate the `status.cbor` file from the `status.edn` file, and validate it against
the CDDL, run the following.

```sh
$ diag2cbor.rb status.edn >status.cbor
$ cddl status.cddl validate status.cbor
```

At the moment the pretty-printed CBOR in the draft has to be kept in sync manually with
the `status.edn` file.
