# Kargo Merge Methods Example

This example demonstrates end-to-end testing of the `mergeMethod` configuration option
in the `git-merge-pr` promotion step. It showcases three different Git merge strategies
that can be used when automatically merging pull requests during Kargo promotions.

## Prerequisites

- Kubernetes cluster with Kargo installed
- Git credentials configured in Kargo

## Deploy

```kubectl apply -k kargo```
