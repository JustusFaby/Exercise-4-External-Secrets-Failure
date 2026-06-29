# Exercise 4 – External Secrets Failure

## Overview

This project demonstrates how to troubleshoot an **External Secrets synchronization failure** in Kubernetes using the **External Secrets Operator** and **AWS Secrets Manager**.

The objective is to identify why an `ExternalSecret` is not syncing, investigate the issue using Kubernetes commands and controller logs, determine the root cause, apply the necessary fixes, and verify successful synchronization.

---

## Objective

* Deploy the External Secrets Operator.
* Configure a SecretStore to connect with AWS Secrets Manager.
* Create an ExternalSecret resource.
* Investigate synchronization failures.
* Identify the root cause.
* Fix the issue.
* Verify successful synchronization.

---

## Technologies Used

* Kubernetes (Minikube)
* Helm
* External Secrets Operator
* AWS Secrets Manager
* kubectl
* PowerShell

---

## Project Structure

```text
exercise-4/
│── secretstore.yaml
│── externalsecret.yaml
│── README.md
```

---

## Prerequisites

* Minikube
* kubectl
* Helm
* AWS Account
* AWS IAM User with:

  * SecretsManagerReadWrite (or equivalent permissions)
* AWS Secrets Manager Secret
* PowerShell

---

## Deployment Steps

### 1. Start Minikube

```powershell
minikube start
```

---

### 2. Install External Secrets Operator

```powershell
helm repo add external-secrets https://charts.external-secrets.io

helm repo update

helm install external-secrets external-secrets/external-secrets `
-n external-secrets `
--create-namespace
```

---

### 3. Create Namespace

```powershell
kubectl create namespace exercise4
```

---

### 4. Deploy SecretStore

```powershell
kubectl apply -f secretstore.yaml
```

---

### 5. Deploy ExternalSecret

```powershell
kubectl apply -f externalsecret.yaml
```

---

## Troubleshooting Performed

### Investigation Commands

```powershell
kubectl get externalsecret -n exercise4

kubectl describe externalsecret app-secret -n exercise4

kubectl get secretstore -n exercise4

kubectl describe secretstore aws-store -n exercise4

kubectl logs deployment/external-secrets -n external-secrets

kubectl get secrets -n exercise4
```

---

## Issues Identified

### Issue 1

**Problem**

ExternalSecret failed to synchronize.

**Root Cause**

The Kubernetes secret `aws-secret` containing AWS credentials was missing.

**Resolution**

Created the required Kubernetes secret.

```powershell
kubectl create secret generic aws-secret `
--from-literal=access-key=<AWS_ACCESS_KEY> `
--from-literal=secret-access-key=<AWS_SECRET_KEY> `
-n exercise4
```

---

### Issue 2

**Problem**

External Secrets Operator connected successfully but could not locate the AWS secret.

**Root Cause**

The SecretStore was configured with the wrong AWS region (`ap-south-1`), while the secret existed in `us-east-1`.

**Resolution**

Updated the SecretStore region to:

```yaml
region: us-east-1
```

Applied the updated configuration:

```powershell
kubectl apply -f secretstore.yaml
```

---

## Verification

### Verify SecretStore

```powershell
kubectl get secretstore -n exercise4
```

Expected:

```text
READY   True
```

---

### Verify ExternalSecret

```powershell
kubectl get externalsecret -n exercise4
```

Expected:

```text
READY   True
```

---

### Verify Kubernetes Secret

```powershell
kubectl get secrets -n exercise4
```

Expected:

```text
app-secret
aws-secret
```

---

### Decode Secret

```powershell
[System.Text.Encoding]::UTF8.GetString(
    [System.Convert]::FromBase64String(
        (kubectl get secret app-secret -n exercise4 -o jsonpath="{.data.password}")
    )
)
```

---

## Key Learnings

* Understanding the External Secrets Operator.
* Configuring SecretStore resources.
* Integrating Kubernetes with AWS Secrets Manager.
* Troubleshooting synchronization failures.
* Reading controller logs.
* Diagnosing Kubernetes configuration issues.
* Identifying AWS region mismatches.
* Validating successful secret synchronization.

---

## Demo Video

**Demo Link:**

```text
https://drive.google.com/file/d/1R3h2pRmVdwaXv7_n46wKPlX_8nE4gM_C/view?usp=sharing
```

---

## Cleanup

```powershell
kubectl delete namespace exercise4

helm uninstall external-secrets -n external-secrets

kubectl delete namespace external-secrets

minikube stop

minikube delete
```

---

## Author

**Justus Faby Jeyakumar**

GitHub: https://github.com/JustusFaby
