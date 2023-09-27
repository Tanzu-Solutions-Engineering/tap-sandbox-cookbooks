# Switching Installed Supply Chain to Testing Scanning Supply Chain
By default, the TAP Sandbox is optimized for easy consumption of the developer, inner loop experience.  To that end, the default supply chain installed in TAP Sandbox is the “OOTB Basic” supply chain.  If you want to switch to the “OOTB Testing and Scanning” supply chain, you can do so by modifying the TAP Install values secret in the “tap-install” namespace. 

> **&#9432; IMPORTANT**
> 
> Before you follow any of these procedures, make sure to take a backup copy of the TAP install values secret so that you can reset everything to defaults if something gets misconfigured.  You can do this with the following command:
```shell
kubectl get secret tap-tap-install-values -n tap-install -o yaml > tap-sandbox-backup.yaml
```

## Overlay Overview
We’re going to modify the TAP Install Values secret in your sandbox cluster with a ytt Overlay.  The overlay does a few things for you.  First, it changes the supply chain to the “testing_scanning” supply chain.  Next, it renames the key “ootb_supply_chain_basic” to  “ootb_supply_chain_testing_scanning”.  Next, it sets up the namespace provisioner pointing to an example repo that contains a scan policy and Tekton Pipeline for java.  You can clone that repo or copy the resources out of it and change the overlay to point to your custom repo.  We also configure the metadata_store to export it’s certificate to all namespaces, so the scan jobs can store data about the app.  Finally, we put some limits on the scan job to make sure it doesn’t exceed the default quotas for the TAP Sandbox.

## Test the Overlay
Before applying, you can review the overlayed values with the following command to make sure everthing looks ok:

```shell
kubectl create secret generic tap-tap-install-values -n tap-install --dry-run=client -o yaml --from-file=values.yaml=<(kubectl get secret tap-tap-install-values -n tap-install -o jsonpath='{.data.values\.yaml}' | base64 -d | ytt -f- -f values-overlay.yaml)
```

Once you are happy with the results, you can apply the overlay with the following command:

```shell
kubectl create secret generic tap-tap-install-values -n tap-install --dry-run=client -o yaml --from-file=values.yaml=<(kubectl get secret tap-tap-install-values -n tap-install -o jsonpath='{.data.values\.yaml}' | base64 -d | ytt -f- -f values-overlay.yaml) | kubectl apply -f-
```