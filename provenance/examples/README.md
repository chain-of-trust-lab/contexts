# ProvenanceCredential v1 examples

The signed examples, one of each type, are part of the end-to-end example in
[`examples/provenance`](../../examples/provenance), next to the role credentials of the agents that issue them.

[`invalid/`](invalid) holds unsigned credentials that the
[JSON Schema](https://chain-of-trust-lab.github.io/contexts/provenance/v1.schema.json) rejects:

| File | Why it is rejected |
| --- | --- |
| [missing-previous-credential.json](invalid/missing-previous-credential.json) | A PackageBuiltCredential without `previousCredential` |
| [wrong-artifact-type.json](invalid/wrong-artifact-type.json) | A PackageBuiltCredential whose subject is a DeployableImage |
| [undefined-field.json](invalid/undefined-field.json) | A field the context does not define; ACA-Py also refuses to sign it |
| [unknown-result.json](invalid/unknown-result.json) | A `result` that is neither Passed nor Failed |
| [inputs-on-test-step.json](invalid/inputs-on-test-step.json) | `inputs` on a PackageTestedCredential, where only build steps have them |
