# End-to-end example

One consistent scenario with every credential of the chain of trust, for the repository
[chain-of-trust-lab/rest-api](https://github.com/chain-of-trust-lab/rest-api):

1. The organization gives each of the seven agents its role ([`role/`](role)), as `04.bootstrap` does.
2. One run of the Delivery Pipeline builds (compiling and running the unit tests), tests, publishes, approves and
   deploys one commit
   ([`provenance/`](provenance)). Each step's credential is issued by the agent that holds the step's role and is sent
   to the project agent, which holds the chain.

The data is made up but consistent: DIDs, digests, versions and dates fit together the way a real run produces them.
The credentials are really signed (`Ed25519Signature2018`), but with example keys: these DIDs are not on any ledger.

## Agents and role credentials

Issuer of every role credential: the organization, `did:sov:SeQNS5zqxdhL6hHDCeA3ge`. Every role has the scope
`https://github.com/chain-of-trust-lab/rest-api`.

| Credential | Agent | DID | Actor type | Role | Issues |
| --- | --- | --- | --- | --- | --- |
| [01](role/01-maintainer.json) | maintainer | `did:sov:6StKqB4QruRydr4JA8WQMT` | Person | AuthorizedMaintainer | 01 |
| [02](role/02-approver.json) | approver | `did:sov:LyT4sf2XNxvcquAKA2M2Db` | Person | ReleaseApprover | 08 |
| [03](role/03-build.json) | build | `did:sov:29XjXQCyrW8KiqqWg4dAkV` | Stage | Builder | 02, 05 |
| [04](role/04-test.json) | test | `did:sov:C1wnJ6gyFZdmN4smVAyCoV` | Stage | Tester | 03, 06 |
| [05](role/05-deliver.json) | deliver | `did:sov:3pH8iriDney9f4b2WBAFY9` | Stage | Publisher | 04, 07 |
| [06](role/06-deploy.json) | deploy | `did:sov:G2rS1cQPkeVHFcYh4x9APD` | Service | DeployGate | 09 |
| [07](role/07-project.json) | project | `did:sov:BCtYL1m4BLVLmdMRUnuJyK` | Service | ReleaseCustodian | none; holds the chain |

## The pipeline run

Run [19482736105](https://github.com/chain-of-trust-lab/rest-api/actions/runs/19482736105/attempts/1), attempt 1, of
`.github/workflows/pipeline.yml`, on 19 Nov 2026. The commit `32f62178379e…` gives version `0.0.1-32f62178379e`.

| # | Credential | Time (UTC) | Issuer | Subject | Step fields |
| --- | --- | --- | --- | --- | --- |
| [01](provenance/01-commit.json) | SourceCommittedCredential | 09:14:22 | maintainer | commit | none |
| [02](provenance/02-build-package.json) | PackageBuiltCredential | 09:18:31 | build | package | inputs: the commit, SBOM, unit-test report |
| [03](provenance/03-test-package.json) | PackageTestedCredential | 09:20:05 | test | package | result Passed, report |
| [04](provenance/04-deliver-package.json) | PackagePublishedCredential | 09:21:40 | deliver | package | GitHub Packages |
| [05](provenance/05-build-image.json) | ImageBuiltCredential | 09:24:12 | build | image | inputs: the package |
| [06](provenance/06-test-image.json) | ImageTestedCredential | 09:26:57 | test | image | result Passed, report |
| [07](provenance/07-deliver-image.json) | ImagePublishedCredential | 09:28:19 | deliver | image | ghcr.io |
| [08](provenance/08-approve.json) | ImageApprovedCredential | 10:02:44 | approver | image | Approved for Production |
| [09](provenance/09-deploy.json) | ImageDeployedCredential | 10:05:10 | deploy | image | Production |

The artifacts:

| Subject | `id` | `name` |
| --- | --- | --- |
| commit | `urn:git:sha1:32f62178379e…` | `https://github.com/chain-of-trust-lab/rest-api` |
| package | `urn:sha256:0517e61b0f0a…` | `pkg:maven/com.github.chain-of-trust-lab/rest-api@0.0.1-32f62178379e` |
| image | `urn:sha256:2a1ec909d4c8…` | `ghcr.io/chain-of-trust-lab/rest-api:0.0.1-32f62178379e` |

## What fits together

- Each provenance credential's `issuer` is the `credentialSubject.id` of the role credential for that step, and the
  role's `scope` equals `pipelineRun.repository`. Role credentials are dated before the run.
- Each `previousCredential.credentialDigest` is the sha256 of the RFC 8785 canonical JSON of the credential before it,
  proof included; changing any credential breaks the next link.
- `inputs` of the package is the commit's subject; `inputs` of the image is the package's subject.
- Every credential passes its JSON Schema
  ([role](https://chain-of-trust-lab.github.io/contexts/role/v1.schema.json),
  [provenance](https://chain-of-trust-lab.github.io/contexts/provenance/v1.schema.json)), and every signature
  verifies with ACA-Py 1.6.2's JSON-LD code.

Invalid provenance credentials that the schema rejects are in
[`../provenance/examples/invalid`](../provenance/examples/invalid).
