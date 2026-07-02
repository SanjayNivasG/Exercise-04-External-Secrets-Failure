# 🔐 Exercise 04 – External Secrets Failure Investigation (AWS EKS)

## 📌 Project Overview

This project demonstrates how to investigate and troubleshoot an **External Secrets synchronization failure** in an Amazon EKS cluster. The application failed to start because the required database password was not injected from AWS Secrets Manager into Kubernetes.

The objective was to identify whether the issue originated from **AWS**, **Kubernetes**, or the **Secret configuration**, and perform a systematic production-style root cause analysis.

---

# 🏗️ Architecture

```
Application Pod
      │
      ▼
External Secret
      │
      ▼
SecretStore
      │
      ▼
AWS Secrets Manager
      │
      ▼
IAM Authentication
```

---

# 🚨 Incident

Application startup failed with the following error:

```
FATAL:
Database password not found
Environment Variable DB_PASSWORD missing
```

External Secret status:

```
READY=False
```

Controller error:

```
SecretSyncedError

AccessDeniedException:
User is not authorized to perform:
secretsmanager:GetSecretValue
```

---

# 🎯 Objective

* Install External Secrets Operator
* Configure AWS Secrets Manager integration
* Create SecretStore and ExternalSecret
* Reproduce the production failure
* Investigate the issue
* Identify the root cause
* Document the findings

---

# 🛠️ Technologies Used

* Amazon EKS
* Kubernetes
* AWS Secrets Manager
* External Secrets Operator
* IAM
* Helm
* AWS CLI
* kubectl

---

# 📂 Project Structure

```
Exercise-04-External-Secrets-Failure
│
├── manifests
│   ├── secretstore.yaml
│   └── externalsecret.yaml
│
├── screenshots
│
└── README.md
```

---

# 🔍 Investigation Steps

## 1. Verified Cluster Health

```bash
kubectl get nodes
kubectl get pods -A
```

Verified that the Kubernetes cluster and External Secrets Operator were running successfully.

---

## 2. Verified External Secret

```bash
kubectl get externalsecret -n payment
```

Output:

```
READY=False
STATUS=SecretSyncedError
```

---

## 3. Described External Secret

```bash
kubectl describe externalsecret database-secret -n payment
```

Observed:

```
AccessDeniedException

secretsmanager:GetSecretValue
```

---

## 4. Checked Controller Logs

```bash
kubectl logs deployment/external-secrets \
-n external-secrets
```

The controller repeatedly attempted to retrieve the secret from AWS Secrets Manager but received an authorization failure.

---

## 5. Verified AWS Secret

```bash
aws secretsmanager get-secret-value \
--secret-id database-secret
```

Confirmed that the secret existed and contained the expected password.

---

# 🔎 Root Cause Analysis

## AWS Issue ✅

The EKS node IAM role did not have permission to perform:

```
secretsmanager:GetSecretValue
```

As a result, External Secrets could not retrieve the secret from AWS Secrets Manager.

---

## Kubernetes Issue ✅

No Kubernetes issue was identified.

Verified:

* Cluster healthy
* Pods running
* SecretStore created
* External Secret created
* Controller functioning correctly

---

## Secret Issue ✅

The secret existed in AWS Secrets Manager and contained valid data.

The issue was not with the secret itself but with the IAM permissions required to access it.

---

# 📈 Failure Flow

```
Application

↓

External Secret

↓

SecretStore

↓

AWS Secrets Manager

↓

IAM Permission Missing

↓

AccessDeniedException

↓

Secret Not Synced

↓

READY=False

↓

Application Startup Failure
```

---

# ✅ Resolution

Grant the required IAM permission:

```
secretsmanager:GetSecretValue
```

After updating IAM permissions:

* External Secret synchronizes successfully.
* Kubernetes Secret is created.
* Application receives `DB_PASSWORD`.
* Application starts successfully.

---

# 📸 Screenshots

* Cluster Running
* External Secrets Operator Running
* ExternalSecret READY=False
* Describe ExternalSecret
* Controller Logs
* AWS Secret Verification
* Project Structure

---

# 💡 Key Learnings

* External Secrets Operator architecture
* AWS Secrets Manager integration
* SecretStore configuration
* ExternalSecret synchronization
* IAM permission troubleshooting
* Kubernetes debugging
* Production incident investigation
* Root Cause Analysis (RCA)

---

# 🏆 Skills Demonstrated

* Kubernetes Troubleshooting
* Amazon EKS
* AWS IAM
* AWS Secrets Manager
* External Secrets Operator
* Production Incident Response
* Root Cause Analysis
* DevOps Debugging
* Cloud Security
* Infrastructure Troubleshooting

---

# 📚 Commands Used

```bash
kubectl get externalsecret -n payment

kubectl describe externalsecret database-secret -n payment

kubectl logs deployment/external-secrets -n external-secrets

aws secretsmanager get-secret-value --secret-id database-secret

kubectl get secret -n payment
```
