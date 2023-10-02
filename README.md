# Github Status Checks Workaround For CircleCI's Dynamic Config

## What does this solve?

### Background

Github has a feature in their branch protections that allows for selected [status checks](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/collaborating-on-repositories-with-code-quality-features/about-status-checks) to be required for a pull request to be merged into a branch.

### Problem

When a workflow is not run within CircleCI, there is no status check provided to Github. There is no skipped status that CircleCI sends when a workflow is not run due to [dynamic config](https://circleci.com/docs/dynamic-config/) in CircleCI. This can result in the PR being unable to merge as the required checks aren't satisfied.

### Solution

CircleCI provides status checks for workflows and individual jobs. Github is looking for a status check from CircleCI with the same _name_ as the required check. Selecting the final job(s) in workflows to be required for status checks, instead of selecting the full workflow, allows for a stand-in job with the same name to be used to satisfy the check. A stand-in workflow with a stand-in job, that has the same name as the final job in the original workflow, can be run.

This can be simplified further by having a single job at the end of each workflow that is fanned into.

## Pros and Cons

### Pros 

- Easy to implement
- Prevents human error by being fully automated
- Stand-in workflow and job make it clear what is happening in both the UI and the CircleCI config
- Setup config can be locked down with [config policies](https://circleci.com/docs/config-policy-management-overview/)

### Cons

- Requires a job to be spun up to satisfy the check
- Increases status check management overhead if workflow cannot be fanned back into a single or a small amount of jobs

## Demo

### Branch Protection Settings

Merging into the main branch requires the following status checks:

<img width="726" alt="Screenshot 2023-10-02 at 10 26 48 AM" src="https://github.com/felixshiftellecon/circleci-github-status-checks-workaround/assets/57482070/7574c745-3fe0-4199-a7e1-7d466f43115c">

<br>
<br>

---

<br>
<br>

### Setup Workflow

The CircleCI config uses the [path filtering orb to set parameters](https://github.com/felixshiftellecon/circleci-github-status-checks-workaround/blob/main/.circleci/config.yml#L24) to `true` if a change is made to any file within one of the three directories present. There is one parameter per directory.

A [script is run after](https://github.com/felixshiftellecon/circleci-github-status-checks-workaround/blob/main/.circleci/config.yml#L31) the parameters are set to check for any existing parameters. If any of the three parameters were not set to `true` then a stand-in parameter is set to `true`.

The [downstream workflows are then triggered](https://github.com/felixshiftellecon/circleci-github-status-checks-workaround/blob/main/.circleci/config.yml#L59).

<br>
<br>

---

<br>
<br>

### Files Changed Workflows

When files are changed within the directories, any parameters that were set to `true` [run their normal workflows](https://github.com/felixshiftellecon/circleci-github-status-checks-workaround/blob/main/.circleci/continued_config.yml#L64) and the checks are satisfied. Below is a pull request showing a file being changed in each of the three directories, triggering their normal workflows and satisfying the checks.

- [Files Changed Pull Request](https://github.com/felixshiftellecon/circleci-github-status_checks-workaround/pull/4)
- [Files Changed CircleCI Pipelines](https://app.circleci.com/pipelines/github/felixshiftellecon/circleci-github-status-checks-workaround?branch=files-changed)

<img width="854" alt="Screenshot 2023-10-02 at 10 59 25 AM" src="https://github.com/felixshiftellecon/circleci-github-status-checks-workaround/assets/57482070/1ed063d1-875b-4f62-aee9-782fa1b0fd0c">
<img width="1024" alt="Screenshot 2023-10-02 at 10 59 58 AM" src="https://github.com/felixshiftellecon/circleci-github-status-checks-workaround/assets/57482070/adf97df8-3350-42dd-b44b-ec3af1620263">

<br>
<br>

---

<br>
<br>

### No Changes Workflows

Any parameters that were _not_ set to `true` have a stand-in parameter set to `true` which [triggers a stand-in workflow](https://github.com/felixshiftellecon/circleci-github-status-checks-workaround/blob/main/.circleci/continued_config.yml#L86). The stand-in workflow contains a job with the same name as the required check, which satisfies it. Below is a pull request showing no files being changed, triggering stand-in workflows to satisfy the checks.

- [No File Changes Pull Request]( https://github.com/felixshiftellecon/circleci-github-status-checks-workaround/pull/6)
- [No File Changes Pipelines](https://app.circleci.com/pipelines/github/felixshiftellecon/circleci-github-status-checks-workaround?branch=no-changes-to-files)

<img width="849" alt="Screenshot 2023-10-02 at 11 00 20 AM" src="https://github.com/felixshiftellecon/circleci-github-status-checks-workaround/assets/57482070/fdacea96-d9df-4df0-b3d6-5ac326f6ec9f">
<img width="453" alt="Screenshot 2023-10-02 at 11 00 47 AM" src="https://github.com/felixshiftellecon/circleci-github-status-checks-workaround/assets/57482070/1eeacd3d-4b63-4c37-8a74-e664461a1a3f">

<br>
<br>

## Useful Links

- [Github's Status Checks](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/collaborating-on-repositories-with-code-quality-features/about-status-checks)
- [CircleCI's Dynamic Config](https://circleci.com/docs/dynamic-config/)
- [CircleCI Feature Request to Better Support Status Checks](https://ideas.circleci.com/cloud-feature-requests/p/support-utilizing-checks-with-dynamic-configs)
- [Alternative Workaround: Exiting early](https://github.com/kelvintaywl-cci/dynamic-config-showcase/tree/main)
