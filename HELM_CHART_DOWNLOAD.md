# Manual Helm Chart Download Instructions

Due to network connectivity issues with automated helm pull, please download the chart manually:

## Option 1: Download via Browser

1. **Download the chart tarball**:
   - URL: https://github.com/external-secrets/external-secrets/releases/download/helm-chart-0.9.11/external-secrets-0.9.11.tgz
   - Save to: `c:\Users\hp\Documents\GITOPS\gitops-repo\`

2. **Extract the chart**:
   ```bash
   cd c:\Users\hp\Documents\GITOPS\gitops-repo
   mkdir charts
   tar -xzf external-secrets-0.9.11.tgz -C charts
   ```

3. **Verify extraction**:
   ```bash
   ls charts/external-secrets/
   cat charts/external-secrets/Chart.yaml
   ```

## Option 2: Use Git Clone (if GitHub access works)

```bash
cd c:\Users\hp\Documents\GITOPS\gitops-repo
mkdir -p charts
cd charts

# Clone the external-secrets repo
git clone --depth 1 --branch helm-chart-0.9.11 https://github.com/external-secrets/external-secrets.git temp

# Copy just the chart
cp -r temp/deploy/charts/external-secrets ./
rm -rf temp

cd ..
```

## Option 3: Download from GitHub Archive

1. Visit: https://github.com/external-secrets/external-secrets/tree/helm-chart-0.9.11/deploy/charts/external-secrets
2. Click "Code" → "Download ZIP"
3. Extract and copy the `deploy/charts/external-secrets` folder to `c:\Users\hp\Documents\GITOPS\gitops-repo\charts\`

## After Download

Once you have the chart in `charts/external-secrets/`, run:

```bash
# Verify
ls charts/external-secrets/Chart.yaml

# The ArgoCD application has already been updated to use this path
# Just commit and push:
git add charts/
git commit -m "Vendor External Secrets Helm chart v0.9.11"
git push origin main
```
