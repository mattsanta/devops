---
name: cloud-deploy-pipeline-designer
description: >
  Design and create Cloud Deploy delivery pipelines to deploy applications to Cloud Run and Google Kubernetes Engine (GKE). Use when users want to deploy their application to multiple environments (e.g. dev and prod), require approvals for promoting their application to production, configure automatic rollbacks, or leverage deployment strategies (i.e. canary).
---

# Cloud Deploy Pipeline Designer

## Overview

This skill helps you design and create Cloud Deploy delivery pipelines. It guides you through defining targets, configuring promotion sequences, enabling automatic rollbacks, and setting up advanced deployment strategies like canary deployments.

## Constraints & Rules

1.  **NO PLACEHOLDERS**: Never generate YAML with placeholders like `<PROJECT_ID>`. Ask the user for values first.
2.  **Context First**: Always check existing files and conversation context before asking.
3.  **Step-by-Step**: Perform the steps one at a time. The goal is to guide the user through designing a delivery pipeline.

## Workflow: Designing a Pipeline

Follow these steps to create a comprehensive Cloud Deploy pipeline.

### Step 0: Prerequisites

**Required Context**: Before generating ANY configuration for this workflow, you **MUST** have the following values. Ask the user strictly for any missing information:
- **Project ID**: The Google Cloud project ID for the Cloud Deploy resources.
- **Region**: The region for the Cloud Deploy resources (e.g., `us-central1`).
- **Application Name**: The name of the application that will be deployed. Use the application name to generate the names of the Cloud Deploy resources, such as `DeliveryPipeline` and `Target` resources.
- **Runtime**: Either Cloud Run or Google Kubernetes Engine (GKE).
  - **If Cloud Run**: The Cloud Run project and location.
  - **If GKE**: The GKE cluster name.

### Step 1: Define the target environments

1. Identify the number of environments (e.g., dev, staging, production).
2. Identify if promotions should require user approval.
3. Define each of the environments as Cloud Deploy `Target` resources in a `clouddeploy.yaml` file. 
    - Use `references/configure-targets.md` as a reference when generating the resource YAML.
    - Use the application name provided by the user when naming the Cloud Deploy `Target` resources. For example, if the user wants to deploy an application named "hello-world" to a test and production environment then use "hello-world-test" and "hello-world-prod" as the `Target` IDs.

### Step 2: Define the delivery pipeline

1. Identify whether the user wants to use a canary deployment strategy for any of the target environments.
2. Define the Cloud Deploy `DeliveryPipeline` in the `clouddeploy.yaml` file.
    - Use `references/configure-pipelines.md` as a reference when generating the resource YAML.
    - Use application name as the `DeliveryPipeline` ID.

### Step 3: Define automations

1. Identify whether the user wants to automatically rollback if any failures occur during the rollout.
2. **If the user specified multiple environments in the previous step** 
  - Identify if they want automatic promotions between environments.
3. **If the user specified a canary deployment strategy in the previous step** 
  - Identify if they want to automatically advance the rollout through the phases after a wait period.
4. Define the Cloud Deploy `Automation` resources in the `clouddeploy.yaml` file.
  - Use `references/configure-automations.md` as a reference when generating the resource YAML.

### Step 4: Validate the clouddeploy.yaml file

Ensure that the `clouddeploy.yaml` file is valid. See https://docs.cloud.google.com/deploy/docs/config-files for the schema.

### Step 5: Create the delivery pipeline

Run the following command to create the Cloud Deploy `DeliveryPipeline` and associated resources, using the values collected in Step 0:

```bash
gcloud deploy apply --file=clouddeploy.yaml --region=<REGION> --project=<PROJECT_ID>
```

### Step 6: Create a skaffold.yaml file and runtime manifests

**Required Context**: Before generating a `skaffold.yaml` file, you **MUST** know if the user has manifests for the runtime they are deploying to.

1. **If the user does not have runtime manifests**: Generate some basic ones based on the runtime.
    - **If Cloud Run**: Generate a Cloud Run manifest. Use `references/basic-cloudrun-manifests.md` as a reference.
    - **If GKE**: Generate a Kubernetes `Deployment` and `Service `manifest. Use `references/basic-k8s-manifests.md` as a reference.
2. Create a `skaffold.yaml` file required to create a Cloud Deploy `Release` for the `DeliveryPipeline`.
    - Use `references/configure-skaffold.md` as a reference when generating the `skaffold.yaml` file.
