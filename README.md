# Stacksmith application example

This application owns a stack and dev/prod inputs. Its `private_bucket` implementation comes from an immutable commit in [the platform example](https://github.com/ci-sourcerer/stacksmith-example-platform). Configuration and module changes are reviewed there; adoption is reviewed here as a pin change.

## Prepare remote state

Use a dedicated S3 state bucket in `us-east-1`, separate from both application buckets. Create it through your account bootstrap process, enable versioning and encryption, and block public access. Give the execution identity access to that state bucket and permission to manage the example S3 resources. Replace the bucket placeholder in `backend.yaml` and choose globally unique names in `vars/dev.yaml` and `vars/prod.yaml`.

## Run locally

Authenticate to AWS before running these commands from this repository. With an interactive root profile, use `aws login --profile root` and export its credentials into the execution environment. Prefer a scoped role for ongoing CI use.

```sh
stacksmith ci environments --gitops-root . --discovery-mode auto
stacksmith validate --runfile common/stacksmith.yaml --runfile environments/dev.yaml
stacksmith plan --build-dir .stacksmith/dev --runfile common/stacksmith.yaml --runfile environments/dev.yaml
stacksmith apply --build-dir .stacksmith/dev --runfile common/stacksmith.yaml --runfile environments/dev.yaml
```

Repeat with `prod` to inspect its separate target. Never reuse application bucket names between environments.

## Platform onboarding

Connect this repository, select its root deployment, and choose manual runs initially. Configure environment credentials and permissions. Preview the shared stack in both deployment contexts, plan dev, review the changes, and approve execution. Platform supports one deployment root per repository.

## Cleanup

Empty application buckets if objects were added. Run `stacksmith destroy` with the same build directory and runfiles for each environment. Verify both buckets are removed before deleting remote state. Retain state versions until cleanup is confirmed; remove the bootstrap bucket last through your bootstrap process.

## Optional CI execution

The manual deployment workflow pins the Stacksmith workflow commit. Configure AWS credentials and reviewed-plan encryption secrets required by that workflow before dispatch. Its execution image follows the Stacksmith reusable workflow configuration; select a compatible image version through `STACKSMITH_IMAGE_VERSION`. No push or pull request automatically deploys resources.
