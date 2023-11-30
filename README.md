# Apigee CI/CD using apigeecli

## Setting up Workload Identity Federation

```sh
export PROJECT_ID="ssv-apigee-asm-demo"
export SERVICE_ACCOUNT="apigee-ci-cd"
export WORKLOAD_IDENTITY_POOL="apigee-ci-cd-pool"
export WORKLOAD_IDENTITY_POOL_PROVIDER="apigee-ci-cd-pool-provider"
export REPO="ssvaidyanathan/apigee-ci-cd-apigeecli-gh-actions"

gcloud services enable iamcredentials.googleapis.com \
  --project "${PROJECT_ID}"

gcloud iam service-accounts create "${SERVICE_ACCOUNT}" \
  --project "${PROJECT_ID}"

gcloud projects add-iam-policy-binding "$PROJECT_ID" \
  --member="serviceAccount:${SERVICE_ACCOUNT}@${PROJECT_ID}.iam.gserviceaccount.com" \
  --role="roles/apigee.admin"

gcloud iam workload-identity-pools create "${WORKLOAD_IDENTITY_POOL}" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --display-name="Apige CICD Pool"

export WORKLOAD_IDENTITY_POOL_ID=$(gcloud iam workload-identity-pools describe "${WORKLOAD_IDENTITY_POOL}" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --format="value(name)")

  # will be projects/projectId/locations/global/workloadIdentityPools/apigee-ci-cd-pool

gcloud iam workload-identity-pools providers create-oidc "${WORKLOAD_IDENTITY_POOL_PROVIDER}" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --workload-identity-pool="${WORKLOAD_IDENTITY_POOL}" \
  --display-name="Apigee CICD Pool Provider" \
  --attribute-mapping="google.subject=assertion.sub,attribute.actor=assertion.actor,attribute.repository=assertion.repository" \
  --issuer-uri="https://token.actions.githubusercontent.com"

gcloud iam service-accounts add-iam-policy-binding "${SERVICE_ACCOUNT}@${PROJECT_ID}.iam.gserviceaccount.com" \
  --project="${PROJECT_ID}" \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/${WORKLOAD_IDENTITY_POOL_ID}/attribute.repository/${REPO}"

export WORKLOAD_IDENTITY_POOL_NAME=$(gcloud iam workload-identity-pools providers describe "${WORKLOAD_IDENTITY_POOL_PROVIDER}" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --workload-identity-pool="${WORKLOAD_IDENTITY_POOL}" \
  --format="value(name)")

echo "WORKLOAD_IDENTITY_POOL_NAME: ${WORKLOAD_IDENTITY_POOL_NAME}"

# projects/projectId/locations/global/workloadIdentityPools/apigee-ci-cd-pool/providers/apigee-ci-cd-pool-provider

```

projects/769749427232/locations/global/workloadIdentityPools/apigee-ci-cd-pool/providers/apigee-ci-cd-pool-provider