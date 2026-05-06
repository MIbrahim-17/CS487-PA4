## Information

| Field | Value |
|---|---|
| Name | Mohammad Ibrahim |
| Roll Number | 27100085 |
| GitHub Repository URL | https://github.com/MIbrahim-17/CS487-PA4 |
| Resource Group | `rg-sp26-27100085` |
| Assigned Region | `ukwest` (UK West) |


## Task 1: App Service Web App (15 points)

### Task 1 — Forked GitHub Repository

![Forked repository at MIbrahim-17/CS487-PA4 showing PA4 starter structure](docs/Part-A/Task-1/Task-1-GitHub-forked-repo.png)

The MIbrahim-17/CS487-PA4 fork showed the complete PA4 starter directory structure — `webapp/`, `function-app/`, `validate-api/`, `report-job/`, and `docs/` — confirming the working copy was in place for all subsequent development.

![Web App Overview — Running](docs/Part-A/Task-1/Task-1-web-app-overview.png)

The `pa4-27100085` Web App in `rg-sp26-27100085` showed a Running status with Node.js runtime on a B1 plan and public URL https://pa4-27100085.azurewebsites.net.

![Deployment Center — GitHub Connected](docs/Part-A/Task-1/Task-1-deployment-center-github-connected.png)

The Deployment Center configuration showed `pa4-27100085` connected to the MIbrahim-17/CS487-PA4 fork via GitHub Actions on the main branch, enabling automatic deployments on every push.

![Deployment Center — Deployment Succeeded](docs/Part-A/Task-1/Task-1-deployment-center-deployment-succeeded.png)

The Deployment Center history confirmed the latest GitHub Actions run completed with a Succeeded status and a commit-level timestamp, proving the CI/CD pipeline was delivering code to App Service.

![TaskFlow Dashboard Loading in Browser](docs/Part-A/Task-1/Task-1-browser-showing-taskflow-dashboard.png)

The TaskFlow order submission dashboard loaded correctly at https://pa4-27100085.azurewebsites.net, confirming App Service was serving the Node.js Express application and static assets from `webapp/public/`.

![Application Settings — FUNCTION_START_URL and FUNCTION_STATUS_URL](docs/Part-A/Task-1/Task-1-env-variables-page.png)

The App Service application settings showed `FUNCTION_START_URL` and `FUNCTION_STATUS_URL` configured with the Durable Function App endpoints, connecting the Web App frontend to the `http_starter` trigger for starting orchestrations and to the status polling endpoint for tracking order progress.

---

## Task 2 — Azure Container Registry (15 pts)

The `pa427100085` Azure Container Registry was created in `rg-sp26-27100085` (UK West, Standard SKU) to host the three pipeline images. All three images — `validate-api`, `report-job`, and `func-app` — were built locally from their respective directories and pushed to `pa427100085.azurecr.io` with the `v1` tag. A local smoke test against `validate-api` confirmed the image logic was correct before the push.

![ACR Overview — Succeeded](docs/Part-B/Task-2/Task-2-acr-overview-showing-succeeded.png)

The `pa427100085` registry in `rg-sp26-27100085` showed a Succeeded provisioning state and Standard SKU at endpoint `pa427100085.azurecr.io`, confirming it was ready to accept image pushes.

![Docker Build — validate-api](docs/Part-B/Task-2/Task-2-docker-build-1.png)

The `docker build` output showed the `validate-api` image built from the `validate-api/` directory and tagged as `pa427100085.azurecr.io/validate-api:v1`, with all Python base layers and dependencies resolved.

![Docker Build — report-job](docs/Part-B/Task-2/Task-2-docker-build-2.png)

The `docker build` output showed the `report-job` image built from `report-job/` with PDF generation dependencies installed and tagged as `pa427100085.azurecr.io/report-job:v1`.

![Docker Build — func-app](docs/Part-B/Task-2/Task-2-docker-build-3.png)

The `docker build` output showed the `func-app` image built from `function-app/` with Azure Durable Functions and Container Instance SDK dependencies installed and tagged as `pa427100085.azurecr.io/func-app:v1`.

![Local Validator Test — POST /validate returning valid:true](docs/Part-B/Task-2/Task-2-POST-returning-true.png)

A `curl` POST to the locally running `validate-api` container returned `{"valid": true}` for an order with an in-range quantity, confirming the image validation logic was correct before the push to ACR.

![Docker Push — validate-api and report-job](docs/Part-B/Task-2/Task-2-docker-push-1.png)

The `docker push` output showed `validate-api:v1` and `report-job:v1` uploaded to `pa427100085.azurecr.io` with per-layer progress visible and a registry-returned digest confirming successful storage.

![Docker Push — func-app](docs/Part-B/Task-2/Task-2-docker-push-2.png)

The `docker push` output showed `func-app:v1` uploaded to `pa427100085.azurecr.io` with all layers transferred and a digest hash returned by the registry.

![ACR Repository List — all three images](docs/Part-B/Task-2/Task-2-az-acr-repository-list.png)

The `az acr repository list` output listed `validate-api`, `report-job`, and `func-app` as repositories in `pa427100085.azurecr.io`, confirming all three `v1`-tagged images were available for deployment.

---

## Task 3 — Durable Orchestrator Implementation (12 pts)

The `function_app.py` was completed with four handlers: `http_starter` (receives the order JSON and starts a new Durable instance), `validate_activity` (POSTs the order to the AKS `VALIDATE_URL` and returns the JSON response dict), `report_activity` (creates a per-order ACI container group via `ContainerInstanceManagementClient`, polls every 5 seconds until Succeeded or Failed, deletes the group, and returns the blob URL), and `my_orchestrator` (calls `validate_activity` first, returns `{"status": "rejected"}` immediately if `valid` is false, otherwise calls `report_activity` and returns `{"status": "completed", "report_url": ...}`).

[View completed function\_app.py](function-app/function_app.py)

![Completed function_app.py on GitHub](docs/Part-C/Task-3/Task-3-completed-func-app-on-GitHub.png)

The GitHub file viewer showed the completed `function_app.py` committed to MIbrahim-17/CS487-PA4 with all four handler decorators — `@app.route`, `@app.orchestration_trigger`, and two `@app.activity_trigger` — visible in the file.

![All 4 Functions Registered in Portal](docs/Part-C/Task-3/Task-3-all-functions-listed.png)

The Azure Portal Function App overview for `pa4--27100085` listed all four registered handlers (`http_starter`, `my_orchestrator`, `validate_activity`, `report_activity`) as active; the Portal was used instead of local `func start` output because the Python 3.14 worker on the local machine is incompatible with the Azure Functions Core Tools runtime version, but the deployed function works correctly as demonstrated by the Task 7 end-to-end test.

### validate_activity Smoke Test Against AKS

Since local `func start` was not possible due to a Python 3.14 worker incompatibility,
the deployed orchestration output serves as the smoke test. The Durable status API
confirmed `validate_activity` reached the AKS validator and returned `valid:true` for
a real order.

![Orchestration Output — validate_activity Reached AKS Validator](docs/Part-E/Task-7/Happy-path/Task-7-AKS-validator-received-requests.png)

The `kubectl logs` output for the `validate-api` pod showed inbound `POST /validate 200 OK`
entries from the Function App's IP, confirming `validate_activity` successfully called the
AKS validator endpoint using the `VALIDATE_URL` environment variable during a live
orchestration run.

---

## Task 4 — Function App Container Deployment (8 pts)

The `func-app` image was rebuilt after Task 3 code completion and pushed to ACR. A Linux Function App `pa4--27100085` was created on the existing B1 App Service plan in UK West, configured to pull `pa427100085.azurecr.io/func-app:v1`. The pre-provisioned user-assigned managed identity `mi-pa4-27100085` was attached to satisfy the subscription's security policy blocking static `AzureWebJobsStorage` connection strings. A smoke test confirmed the HTTP starter endpoint was reachable and the Durable runtime created a new orchestration instance.

![Function App Container Image Configuration](docs/Part-C/Task-4/Task-4-func-app-deployment-center-showing-image-func-app.png)

The `pa4--27100085` Deployment Center showed the Function App configured to pull `func-app:v1` from `pa427100085.azurecr.io`, confirming a container-based deployment was active rather than a code-based one.

![User Assigned Identity — mi-pa4-27100085 Attached](docs/Part-C/Task-4/Task-4-user-assigned-identity-mi-pa4-27100085.png)

The Function App identity page showed `mi-pa4-27100085` attached as a user-assigned managed identity, granting the runtime authenticated access to Azure Storage and the Container Instance Management API without stored credentials.

![Environment Variables — Managed Identity Storage Settings](docs/Part-C/Task-4/Task-4-env-variables.png)

The application settings showed `AzureWebJobsStorage__accountName`, `AzureWebJobsStorage__credential` (set to `managedidentity`), and `AzureWebJobsStorage__clientId` configured, replacing the blocked static connection string with managed-identity-based storage authentication.

![Functions List — All 4 Handlers Enabled](docs/Part-C/Task-4/Task-4-func-list-in-overview.png)

The Function App overview listed all four functions enabled under `pa4--27100085`, confirming `func-app:v1` started successfully and the Durable Functions runtime registered all handlers.

![Smoke Test — Orchestration Started](docs/Part-C/Task-4/Task-4-smoke-test-terminal-output.png)

The terminal showed a `curl` POST to the `http_starter` endpoint returning a JSON payload with an orchestration `id` and `statusQueryGetUri`, confirming the Durable runtime accepted the request and created a new orchestration instance.

![Status Query — Expected Failed Before VALIDATE_URL Configured](docs/Part-C/Task-4/Task-4-runtime-status-showing-failed.png)

The `statusQueryGetUri` response showed the orchestration in a Failed state because `VALIDATE_URL` was not yet set, causing `os.environ["VALIDATE_URL"]` in `validate_activity` to raise a `KeyError`; this is the correct Task 4 checkpoint — it proved the full deployment was functional and only the downstream AKS service was missing.

---

## Task 5 — AKS Validator Microservice (15 pts)

A single-node Standard_B2s AKS cluster `pa4-27100085` was created in UK West. An ACR image pull secret was created and referenced in the deployment manifest so the cluster could pull `validate-api:v1` from `pa427100085.azurecr.io`. The `validate-api` Deployment and LoadBalancer Service were applied via the manifests in `validate-api/k8s/`. Once the external IP was assigned, `VALIDATE_URL` was added to the Function App's application settings.

![kubectl get nodes — Node in Ready State](docs/Part-D/Task-5/Task-5-get-nodes-showing-ready-state.png)

The `kubectl get nodes` output showed the single Standard_B2s worker node in a Ready status, confirming the `pa4-27100085` AKS cluster control plane and kubelet were communicating correctly.

![kubectl get pods — Validator Pod Running](docs/Part-D/Task-5/Task-5-get-pods-showing-running-state.png)

The `kubectl get pods` output showed the `validate-api` pod in Running state with all containers ready, confirming `validate-api:v1` was pulled from `pa427100085.azurecr.io` and the FastAPI application started without errors.

![kubectl get service — External IP Assigned](docs/Part-D/Task-5/Task-5-get-service-showing-external-ip.png)

The `kubectl get service validate-service` output showed a LoadBalancer type with an external IP assigned on port 80, providing the stable endpoint set in the `VALIDATE_URL` application setting.

![GET /health — 200 OK](docs/Part-D/Task-5/Task-5-health-response.png)

A `curl` GET to `/health` on the LoadBalancer external IP returned a 200 healthy response, confirming the LoadBalancer service was routing external traffic to the `validate-api` pod correctly.

![POST /validate — Valid Order Returns valid:true](docs/Part-D/Task-5/Task-5-good-order-returning-true.png)

A `curl` POST to `/validate` with an in-range quantity returned `{"valid": true}`, confirming the happy-path validation logic in `validate-api/app.py` was operational after deployment and would allow `my_orchestrator` to proceed to `report_activity`.

![POST /validate — Invalid Order (qty=999) Returns valid:false](docs/Part-D/Task-5/Task-5-bad-order-returning-false.png)

A `curl` POST to `/validate` with `qty=999` returned `{"valid": false, "reason": ...}`, confirming the rejection rule was enforced and the orchestrator would skip `report_activity` and return `{"status": "rejected"}` for this order.

![Function App — VALIDATE_URL Set](docs/Part-D/Task-5/Task-5-function-app-env-var-validate-url.png)

The `pa4--27100085` application settings showed `VALIDATE_URL` set to the AKS LoadBalancer external IP and `/validate` path — the value read by `validate_activity` at runtime via `os.environ["VALIDATE_URL"]`.

---

## Task 6 — ACI Report Job (15 pts)

A `reports` blob container was created in storage account `pa427100085`. A manual `az container create` test using the `report-job:v1` image confirmed the container parsed `ORDER_JSON`, generated a PDF, and wrote it to the `reports` container before exiting with Succeeded. The `mi-pa4-27100085` managed identity (Contributor on `rg-sp26-27100085`) was verified as attached to the Function App, and all `REPORT_*`, `ACR_*`, `STORAGE_ACCOUNT_URL`, and `AZURE_CLIENT_ID` application settings were configured so `report_activity` can provision and authenticate per-order ACI container groups.

![Reports Blob Container Created](docs/Part-D/Task-6/Task-6-reports-blob-container-created.png)

The `pa427100085` storage account displayed the `reports` blob container as the destination where `report-job` writes `{order_id}.pdf` files using the managed identity for authentication.

![az container show — State: Succeeded](docs/Part-D/Task-6/Task-6-azure-container-show-showing-succeeded.png)

The `az container show` output for `ci-report-test` showed a final instance state of Succeeded with exit code 0, confirming the batch container completed PDF generation and blob upload before terminating cleanly.

![az container logs — PDF Generation Output](docs/Part-D/Task-6/Task-6-azure-container-logs-showing-generation-output.png)

The `az container logs` output for `ci-report-test` showed the report job parsing `ORDER_JSON`, generating the PDF, and uploading it to the `reports` container in `pa427100085`, with a success log line confirming the blob write completed.

![az storage blob list — TEST-001.pdf in Reports Container](docs/Part-D/Task-6/Task-6-azure-container-blob-list-showing-pdf.png)

The blob list for the `reports` container in `pa427100085` showed `TEST-001.pdf` with a creation timestamp matching the manual ACI run, proving the container instance wrote the generated PDF to storage.

![Function App Identity — mi-pa4-27100085 User Assigned](docs/Part-D/Task-6/Task-6-user-assigned-identity-showing-mi-pa4-27100085.png)

The managed identity details showed `mi-pa4-27100085` with a Contributor role on `rg-sp26-27100085`, granting `report_activity` permission to call `container_groups.begin_create_or_update` and access blob storage without static credentials.

![Function App Settings — REPORT_* and ACR_* Variables (page 1)](docs/Part-D/Task-6/Task-6-func-app-env-variables-1.png)

The first settings page for `pa4--27100085` displayed `REPORT_RG`, `REPORT_LOCATION`, `REPORT_IMAGE` (`pa427100085.azurecr.io/report-job:v1`), and the `ACR_SERVER`, `ACR_USERNAME`, and `ACR_PASSWORD` (masked) values used by `ImageRegistryCredential` to authenticate ACI's pull of `report-job:v1`.

![Function App Settings — STORAGE_ACCOUNT_URL and AZURE_CLIENT_ID (page 2)](docs/Part-D/Task-6/Task-6-func-app-env-variables-2.png)

The second settings page showed `STORAGE_ACCOUNT_URL` pointing to `pa427100085`, `AZURE_CLIENT_ID` holding the managed identity client ID passed to the ACI container so `DefaultAzureCredential` could authenticate blob writes, and `SUBSCRIPTION_ID` for scoping `ContainerInstanceManagementClient` API calls; secrets were masked.

---

## Task 7 — End-to-End Pipeline Test (15 pts)

### Web App Wiring

The Web App `pa4-27100085` was connected to the Function App by setting `FUNCTION_START_URL`
to the `http_starter` endpoint with the function key, and `FUNCTION_STATUS_URL` to the
Durable Task status prefix. These two settings are the only contract between the frontend
and the orchestration layer — the Web App has no direct knowledge of AKS or ACI.

![Web App — FUNCTION_START_URL and FUNCTION_STATUS_URL Configured](docs/Part-A/Task-1/Task-1-env-variables-page.png)

The `pa4-27100085` App Service application settings showed `FUNCTION_START_URL` pointing to
the `http_starter` trigger on `pa4--27100085` with a function key appended as a query
parameter, and `FUNCTION_STATUS_URL` set to the Durable Task runtime webhook prefix used
by the frontend polling loop to track order status.

### Happy Path

Order ORD-003 was submitted with `qty=2`; the pipeline completed in approximately 45 seconds with the AKS validator approving it, an ACI report container generating the PDF, and the blob URL returned to the Web App dashboard.

![Order Form Filled — Before Submit](docs/Part-E/Task-7/Happy-path/Task-7-order-before-clicking-submit.png)

The order submission form displayed product name, `qty=2`, and customer details pre-filled and ready to submit, capturing the initial state of the happy-path test.

![Status Panel — Running with Instance ID](docs/Part-E/Task-7/Happy-path/Task-7-order-showing-running.png)

The status panel showed the orchestration in a Running state with the instance ID immediately after submission, confirming the Web App called `FUNCTION_START_URL` and the Durable runtime created a new orchestration instance.

![Status Panel — Completed with Report URL](docs/Part-E/Task-7/Happy-path/Task-7-order-showing-completed.png)

The status panel transitioned to Completed with a PDF download link constructed from `{STORAGE_ACCOUNT_URL}/reports/{order_id}.pdf`, confirming all three activities — AKS validation, ACI creation, and PDF upload — succeeded.

![Generated PDF Open in Viewer](docs/Part-E/Task-7/Happy-path/Task-7-order-pdf.png)

The generated PDF opened from the blob storage URL showed order content matching the submitted form values, confirming the file was accessible from the `reports` container in `pa427100085`.

![Function App Monitor — Orchestration Invocations](docs/Part-E/Task-7/Happy-path/Task-7-orchestrator-invocations.png)

The Function App Monitor showed the `my_orchestrator` instance triggered and completed without a top-level error, with the orchestration ID matching the one returned to the Web App's polling loop.

![Function App Monitor — Activity Logs](docs/Part-E/Task-7/Happy-path/Task-7-orchestrator-logs.png)

The activity execution logs showed `validate_activity` called first and returning `valid:true`, followed by `report_activity` being invoked, confirming the conditional branch in `my_orchestrator` chained the activities in the correct order.

![az container list — ACI Spawned for Order](docs/Part-E/Task-7/Happy-path/Task-7-ACI-spawned.png)

The output showed the ACI container group `ci-report-{order_id}` provisioned in `rg-sp26-27100085`, confirming `report_activity` dynamically created a dedicated per-order instance using `ContainerInstanceManagementClient.container_groups.begin_create_or_update`.

![Blob Storage — PDF Matching Order ID](docs/Part-E/Task-7/Happy-path/Task-7-pdf-in-blob-storage.png)

The `reports` container in `pa427100085` showed a PDF blob named with the order ID and a creation timestamp within the ACI execution window, confirming `report-job` wrote the file before `report_activity` deleted the container group.

![kubectl logs — AKS Validator Received Traffic](docs/Part-E/Task-7/Happy-path/Task-7-AKS-validator-received-requests.png)

The AKS validator pod logs showed an inbound `POST /validate` request from `validate_activity`, confirming all four pipeline services — App Service, Durable Function, AKS, and ACI — participated in processing the order.

### Reject Path

Order ORD-BAD was submitted with `qty=999`; the AKS validator returned `valid:false`, the orchestrator returned `{"status": "rejected"}` without calling `report_activity`, and no ACI container was created.

![UI — Order Rejected with Reason](docs/Part-E/Task-7/Reject-path/Task-7-rejected-order-with-reason.png)

The dashboard displayed a Rejected status with the rejection reason from the AKS validator for `qty=999`, confirming `validate_activity` received `{"valid": false, "reason": ...}` and the orchestrator returned the rejection payload to the polling loop.

![az container list — No ACI Created for Rejected Order](docs/Part-E/Task-7/Reject-path/Task-7-no-aci-spawned.png)

The container list output showed no `ci-report-*` container group was created during the reject-path test, confirming the `if not validation.get("valid"): return {...}` branch in `my_orchestrator` prevented `report_activity` from being called.

![Orchestrator Output — status:rejected](docs/Part-E/Task-7/Reject-path/Task-7-rejected-order-status-showing-rejected.png)

The Durable status query for the rejected instance showed a Completed runtime status with output `{"status": "rejected", "reason": ...}` and the orchestration `instanceId`, confirming the orchestrator terminated cleanly on the reject path as a deliberate decision rather than an unhandled exception.

### Resource Group Overview

![Resource Group rg-sp26-27100085 — All Deployed Resources](docs/Part-E/Task-7/Task-7-resource-group-overview-showing-resources.png)

The `rg-sp26-27100085` resource group overview showed the complete TaskFlow infrastructure: `pa4-27100085` App Service and AKS cluster, `pa4--27100085` Function App, `pa427100085` ACR and Storage Account, `mi-pa4-27100085` Managed Identity, and the per-order ACI container instance, confirming all services were deployed and operational in UK West.

---

## Task 8 — Documentation and Architecture Diagram (5 pts)

### Architecture Diagram

> Architecture diagram to be added at docs/architecture.png

### Service Selection

**App Service:** App Service is the right choice for the TaskFlow web UI because it provides a fully managed platform with built-in CI/CD from GitHub, eliminating server management overhead. It supports long-running Node.js processes with persistent connections, which suits a dashboard that polls orchestration status over time. The B1 SKU provides predictable billing for a frontend that receives moderate steady traffic, and deployment rollbacks are one click away in the Deployment Center.

**Durable Functions:** Durable Functions coordinates the multi-step TaskFlow pipeline because it persists orchestration state between activities, meaning a crash or timeout during report generation does not restart the entire flow from validation. Plain HTTP-triggered functions cannot checkpoint state, making long-running sequential workflows fragile and prone to timeout errors. The containerised deployment on the existing App Service plan avoids cold starts while keeping infrastructure costs minimal by reusing already-allocated compute.

**AKS:** The validator runs on AKS because it is a long-lived HTTP microservice that must respond within milliseconds on every order submission, requiring a stable always-running endpoint rather than a scale-to-zero serverless model. AKS provides declarative Kubernetes manifests, a LoadBalancer service for a stable external IP, and fine-grained resource controls such as CPU and memory requests per pod. This pattern is the industry standard for running stateless microservices that must handle sustained concurrent traffic with predictable latency.

**ACI:** The report generator runs on ACI because it is a short-lived batch job that starts, writes a PDF to blob storage, and exits with no idle period and no need for a persistent endpoint. ACI bills only for the seconds the container is alive, making per-order cost negligible when no orders are being processed. Keeping this workload off the AKS cluster avoids wasting node resources on a job that runs for roughly 20 seconds per order and then disappears.

### ACI vs AKS Comparison

When the AKS cluster is idle the single Standard_B2s node continues running and billing at its hourly VM rate because Kubernetes does not terminate nodes based on traffic absence. ACI has no concept of idle in this pipeline: each container is created fresh per order, runs for approximately 20 seconds, and is deleted by `report_activity`, so there is zero cost between orders. If a user spammed Submit 1000 times in a minute, ACI would incur the most cost because 1000 separate container instances would be created and billed individually, whereas the AKS node cost remains fixed at its hourly rate regardless of request volume.

### Durable Functions vs Plain HTTP

If the same flow were implemented as two plain HTTP functions calling each other, the first function would need to wait synchronously for the ACI report job to finish — up to 60 seconds — exceeding the default HTTP timeout and causing a 5xx error before the work completes. Additionally, if the runtime recycled the host mid-execution all orchestration state would be lost since it lives only in memory, forcing the entire order to be resubmitted. Durable Functions solves both problems by checkpointing after every activity into Azure Storage, allowing the orchestrator to replay safely from the last successful step without re-executing completed activities.

### Cost Review

Cost Management access was not available during this assignment as the required Reader role on the billing scope was not granted on the instructor's subscription. Based on resource usage, the AKS cluster running a single Standard_B2s node continuously was the single most expensive resource as it bills at the VM hourly rate regardless of whether orders are being processed. The Durable Function App and Web App share the existing B1 App Service plan adding no extra compute cost, and ACI costs were minimal as each report container ran for approximately 20 seconds per order before being deleted.

### Challenges Faced

**Challenge 1 — GitHub Actions deployment failure:** The auto-generated workflow failed with exit code 254 because it did not account for the Node.js app living inside the `webapp/` subfolder. The workflow zipped `webapp/` as a nested folder so Azure deployed `server.js` at `webapp/server.js` instead of the root, producing a `Cannot GET /` error. The fix was to rewrite the workflow to zip the contents of `webapp/` directly and set the App Service startup command to `node webapp/server.js`.

**Challenge 2 — Function App storage auth crash:** The Function App crashed immediately after creation because the subscription's security policy blocks default `AzureWebJobsStorage` connection strings. Debugging required reading the container startup logs and following the instructor's note on managed identity storage auth. The fix was to attach `mi-pa4-27100085` and replace the `AzureWebJobsStorage` connection string with three settings using `accountName`, `credential=managedidentity`, and `clientId`.

**Challenge 3 — PowerShell JSON quoting for ACI environment variables:** The manual ACI test failed with a `JSONDecodeError` because PowerShell strips double quotes from variables expanded inside double-quoted strings passed to `az` CLI. The `ORDER_JSON` environment variable arrived at the container with single quotes making it invalid JSON. The fix was to pre-escape the JSON using backslash-quote notation in the PowerShell variable before passing it to `az container create`.
