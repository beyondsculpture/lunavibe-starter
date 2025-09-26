# Blitzy Orchestration Pack — Lunavibe AI VID.GEN

Purpose: hand Blitzy a precise, constraint-driven plan to bulk-build Lunavibe with **Google-only** runtime stack and CI/CD. Blitzy’s role: write code and open PRs. **Cloud Build** handles deploys. **Secure Source Manager (SSM)** replaces Cloud Source Repositories for new orgs.

---

## Step 1 — Put code under Google **Secure Source Manager**
> Cloud Source Repositories is not available to new customers after Jun 17, 2024. Use **Secure Source Manager** (SSM).

1) Create SSM instance + repo (Console → Secure Source Manager).  
2) Push the `lunavibe` repo into SSM:
```bash
gcloud alpha securesourcemanager repos clone lunavibe --instance=<SSM_INSTANCE_URI>
# or init & push:
git init
git remote add origin https://<SSM_INSTANCE_URI>/r/lunavibe
git add . && git commit -m "seed"
git push -u origin main
```
3) Protect `main` with PR requirement in SSM (branch protection).

Refs: SSM overview; SSM→Cloud Build triggers. See `docs/links.md` for citations.

## Step 2 — Connect SSM → Cloud Build (auto deploy on PR merge)
- Add the `cloudbuild.yaml` at repository root (already present).
- Add `ssm-triggers.yaml` (provided below) to `/.secure-source/triggers.yaml` to auto-create Cloud Build triggers.

```yaml
# /.secure-source/triggers.yaml
triggers:
- name: lunavibe-main-deploy
  description: Build+Deploy on push to main
  filename: cloudbuild.yaml
  github: null
  includedFiles: ["backend/**","cloudbuild.yaml","deploy/**"]
  ignoredFiles: ["flutter/**"]
  substitutions:
    _REGION: "us-central1"
    _AR_REPO: "lunavibe-ar"
    _SERVICE_NAME: "lunavibe-api"
    _BUCKET: "lunavibe-$PROJECT_ID-assets"
  triggerTemplate:
    branchName: "^main$"
```
Then follow SSM docs to **Connect to Cloud Build** and point it at this triggers file.

## Step 3 — Service account for Cloud Build “user-specified account”
Create `sa/cloud-build-exec` and grant minimal roles:
- `roles/cloudbuild.builds.editor`
- `roles/run.admin`
- `roles/iam.serviceAccountUser` (to deploy with Cloud Run runtime SA)
- `roles/artifactregistry.writer`
- `roles/storage.objectAdmin` (limited to your GCS bucket)
- `roles/aiplatform.user`

Export its email and set as the build SA for the trigger.

## Step 4 — Give Blitzy repo access only
- Add a **Blitzy bot user** to the SSM repo with *Write* access. Or mirror SSM→GitHub and connect Blitzy there (preferred if SSM UI lacks direct invite). Blitzy will push feature branches + PRs. CI deploys on merge.

## Step 5 — Hand Blitzy the Work Orders
- Upload **work_orders.jsonl** to Blitzy as the project spec. 
- Also attach **acceptance_criteria.yaml** and **project_guide.md**. 
- Request *batch build* targeting `feature/*` branches, one PR per work order.

## Step 6 — Your only actions
- Approve PRs in SSM when checks are green.
- Provide Google Cloud authentication (once) in Cloud Shell for bootstrap.
- App store publishing later (Apple/Play) — out of scope of Blitzy.

