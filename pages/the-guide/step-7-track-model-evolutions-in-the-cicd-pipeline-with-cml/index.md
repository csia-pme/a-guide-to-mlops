---
title: "Step 7: Track model evolutions in the CI/CD pipeline with CML"
---

# {% $markdoc.frontmatter.title %}

## Summary

The purpose of this step is to track the model evolutions and generate reports directly in the CI/CD pipeline so it can be discussed collectively online before commiting the changes to the codebase.

{% callout type="note" %}
CML can do much more than just generating reports. Have a look to the [Train the model on a Kubernetes cluster with CML](/advanced-concepts/train-the-model-on-a-kubernetes-cluster-with-cml) guide.
{% /callout %}

## Instructions

{% callout type="warning" %}
This guide has been written for macOS and Linux operating systems in mind. If you use Windows, you might encounter issues. Please use a decent terminal ([GitBash](https://gitforwindows.org/) for instance) or a Windows Subsystem for Linux (WSL) for optimal results.
{% /callout %}

The reports generated by CML compare the current run with a target reference.

The target reference can be a specific commit (what are the differences between the current run and the run from the target commit) or a branch (what are the differences between the current run and the run from the target branch).

Many workflows exist to discuss and integrate the work done to a target reference. For the sake of simplicity, in this guide, we will present two methods that are commonly used on GitHub - Pull Requests (PRs) - and GitLab - Merge Requests (MRs) - to integrate the work done to the `main` branch.

### Update GitHub Actions and create pull request

{% callout type="note" %}
Using GitLab? Go to the [Update GitLab CI and create merge request](#update-gitlab-ci-and-create-merge-request) section!
{% /callout %}

{% callout type="note" %}
Highly inspired by the [_Get Started with CML on GitHub_ - cml.dev](https://cml.dev/doc/start/github) [_Creating an issue_ - docs.github.com](https://docs.github.com/en/issues/tracking-your-work-with-issues/creating-an-issue) and [_Creating a branch to work on an issue_ - docs.github.com](https://docs.github.com/en/issues/tracking-your-work-with-issues/creating-a-branch-for-an-issue) guides and the [`example_cml` - github.com](https://github.com/iterative/example_cml) and [`cml_dvc_case` - github.com](https://github.com/iterative/cml_dvc_case) GitHub repositories.
{% /callout %}

Update the `.github/workflows/mlops.yml` file.

```yaml
name: MLOps Step 7

on:
  # Runs on pushes targeting main branch
  push:
    branches:
      - main

  # Runs on pull requests
  pull_request:

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

jobs:
  train:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
          cache: 'pip'
      - name: Setup DVC
        uses: iterative/setup-dvc@v1
        with:
          version: '2.37.0'
      - name: Login to Google Cloud
        uses: 'google-github-actions/auth@v1'
        with:
          credentials_json: '${{ secrets.GCP_SERVICE_ACCOUNT_KEY }}'
      - name: Train model
        run: |
          # Install dependencies
          pip install --requirement src/requirements.txt
          # Pull data from DVC
          dvc pull
          # Run the experiment
          dvc repro
      - name: Upload evaluation results
        uses: actions/upload-artifact@v3
        with:
          path: evaluation
          retention-days: 5

  report:
    needs: train
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3
        with:
          ref: ${{ github.event.pull_request.head.sha }}
      - name: Download evaluation results
        uses: actions/download-artifact@v3
      - name: Copy evaluation results
        shell: bash
        run: |
          # Delete current evaluation results
          rm -rf evaluation
          # Replace with the new evaluation results
          mv artifact evaluation
      - name: Setup DVC
        uses: iterative/setup-dvc@v1
        with:
          version: '2.37.0'
      - name: Setup CML
        uses: iterative/setup-cml@v1
        with:
          version: '0.18.1'
      - name: Create CML report
        env:
          REPO_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          # Fetch all other Git branches
          git fetch --depth=1 origin main:main

          # Compare parameters to main branch
          echo "# Params: workflow vs. main" >> report.md
          echo >> report.md
          dvc params diff main --show-md >> report.md
          echo >> report.md

          # Compare metrics to main branch
          echo "# Metrics: workflow vs. main" >> report.md
          echo >> report.md
          dvc metrics diff main --show-md >> report.md
          echo >> report.md

          # Create plots
          echo "# Plots" >> report.md
          echo >> report.md

          echo "## Precision recall curve" >> report.md
          echo >> report.md
          dvc plots diff \
            --target evaluation/plots/prc.json \
            -x recall \
            -y precision \
            --show-vega main > vega.json
          vl2png vega.json > prc.png
          echo '![](./prc.png "Precision recall curve")' >> report.md
          echo >> report.md

          echo "## Roc curve" >> report.md
          echo >> report.md
          dvc plots diff \
            --target evaluation/plots/sklearn/roc.json \
            -x fpr \
            -y tpr \
            --show-vega main > vega.json
          vl2png vega.json > roc.png
          echo '![](./roc.png "Roc curve")' >> report.md
          echo >> report.md

          echo "## Confusion matrix" >> report.md
          echo >> report.md
          dvc plots diff \
            --target evaluation/plots/sklearn/confusion_matrix.json \
            --template confusion \
            -x actual \
            -y predicted \
            --show-vega main > vega.json
          vl2png vega.json > confusion_matrix.png
          echo '![](./confusion_matrix.png "Confusion Matrix")' >> report.md
          echo >> report.md

          # Publish the CML report
          cml comment create --pr --publish report.md
```

This GitHub Workflow will create CML reports on each pushes that are related to a pull request.

Push the changes to Git.

```sh
# Add the updated workflow
git add .github/workflows/mlops.yml

# Commit the changes
git commit -m "Enable CML reports on pull requests"

# Push the changes
git push
```

Create a new issue by going to the **Issues** section from the top header of your GitHub repository. Select **New issue** and describe the work/improvements/ideas that you want to integrate to the codebase. In this guide, we will name the issue _Demonstrate step 7_. Create the issue by selecting **Submit new issue**.

The issue opens. Select **Create a branch for this issue or link a pull request** from the right sidebar. Create the branch by selecting **Create branch**. A new pop-up opens with the name of the branch you want to checkout to.

Go to the **Code** section from the top header of your GitHub repository. Select **_N_ branches** (where _N_ is the current number of branches your repository has). Next to the newly created branch, select **New pull request**. Name the pull request and select **Create pull request** to create the pull request.

{% callout type="note" %}
Finished? Go to the [Make changes to the model](#make-changes-to-the-model) section!
{% /callout %}

### Update GitLab CI and create merge request

{% callout type="note" %}
Using GitHub? Go to the [Update GitHub Actions and create pull request](#update-github-actions-and-create-pull-request) section!
{% /callout %}

{% callout type="note" %}
Highly inspired by the [_Using CML on GitLab_ - cml.dev](https://cml.dev/doc/start/gitlab) and [_Personal access tokens_ - docs.gitlab.com](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html) guides and the [`example_cml` - gitlab.com](https://gitlab.com/iterative.ai/example_cml) and the [`cml-dvc-case` - gitlab.com](https://gitlab.com/iterative.ai/cml-dvc-case) GitLab repositories.
{% /callout %}

_TODO_

{% callout type="note" %}
Finished? Go to the [Make changes to the model](#make-changes-to-the-model) section!
{% /callout %}

### Make changes to the model

Checkout the branch locally.

```sh
# Get the latest updates from the remote origin
git fetch origin

# Check to the new branch
git checkout <the name of the new branch>
```

Update the parameters to run the experiment in the `params.yaml` file.

```yaml
prepare:
  split: 0.25
  seed: 20170428

featurize:
  max_features: 500
  ngrams: 3

train:
  seed: 20170428
  n_est: 50
  min_split: 3
```

Run the experiment.

```sh
# Run the experiment. DVC will automatically run all required steps
dvc repro
```

Push the changes to DVC and Git.

```sh
# Upload the experiment data and cache to the remote bucket
dvc push

# Add all the files
git add .

# Commit the changes
git commit -m "I made some changes to the model"

# Push the changes
git push
```

Congrats! You now have a report that will be generated on each PR/MR that can be visualized and discussed with the rest of the team before integrating the changes to the common codebase. When you are happy, you can merge the PR/MR on the target branch.

### GitHub: Merge the changes

{% callout type="note" %}
Using GitLab? Go to the [GitLab: Merge the changes](#gitlab-merge-the-changes) section!
{% /callout %}

Go back to the pull request. At the end of the page, select **Merge pull request**. Confirm the merge by selecting **Confirm merge**.

The associated issue will be automatically closed as well and you can iterate on your model while keeping a trace of the improvements made to it.

{% callout type="note" %}
Finished? Go to the [Check the results](#check-the-results) section!
{% /callout %}

### GitLab: Merge the changes

{% callout type="note" %}
Using GitHub? Go to the [GitHub: Merge the changes](#github-merge-the-changes) section!
{% /callout %}

_TODO_

{% callout type="note" %}
Finished? Go to the [Check the results](#check-the-results) section!
{% /callout %}

## Check the results

Want to see what the result of this step should look like? Have a look at the Git repository directory here: [step-7-track-model-evolutions-in-the-cicd-pipeline-with-cml](https://github.com/csia-pme/a-guide-to-mlops/tree/main/pages/the-guide/step-7-track-model-evolutions-in-the-cicd-pipeline-with-cml).

## State of the MLOps process

_TODO_

## Next & Previous steps

- **Previous**: [Step 6: Orchestrate the workflow with a CI/CD pipeline](/the-guide/step-6-orchestrate-the-workflow-with-a-cicd-pipeline)
- **Next**: [Step 8: Share and deploy model with MLEM](/the-guide/step-8-share-and-deploy-model-with-mlem)