# Coworking Space Service Extension

### Deliverables
1. `Dockerfile`
2. Screenshot of AWS CodeBuild pipeline
3. Screenshot of AWS ECR repository for the application's repository
4. Screenshot of `kubectl get svc`
5. Screenshot of `kubectl get pods`
6. Screenshot of `kubectl describe svc <DATABASE_SERVICE_NAME>`
7. Screenshot of `kubectl describe deployment <SERVICE_NAME>`
8. All Kubernetes config files used for deployment (ie YAML files)
9. Screenshot of AWS CloudWatch logs for the application
10. `README.md` file in your solution that serves as documentation for your user to detail how your deployment process works and how the user can deploy changes. The details should not simply rehash what you have done on a step by step basis. Instead, it should help an experienced software developer understand the technologies and tools in the build and deploy process as well as provide them insight into how they would release new builds.

### Deployment Process

**How it works**
- **Build:** Pushing to GitHub triggers AWS CodeBuild. It runs `buildspec.yml`, which builds the Docker image from `analytics/Dockerfile` and pushes it to Amazon ECR, tagged with the CodeBuild build number and `latest`.
- **Run:** The app runs on Amazon EKS. Kubernetes manifests live in `deployment/`:
  - `coworking.yaml`: the app Deployment, plus a LoadBalancer Service on port 5153.
  - `configmap.yaml`: DB settings (ConfigMap) and the DB password (Secret).
  - `postgresql-*.yaml`, `pv.yaml`, `pvc.yaml`: the Postgres database and its storage.
- **Monitor:** CloudWatch Container Insights (the `amazon-cloudwatch-observability` add-on) sends container logs to `/aws/containerinsights/<cluster>/application`.

**Releasing a new build**
1. Commit and push your code changes. CodeBuild builds and pushes a new image automatically.
2. Find the new image tag in ECR.
3. Update `image:` in `deployment/coworking.yaml` to the new tag, then run `kubectl apply -f deployment/coworking.yaml`.
4. Check the rollout with `kubectl rollout status deployment/coworking`. If something goes wrong, run `kubectl rollout undo deployment/coworking`.

**Changing configuration**
Edit `deployment/configmap.yaml`, then run `kubectl apply -f deployment/configmap.yaml` and `kubectl rollout restart deployment/coworking`.