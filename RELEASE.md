# Releasing the verifier

This is a  document to describe the release process for the verifier.

---

- [Publish release](#publish-release)
- [Verify provenance](#verify-provenance)
- [Update documentation](#update-documentation)
- [Update builders](#update-builders)
- [Announce](#announce)

---

## Publish release

Major and minor releases are released directly from the `main` branch. Patch versions are released from the `release/vX.Y` branch.

### New major or minor release

Create a new tag for the official generator via [slsa-framework/slsa-verifier/releases/new](https://github.com/slsa-framework/slsa-verifier/releases/new). 

Use a "canonical" semantic version without metadata `vX.Y.Z`.

Set the title to `vX.Y.Z`.

Click `Publish release`.

This will trigger a release workflow: wait until it completes and generates the binary and the provenance.

From the repository landing page, use the branch drop-down to create a branch from the tagged release with the format `release/vX.Y`. This will be used for backporting critical fixes and releases patch versions.

### New patch release

Critical patch fixes are released from the `release/vX.Y` branch. Once the backported fix has been merged, create a new tag for the official generator via [slsa-framework/slsa-verifier/releases/new](https://github.com/slsa-framework/slsa-verifier/releases/new). Use the `release/vX.Y` branch as the Target.

Use a "canonical" semantic version without metadata `vX.Y.Z`.

Set the title to `vX.Y.Z`.

Click `Publish release`.

This will trigger a release workflow: wait until it completes and generates the binary and the provenance.

## Verify provenance

Follow the steps:

1. Download the binary and provenance from https://github.com/slsa-verifier/slsa-verifier/releases/tag/vX.Y.Z

2. Clone the slsa-verifier repo, compile and verify the provenance:
```
$ git clone git@github.com:slsa-framework/slsa-verifier.git
$ cd slsa-verifier
# $ (Optional: git checkout tags/v1.1.1)
$ go run . -artifact-path slsa-verifier-linux-amd64 -provenance slsa-verifier-linux-amd64.intoto.jsonl -source github.com/slsa-framework/slsa-verifier -tag vX.Y.Z
```

You should include the `-branch release/vX.Y` for patch version releases.

If the provenance verification fails, delete the release and the tag. Otherwise, continue.

## Update documentation

Follow these steps:

1. Compute the hash of the binary. One of the following commands will do:
```
$ cat slsa-verifier-linux-amd64.intoto.jsonl | jq -r '.payload' | base64 -d | jq -r '.subject[0].digest.sha256'
```
or
```
$ sha256sum slsa-verifier-linux-amd64
```

2. Add an additional entry at the top of [SHA256SUM.md](./SHA256SUM.md):

```
### [vX.Y.Z](https://github.com/slsa-framework/slsa-verifier/releases/tag/vX.Y.Z)
<the-hash>  slsa-verifier-linux-amd64
```

3. Update the latest version in the [README.md](./README.md):

```shell
$ sed -i "s/v1.0.0/v1.1.1/g" ./README.md
```

4. Send a pull request with the changes. In the description, explain the steps to verify the hash update, i.e., reviewers shoud LGTM only if the provenance verification succeeds
and the hash in the pull request matches the one computed on the binary. You can use [#slsa-framework/slsa-github-generator#113](https://github.com/slsa-framework/slsa-github-generator/pull/113) as example.

## Update builders

Send a similar pull request to update the hash and version of the verifier for the workflow [slsa-framework/slsa-github-generator/blob/main/.github/workflows/builder_go_slsa3.yml#L30-L31](https://github.com/slsa-framework/slsa-github-generator/blob/main/.github/workflows/builder_go_slsa3.yml#L30-L31). Explain the steps to verify the hash. If the pull request for the verifier is already merged, you can simply point to it instead.

Note: you need not cut a release for the generator, unless the verifier has important changes that are required for the builders to work properly.

