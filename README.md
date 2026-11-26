<!--
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: 2025 The Linux Foundation
-->

# 🏷️ Verify Pushed Tag

> [!WARNING]
> The maintainers have deprecated this action, and it gets no further
> changes or releases.
> This repository is due for archiving; references to its existing tags
> and commits keep working after that.
> [`lfreleng-actions/tag-validate-action`](https://github.com/lfreleng-actions/tag-validate-action)
> replaces it: see
> [Migrating to tag-validate-action](#migrating-to-tag-validate-action).

Verifies the action/workflow event trigger was a tag push event.

Optionally, checks tag compliance with a given versioning type.

## tag-push-verify-action

## Supported Versioning Types

Checks the pushed tag against these versioning types:

- [Semantic Versioning](https://semver.org/)
- [Calendar Versioning](https://calver.org/)

Can optionally disable versioning compliance checks.

## Usage Example

<!-- markdownlint-disable MD046 -->

```yaml
steps:
  - name: 'Verify Pushed Tag'
    uses: lfreleng-actions/tag-push-verify-action@main
    with:
      versioning: 'calver'
      exit_on_fail: 'false'
```

<!-- markdownlint-enable MD046 -->

## Tag

The validated tag comes from: `${{ github.ref_name }}`

## Inputs

 <!-- markdownlint-disable MD013 -->

| Input Name   | Required | Default | Description                                                    |
| ------------ | -------- | ------- | -------------------------------------------------------------- |
| versioning   | False    | semver  | Check tag compliance with versioning type [semver/calver/none] |
| exit_on_fail | False    | true    | Exit with error if tag fails versioning compliance check       |

<!-- markdownlint-enable MD013 -->

The action will always exit with an error if a tag push event did not trigger
the action/workflow run. When "exit_on_fail" set false, the versioning
type/compliance check generates a warning rather than an error.

## Versioning Types

The "versioning" input string must match one of the options below:

 <!-- markdownlint-disable MD013 -->

| Versioning | Description                 |
| ---------- | --------------------------- |
| semver     | Semantic versioning         |
| calver     | Calendar versioning         |
| none       | No tag validation performed |

<!-- markdownlint-enable MD013 -->

## Outputs

<!-- markdownlint-disable MD013 -->

| Output Name | Description                                                |
| ----------- | ---------------------------------------------------------- |
| tag_push    | Set true when workflow/job trigger was a tag push event    |
| valid       | Set based on the validation results: true/false/untested   |
| tag         | Set to the pushed tag value ${{ github.ref_name }}         |
| dev_version | Set true when tag contains pre-release/development strings |

Note: also exports the pushed tag to the environment.

<!-- markdownlint-enable MD013 -->

## Migrating to tag-validate-action

Replace the step from the usage example above with a checkout followed
by a call to `tag-validate-action`. Pin both actions by commit SHA:

<!-- markdownlint-disable MD013 MD046 -->

```yaml
steps:
  - name: 'Checkout repository'
    uses: actions/checkout@0000000000000000000000000000000000000000  # REPLACE: vX.Y.Z
    with:
      fetch-depth: 0

  - name: 'Verify Pushed Tag'
    id: tag-validate
    uses: lfreleng-actions/tag-validate-action@0000000000000000000000000000000000000000  # REPLACE: vX.Y.Z
    with:
      require_type: 'calver'
```

<!-- markdownlint-enable MD013 MD046 -->

### Input Mapping

<!-- markdownlint-disable MD013 -->

| This action                               | tag-validate-action                         |
| ----------------------------------------- | ------------------------------------------- |
| `versioning` (`semver`, `calver`, `none`) | `require_type`, with the same three values  |
| `exit_on_fail`                            | None; see the differences below             |

<!-- markdownlint-enable MD013 -->

### Output Mapping

<!-- markdownlint-disable MD013 -->

| This action   | tag-validate-action                       |
| ------------- | ----------------------------------------- |
| `tag_push`    | None; see the differences below           |
| `valid`       | `valid`                                   |
| `tag`         | `tag_name`                                |
| `dev_version` | `development_tag`                         |

<!-- markdownlint-enable MD013 -->

### Differences

1. **A failed validation fails the step.** Setting `exit_on_fail: 'false'`
   here reports a failure as a warning and carries on. The replacement
   has no such input: to carry on, set `continue-on-error: true` on the
   step and read the `valid` output.
2. **Runs not triggered by a tag push.** Given neither `tag_location`
   nor `tag_string`, the replacement validates the tag that triggered
   the run, and fails when there is none. Set `permit_missing: 'true'`
   to stop that failure. It has no `tag_push` output; to tell whether a
   tag push triggered the run, test `github.ref_type == 'tag'`.
3. **`valid` takes two values.** This action sets `valid` to `true`,
   `false` or `untested` (when `versioning` is `none`). The replacement
   never reports `untested`: with `require_type: none`, or with no
   `require_type` at all, it accepts any tag format.
4. **It needs the repository.** Check out the repository with
   `fetch-depth: 0` before calling it, as in the example above.
5. **It does more by default.** It uploads its result as a JSON
   artifact (`upload_json`, default `true`) and writes a job summary
   (`github_step_summary`, default `true`). Set both to `false` to turn
   these off.
6. **The pushed tag in the environment.** This action also exports the
   pushed tag to the environment. Read the `tag_name` output of the
   replacement instead.

The replacement can also check tag signatures, the branch, tag age and
version increments; its
[README](https://github.com/lfreleng-actions/tag-validate-action#readme)
describes those inputs.
