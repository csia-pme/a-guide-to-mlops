---
title: "A guide to MLOps"
---

# {% $markdoc.frontmatter.title %}

## Introduction & Problematic

This guide will help you through incremental steps from a traditional approach of managing ML projects to a modern ML Ops approach designed to improve team collaboration and reproducibility.

It has been written using many different sources that will be mentioned in each step of the process.

## Get started

Not sure where to start? Check the following links to get started!

- [What is MLOps?](/get-started/what-is-mlops)
- [What problems is MLOps trying to solve?](/get-started/what-problems-is-mlops-trying-to-solve)
- [Why would MLOps be useful for me?](/get-started/why-would-mlops-be-useful-for-me)
- [The tools used in this guide](/get-started/the-tools-used-in-this-guide)
- [Start the guide](#the-guide)

## The guide

- [Introduction](/the-guide/introduction)
- [Step 1: Run a simple ML experiment](/the-guide/step-1-run-a-simple-ml-experiment)
- [Step 2: Share your ML experiment code with Git](/the-guide/step-2-share-your-ml-experiment-code-with-git)
- [Step 3: Share your ML experiment data with DVC](/the-guide/step-3-share-your-ml-experiment-data-with-dvc)
- [Step 4: Reproduce the experiment with DVC](/the-guide/step-4-reproduce-the-experiment-with-dvc)
- [Step 5: Track model evolutions with DVC](/the-guide/step-5-track-model-evolutions-with-dvc)
- [Step 6: Orchestrate the workflow with a CI/CD pipeline](/the-guide/step-6-orchestrate-the-workflow-with-a-cicd-pipeline)
- [Step 7: Track model evolutions in the CI/CD pipeline with CML](/the-guide/step-7-track-model-evolutions-in-the-cicd-pipeline-with-cml)
- [Step 8: Serve the model with MLEM](/the-guide/step-8-serve-the-model-with-mlem)
- [Conclusion](/the-guide/conclusion)

## Label Studio

- [Introduction](/label-studio/introduction)
- [Create a Label Studio project](/label-studio/create-a-label-studio-project)
- [Import existing data to Label Studio](/label-studio/import-existing-data-to-label-studio)
- [Annotate new data with Label Studio](/label-studio/annotate-new-data-with-label-studio)
- [Export data from Label Studio](/label-studio/export-data-from-label-studio)

## Advanced concepts

- [Convert the experiment data from the guide for Label Studio](/advanced-concepts/convert-the-experiment-data-from-the-guide-for-label-studio)
- [Deploy MinIO](/advanced-concepts/deploy-minio)
- [Deploy Label Studio](/advanced-concepts/deploy-label-studio)
- [Link your ML model with Label Studio](/advanced-concepts/link-your-ml-model-with-label-studio)
- [Train the model on a Kubernetes cluster with CML](/advanced-concept)

## Known limitations

- [CML: Cannot create a runner every time](/known-limitations/cml-cannot-create-a-runner-every-time)
- [CML: Cannot specify an affinity to run the pod on Kubernetes](/known-limitations/cml-cannot-specify-an-affinity-to-run-the-pod-on-kubernetes)
- [DVC & Git: Data and code cannot evolve independently](/known-limitations/dvc-git-data-and-code-cannot-evolve-independently)
- [Global: Missing elements in comparison to other user-friendly solutions](/known-limitations/global-missing-elements-in-comparison-to-other-user-friendly-solutions)
- [Label Studio: Does the predictions made by our ML model really help the person annotating the dataset](/known-limitations/label-studio-does-the-predictions-made-by-our-ml-model-really-help-the-person-annotating-the-dataset)
- [Label Studio: The export of the dataset is manual](/known-limitations/label-studio-the-export-of-the-dataset-is-manual)
- [Label Studio: The retraining of the ML model is difficult](/known-limitations/label-studio-the-retraining-of-the-ml-model-is-difficult)
