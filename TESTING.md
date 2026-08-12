# Testing

This document describes how to run the tests locally, how GitHub Actions runs
them, and which tests are covered by the test runner in `run.sh`.

## Running tests locally

### Run the functional suite against an existing lab

Use this when the SR Linux Containerlab topology is already deployed:

```bash
./run.sh test
```

This prepares the local collection environment, reverts the device to the
initial checkpoint, and runs the default functional suite. Pass `-vvvv` or
another Ansible verbosity option for debugging:

```bash
./run.sh test -vvvv
```

Run an individual test runner function directly when investigating one case:

```bash
./run.sh test-set-leaves
```

The individual functions are listed by:

```bash
./run.sh help
```

Run the optional OpenConfig tests by passing `oc-tests`:

```bash
./run.sh test oc-tests
```

### Run the CI-style functional setup locally

Use this when the lab is not already deployed:

```bash
./run.sh ci-test
```

This installs the pinned Containerlab version, installs the local collection
and dependencies, deploys the SR Linux topology, prints environment details,
and runs the functional suite with `tests/ci-ansible.cfg`.

### Run Ansible sanity checks locally

```bash
./run.sh sanity-test
```

This installs the local collection, changes into its installed collection
root, and runs:

```text
ansible-test sanity --docker default -v
```

The sanity command requires a working Docker daemon because Ansible runs its
test controller in the default Ansible test container.

### Galaxy installation retries

Collection installation uses a 120-second Galaxy timeout, three attempts, and
exponential backoff of 10 and 20 seconds. Override the defaults when needed:

```bash
GALAXY_INSTALL_TIMEOUT=180 \
GALAXY_INSTALL_ATTEMPTS=5 \
GALAXY_INSTALL_RETRY_DELAY=15 \
./run.sh sanity-test
```

## How tests run in CI

### `.github/workflows/cicd.yml`

The main CI workflow runs on pull requests, version-tag pushes, and manual
dispatches. It has three stages:

1. `prepare-matrix` converts `.github/matrix.yml` from YAML to JSON.
2. `test` runs the functional playbook suite against a Containerlab SR Linux
   device.
3. `ansible-sanity-test` runs Ansible collection sanity checks.

Both test jobs use the same matrix and set `fail-fast: false`, so every
core/Python combination is reported independently.

The functional job:

1. Checks out the repository.
2. Installs the matrix Python version.
3. Installs the exact matrix `ansible-core` version.
4. Runs `./run.sh ci-test`.

The sanity job performs the same checkout, Python setup, and Ansible core
installation, then runs `./run.sh sanity-test`.

### `.github/workflows/container-build.yml`

The container workflow runs on releases and manual dispatches. It uses the
separate `.github/container-matrix.yml` to build and publish Ansible core
images. These image builds are separate from the functional and sanity test
matrix.

## Ansible core/Python matrix

`.github/matrix.yml` is an explicit `include` matrix organized by
`ansible-core` version. Each entry contains:

- `ansible-core-version`: the exact Ansible core package to install
- `python-version`: the control-node Python interpreter
- `runs-on`: the GitHub Actions runner image

The current matrix has 18 combinations:

- ansible-core 2.16: Python 3.10, 3.11, 3.12 on Ubuntu 22.04
- ansible-core 2.17: Python 3.10, 3.11, 3.12 on Ubuntu 22.04
- ansible-core 2.18: Python 3.11, 3.12, 3.13 on Ubuntu 24.04
- ansible-core 2.19: Python 3.11, 3.12, 3.13 on Ubuntu 24.04
- ansible-core 2.20: Python 3.12, 3.13, 3.14 on Ubuntu 24.04
- ansible-core 2.21: Python 3.12, 3.13, 3.14 on Ubuntu 24.04

The supported ranges are based on the
[Ansible core support matrix](https://docs.ansible.com/projects/ansible/latest/reference_appendices/release_and_maintenance.html#ansible-core-support-matrix).

Each matrix entry produces one functional test job and one sanity test job.

## Functional test lifecycle

`ci-test`:

1. Installs the pinned Containerlab version.
2. Installs the local collection and its dependencies through
   `ansible-galaxy`.
3. Deploys the SR Linux Containerlab topology.
4. Prints installed collections, Python packages, and the Python version.
5. Runs the functional suite with `tests/ci-ansible.cfg`.

Most state-changing playbooks revert the device to the `clab-initial`
checkpoint before running. The JSON-RPC replacement test runs last because it
restarts the JSON-RPC server.

## Functional tests in the default suite

The following tests are called by `_run-tests` in `run.sh`. Each test runs the
referenced playbook against the `clab` host and validates the module result
with Ansible assertions or `failed_when` conditions.

### Connectivity and retrieval

- **`test-auth-fail`** — `tests/playbooks/auth-fail.yml`: uses an invalid
  password and verifies that the module returns an `AuthenticationFailed`
  error.
- **`test-get-container`** — `tests/playbooks/get-container.yml`: retrieves
  `/system/information` from the state datastore and verifies the response
  identifies SR Linux.
- **`test-get-wrong-path`** — `tests/playbooks/get-wrong-path.yml`: requests
  an invalid path and verifies the expected `Path not valid` error.
- **`test-get-multiple-paths`** — `tests/playbooks/get-multiple-paths.yml`:
  retrieves multiple paths using different datastores and SR Linux YANG model
  selection, then validates both responses.
- **`test-config-backup`** — `tests/playbooks/backup-cfg.yml`: retrieves the
  complete running configuration, writes JSON and YAML backups, and verifies
  the files contain SR Linux configuration content.

### CLI operations

- **`test-cli-show-version`** — `tests/playbooks/cli-show-version.yml`:
  runs `show version` in structured output mode and validates the architecture,
  then repeats it with text output and checks the serial-number text.
- **`test-cli-wrong-cmd`** — `tests/playbooks/cli-wrong-cmd.yml`: runs an
  invalid CLI command and verifies the expected parsing error.

### TLS and certificate handling

- **`test-tls-fail`** — `tests/playbooks/tls-missed-check-fail.yml`: enables
  HTTPS without certificate verification bypass or a CA file and verifies
  `CERTIFICATE_VERIFY_FAILED`.
- **`test-tls-skip`** — `tests/playbooks/tls-skipped-check.yml`: enables HTTPS
  with certificate validation disabled and verifies the request succeeds.
- **`test-tls-custom-ca`** — `tests/playbooks/tls-with-custom-ca.yml`: enables
  HTTPS with the topology-generated CA certificate and verifies the request
  succeeds.

### Configuration changes

- **`test-set-check-mode`** — `tests/playbooks/set-check.yml`: exercises
  configuration check mode with and without diff mode, verifies a prepared
  diff is returned, and checks that the device is unchanged.
- **`test-set-leaves`** — `tests/playbooks/set-leaves.yml`: updates system
  information and an interface description, then verifies all resulting
  values.
- **`test-set-leaves-twice`** — `tests/playbooks/set-leaves-twice.yml`: applies
  the same configuration twice and verifies the second run reports no change
  and creates no additional commit.
- **`test-set-wrong-value`** — `tests/playbooks/set-wrong-value.yml`: submits
  an unknown leaf and verifies the expected schema error.
- **`test-set-multiple-paths`** — `tests/playbooks/set-multiple-paths.yml`:
  exercises `update`, `replace`, and `delete` operations in one task and
  verifies all three results.
- **`test-set-interface`** — `tests/playbooks/set-interface.yml`: creates and
  configures an Ethernet interface, subinterface, and IPv4 address, then
  verifies the interface state.
- **`test-set-tools`** — `tests/playbooks/set-tools.yml`: writes through the
  tools datastore and verifies the resulting fan-tray locator state.
- **`test-delete-leaves`** — `tests/playbooks/delete-leaves.yml`: creates
  system-information leaves, deletes them, and verifies they are absent.
- **`test-set-idempotent`** — `tests/playbooks/set-idempotent.yml`: verifies
  an initial update changes the device and repeated updates report no change,
  both with and without diff mode.
- **`test-replace-full-config`** — `tests/playbooks/replace-full-cfg.yml`:
  replaces the complete configuration from a golden template and verifies an
  expected interface description.
- **`test-commit-confirm`** — `tests/playbooks/set-confirm-timeout.yml`:
  verifies that an unconfirmed commit reverts after its timeout and that a
  confirmed commit persists.
- **`test-replace-json-rpc-config`** — `tests/playbooks/replace-json-rpc-cfg.yml`:
  replaces JSON-RPC server configuration, allows the server restart, and
  verifies the resulting source address.

## Optional OpenConfig tests

The `_run-tests` function runs these tests only when the `oc-tests` argument is
provided. They are not part of the default GitHub Actions invocation.

- **`test-get-oc-container`** — `tests/playbooks/get-oc-container.yml`:
  retrieves an OpenConfig hostname and an SR Linux system-information path in
  one request.
- **`test-set-oc-leaf`** — `tests/playbooks/set-oc-leaf.yml`: updates the
  OpenConfig system MOTD banner and verifies it through an OpenConfig lookup.
- **`test-oc-validate`** — `tests/playbooks/oc-validate.yml`: validates a
  valid OpenConfig change and verifies rejection of an invalid leaf.

## Tests defined but not in the default suite

`run.sh` also defines `test-cli-put-file`, which runs
`tests/playbooks/cli-put-file.yml`. That playbook currently only templates a
golden configuration file and is not called by `_run-tests`.

`test-validate` is also defined but not called by `_run-tests`. It runs
`tests/playbooks/validate.yml`, which validates a valid OpenConfig change set
and verifies that an invalid leaf is rejected with the expected schema error.

The `test-backup-cfg` helper is also defined separately, but the default suite
uses `test-config-backup`, which runs the same `backup-cfg.yml` playbook.

## Ansible sanity checks

Sanity checks validate collection structure, documentation, imports,
compilation, Python style, YAML style, module metadata, runtime metadata,
symlinks, shell usage, and other Ansible collection conventions. They do not
run the SR Linux functional playbooks.

Version-specific ignore files are stored under `tests/sanity/ignore-*.txt`.
They contain only exceptions accepted for the corresponding Ansible core
version.
