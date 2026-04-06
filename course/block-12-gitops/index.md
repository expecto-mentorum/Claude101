---
layout: block
title: "GitOps Finale — ArgoCD & Live Deployment"
block_number: 12
description: "The grand finale. Install ArgoCD, point it at your repo, push code, and watch your dark-themed app go live on the internet — all through GitOps."
time: "~40 min (10 min presentation + 30 min practical)"
features:
  - ArgoCD installation and configuration
  - GitOps continuous delivery
  - NodePort public exposure
  - /loop for real-time monitoring
  - /schedule for recurring tasks
  - Remote control from mobile
objectives:
  - Understand GitOps as a deployment philosophy and why ArgoCD embodies it
  - Install ArgoCD on your existing k3s cluster
  - Configure ArgoCD to watch the ai-coderrank repository
  - Expose the app via NodePort on your droplet's public IP
  - Push the dark theme changes and watch ArgoCD auto-sync them live
  - Use /loop to poll ArgoCD sync status in real time
  - Set up a /schedule task for recurring health checks
  - Access and monitor your app remotely from a phone
---

## This Is the Moment

Twelve blocks ago, you typed `claude` in a terminal for the first time. You didn't know what it would do. You didn't know if it was useful or just a toy.

Since then you've explored a codebase without reading a single file yourself. You've planned features with ADRs and diagrams. You've implemented a dark theme through conversation. You've built memory systems, custom skills, hooks, MCP integrations, CI/CD pipelines, and sub-agents. You've provisioned a real server, installed Kubernetes, and deployed your app to actual infrastructure.

But nobody can see it yet. Your app is running on a k3s cluster on a DigitalOcean droplet, behind a firewall, accessible only through port-forwarding. It exists, but it's invisible.

That changes in the next 40 minutes.

By the end of this block, you will push code to GitHub, ArgoCD will detect the change, Kubernetes will update, and anyone with a browser and your droplet's IP address will see your dark-themed ai-coderrank app. Live. On the internet. Deployed through GitOps.

This is the finish line.

## What We'll Cover

1. **GitOps philosophy** — why Git should be the single source of truth for your infrastructure
2. **ArgoCD installation** — getting it running on your k3s cluster
3. **ArgoCD configuration** — pointing it at your repo with `argocd/application.yaml`
4. **The sync loop** — watching ArgoCD detect and apply changes automatically
5. **Public exposure** — NodePort on your droplet's public IP (Ingress optional sidebar)
6. **The big push** — dark theme goes live via GitOps
7. **Monitoring with `/loop`** — real-time polling of sync status
8. **Scheduled tasks with `/schedule`** — recurring health checks
9. **Remote control** — monitoring from your phone

## Why This Block Matters

Here's the thing about deployment: most teams treat it as a separate discipline from development. You write code in one workflow, you deploy it in another, and the two are connected by fragile scripts, manual approvals, and that one person who knows the deployment process.

GitOps eliminates the gap. Your repository IS your deployment. The code you commit IS the infrastructure state. There's no "deployment process" separate from "development process" — there's just pushing to Git. ArgoCD handles the rest.

And here's the part that should genuinely excite you: this is the same pattern used by Spotify, Intuit, Tesla, and dozens of the most conservative banks on the planet. You're not learning a toy workflow. You're learning the real thing.

## Cost Notice

> **This block uses the same DigitalOcean droplet** (s-2vcpu-4gb) you provisioned in Block 7. No additional infrastructure cost. No API key needed -- everything runs through your Pro subscription.

## Prerequisites

- Completed Blocks 0-11 (k3s cluster running, app deployed, CI/CD configured)
- k3s cluster on DigitalOcean droplet (s-2vcpu-4gb, from Block 7) with kubectl access from your laptop
- The ai-coderrank repo pushed to GitHub
- Dark theme changes from Block 4 (committed or ready to push)
- The droplet's public IP address (you'll need it for NodePort access)

## Part 1: Presentation {#presentation}

> **Duration**: ~10 minutes
> **Goal**: Students understand GitOps as a philosophy, how ArgoCD implements it, and why this changes everything about how they think about deployments. Build the excitement -- this is the climax.

---

### Slide 1: The Problem With "kubectl apply"

Let's start with a horror story.

It's 2 AM. A production deployment failed. The team is scrambling. Someone asks: "What's actually running in the cluster right now?" Nobody knows. Three different engineers ran `kubectl apply` at different times today. One of them used a manifest from a branch that was never merged. Another patched something directly with `kubectl edit`. The cluster state doesn't match anything in Git.

This is called **configuration drift**, and it is the single most common source of deployment nightmares.

Now imagine the opposite. Every change to the cluster goes through Git. Every. Single. One. No exceptions. If it's not in Git, it doesn't get applied. If you want to know what's running in production, you look at the `main` branch. That's it. Git is the truth.

That's GitOps.

> **Fun fact**: The term "GitOps" was coined by Alexis Richardson at Weaveworks in 2017. It started as a blog post. Today, CNCF has an entire working group dedicated to it, and even the most security-conscious financial institutions — banks that wouldn't let you `ssh` into a server without three approvals — use GitOps for production deployments.

---

### Slide 2: GitOps in One Diagram

Here is the loop. Memorize it, because once you understand this loop, you understand GitOps:

```
Developer pushes code to Git
         |
         v
ArgoCD watches the Git repo (polling or webhook)
         |
         v
ArgoCD compares Git state vs. Cluster state
         |
         v
 Are they different?
    /          \
  No            Yes
   |             |
   v             v
 Do nothing    Sync: apply the diff to the cluster
                 |
                 v
          Cluster now matches Git
                 |
                 v
            Back to watching
```

That's it. The entire model. Git is the desired state. Kubernetes is the actual state. ArgoCD's only job is to make them match.

No `kubectl apply`. No deployment scripts. No Jenkins pipelines that `ssh` into a server and run commands. You push to Git, and the cluster converges to match. If someone manually changes something in the cluster, ArgoCD detects the drift and reverts it back to what Git says.

Git wins. Always.

---

### Slide 3: Why ArgoCD

There are several GitOps tools: ArgoCD, Flux, Jenkins X, Rancher Fleet. We're using ArgoCD because:

1. **Visual dashboard** — a web UI that shows you exactly what's synced, what's out of sync, and what's broken. You'll see it today.
2. **Battle-tested** — graduated CNCF project. Used by Intuit (TurboTax), Red Hat, Tesla, IBM, and thousands of smaller teams.
3. **Declarative configuration** — you describe what you want in YAML. ArgoCD does the rest.
4. **Handles complexity** — supports Helm charts, Kustomize overlays, raw manifests, and Jsonnet. We're using raw manifests today, but the same tool scales to enormous deployments.
5. **Self-healing** — if someone runs `kubectl delete` on a resource that should exist, ArgoCD puts it back. The cluster self-heals based on what Git declares.

Here's what the ArgoCD model looks like in practice:

```yaml
# argocd/application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: ai-coderrank
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/YOUR_USER/ai-coderrank.git
    targetRevision: main
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

That file says: "Watch the `k8s/` directory of this GitHub repo. When it changes, sync it to the `default` namespace. If the cluster drifts, heal it automatically."

One YAML file. That's all the deployment configuration you need.

---

### Slide 4: The Three Principles

GitOps rests on three principles. Let's make them concrete:

**1. Declarative configuration**
You don't write scripts that say "run this, then run that, then check this." You write YAML that says "I want 2 replicas of this container on port 3000 with these environment variables." Kubernetes and ArgoCD figure out how to get there.

> Analogy: It's the difference between giving someone turn-by-turn directions ("go left, then right, then left again") versus giving them an address and letting Google Maps figure out the route. Declarative = the address. Imperative = the directions.

**2. Git as the single source of truth**
Every change is a commit. Every commit has an author, a timestamp, a message, and a diff. You get a full audit trail for free. Want to know who changed the replica count from 2 to 5 last Tuesday? `git log`. Want to roll back? `git revert`. Your deployment history IS your Git history.

**3. Automated reconciliation**
The system continuously compares desired state (Git) with actual state (cluster) and corrects any drift. You don't trigger deployments. They happen because the states diverged.

---

### Slide 5: What We're About to Do

Here's the game plan for the practical. Follow along mentally:

1. **Install ArgoCD** on your k3s cluster — one `kubectl apply` command (the last one you'll ever run manually for deployments)
2. **Get the admin password** and access the ArgoCD UI via port-forward
3. **Apply the ArgoCD Application** — point it at your ai-coderrank repo
4. **Watch the first sync** — ArgoCD reads your `k8s/` manifests and applies them
5. **Expose the app** — set up NodePort so it's reachable from the internet
6. **Push the dark theme** — commit and push, then watch ArgoCD detect and deploy the change
7. **Access from a browser** — your app, on the internet, with the dark theme, deployed via GitOps

And then, because this is Claude Code 101 and we don't stop at "it works":

8. **Use `/loop`** to monitor ArgoCD sync status in real time
9. **Use `/schedule`** to set up a recurring health check
10. **Use remote control** to monitor everything from your phone

---

### Slide 6: /loop, /schedule, and Remote Control

Three features that turn Claude Code from a development tool into an operations tool.

**`/loop`** — Run any command or prompt on a repeating interval:
```
/loop 30s check if ArgoCD sync is complete for ai-coderrank
```
Claude will check every 30 seconds and report back. Perfect for watching deployments roll out, monitoring pod status, or waiting for a slow migration to complete.

**`/schedule`** — Create recurring cloud tasks that run on a cron schedule:
```
/schedule create "Daily health check" --cron "0 9 * * *" \
  --prompt "Check the status of all pods in the ai-coderrank namespace, \
  verify the app responds on the health endpoint, and report any issues"
```
This runs a Claude agent in the cloud — on a schedule, without you being at your desk. Morning health checks, nightly security scans, weekly dependency audits. Set it and forget it.

**Remote control** — Claude Code sessions can be accessed remotely. Start a session on your laptop, then check in from your phone's browser. You can monitor a deployment from the couch, approve a sync from a taxi, or check pod logs from bed. (We won't judge.)

---

### Slide 7: The Big Picture

Let's zoom out. Look at what you've built over these 12 blocks:

```
You write code
     |
     v
Claude Code helps you implement, review, test
     |
     v
Git commit + push
     |
     v
GitHub Actions runs CI (Block 10)
     |
     v
ArgoCD detects the change (Block 12 — RIGHT NOW)
     |
     v
Kubernetes applies the new state
     |
     v
App is live on the internet
     |
     v
/schedule monitors health daily
/loop watches deploys in real time
Remote control lets you manage from anywhere
```

That's a complete, modern, production-grade development and deployment pipeline. Not a tutorial toy. Not a simulation. The real thing.

You built all of that. With Claude Code.

## Part 2: Hands-On {#practical}

> **Duration**: ~30 minutes
> **Outcome**: ArgoCD installed, connected to your repo, app exposed publicly, dark theme deployed via GitOps, and monitoring set up with `/loop` and `/schedule`. The grand finale.
> **Prerequisites**: Completed Blocks 0-11 (k3s cluster running, app deployed, CI/CD configured), kubectl access from your laptop, the ai-coderrank repo pushed to GitHub

---

### Step 1: Install ArgoCD on Your k3s Cluster (~5 min)

This is the last time you'll manually `kubectl apply` something for deployments. After this, ArgoCD takes over.

Start Claude Code in the ai-coderrank project:

```bash
cd ~/ai-coderrank
claude
```

Ask Claude to help install ArgoCD:

```
Help me install ArgoCD on my k3s cluster. Create the argocd namespace and apply
the stable manifests.
```

Claude will generate and run the commands:

```bash
# Create the ArgoCD namespace
kubectl create namespace argocd

# Install ArgoCD using the stable manifests
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Wait for the pods to come up. Ask Claude to check:

```
Check if all ArgoCD pods are running in the argocd namespace. Wait until they're
all Ready.
```

Claude will run:

```bash
kubectl get pods -n argocd -w
```

You should see pods like:
- `argocd-server-xxx` — the API server and web UI
- `argocd-repo-server-xxx` — clones and reads your Git repos
- `argocd-application-controller-xxx` — the brains, watches for drift
- `argocd-redis-xxx` — caching layer
- `argocd-applicationset-controller-xxx` — manages ApplicationSets
- `argocd-notifications-controller-xxx` — sends notifications on sync events
- `argocd-dex-server-xxx` — handles SSO authentication

All pods should show `Running` with `1/1` Ready. This usually takes 1-2 minutes on a k3s cluster.

> **What just happened**: You installed a full GitOps delivery platform. ArgoCD is now running inside your Kubernetes cluster, ready to watch your Git repo and sync changes. In production environments, this is the same ArgoCD installation that manages thousands of services. Same tool, same config, same workflow.

---

### Step 2: Access the ArgoCD UI (~3 min)

Get the initial admin password:

```
Get the ArgoCD admin password from the cluster secret.
```

Claude will run:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d
```

Copy the password. You'll need it in a moment.

Now port-forward the ArgoCD server to your local machine:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Open your browser and go to `https://localhost:8080`. You'll get a certificate warning — that's expected for a self-signed cert. Accept it and proceed.

- **Username**: `admin`
- **Password**: the string you copied above

You should see the ArgoCD dashboard. It's empty right now — no applications. That's about to change.

> **Tip**: Leave this port-forward running in a separate terminal tab. You'll want the ArgoCD UI visible while you work through the next steps.

---

### Step 3: Create the ArgoCD Application Config (~3 min)

Now we tell ArgoCD what to watch. Ask Claude:

```
Create argocd/application.yaml that points ArgoCD at my ai-coderrank GitHub repo.
It should watch the k8s/ directory on the main branch and auto-sync changes to the
default namespace. Enable auto-prune and self-heal.

My GitHub repo is at: https://github.com/YOUR_USERNAME/ai-coderrank.git
```

Claude will create `argocd/application.yaml`:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: ai-coderrank
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/YOUR_USERNAME/ai-coderrank.git
    targetRevision: main
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

Let's unpack what each section does:

| Field | What it means |
|-------|--------------|
| `source.repoURL` | The Git repo ArgoCD will watch |
| `source.targetRevision` | Which branch to track (`main`) |
| `source.path` | Which directory contains the K8s manifests (`k8s/`) |
| `destination.server` | The K8s cluster to deploy to (the local cluster) |
| `destination.namespace` | Which namespace to deploy into |
| `syncPolicy.automated` | Auto-sync when Git changes (no manual approval needed) |
| `syncPolicy.automated.prune` | Delete resources from the cluster if they're removed from Git |
| `syncPolicy.automated.selfHeal` | Revert manual cluster changes back to what Git declares |

Apply it:

```
Apply the ArgoCD application to the cluster.
```

```bash
kubectl apply -f argocd/application.yaml
```

> **This is the pivotal moment.** ArgoCD is now watching your repo. Any changes to the `k8s/` directory on the `main` branch will be automatically applied to your cluster. You've just set up continuous delivery.

---

### Step 4: Watch the First Sync (~2 min)

Switch to the ArgoCD UI in your browser (`https://localhost:8080`). You should see the `ai-coderrank` application appear. Click on it.

ArgoCD will:
1. Clone your repo
2. Read the manifests in `k8s/`
3. Compare them to what's currently in the cluster
4. Show you the diff
5. Apply any differences

If you already deployed the app in Block 7, ArgoCD should show the application as **Synced** and **Healthy** — the manifests in Git match what's running in the cluster.

If there are differences, you'll see ArgoCD sync them automatically. Watch the resource tree in the UI — you can see each Deployment, Service, and Pod as nodes in a visual graph.

Ask Claude to verify from the CLI too:

```
Check the ArgoCD application sync status for ai-coderrank.
```

```bash
kubectl get application ai-coderrank -n argocd -o jsonpath='{.status.sync.status}'
# Should output: Synced

kubectl get application ai-coderrank -n argocd -o jsonpath='{.status.health.status}'
# Should output: Healthy
```

**Synced + Healthy** = Git matches the cluster, and all resources are running correctly. This is the state you want.

---

### Step 5: Expose the App Publicly (~5 min)

Right now, the app is running in the cluster but not accessible from the internet. Let's fix that.

Ask Claude:

```
Help me expose the ai-coderrank app on my droplet's public IP address. My droplet's
IP is YOUR_DROPLET_IP. Set up a NodePort service so I can access
the app from a browser at http://YOUR_DROPLET_IP.
```

**Option A: NodePort (recommended for this course)**

This is the simplest path. No extra config, no Ingress controller fiddling. Your app gets a port on the droplet's public IP and you can reach it directly from any browser.

Claude will help you update the Service in `k8s/` to use NodePort:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ai-coderrank
  namespace: default
spec:
  type: NodePort
  selector:
    app: ai-coderrank
  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 30080    # Accessible at http://DROPLET_IP:30080
```

> **Public IP access**: Your DigitalOcean droplet already has a public IP address. No load balancer is needed. NodePort exposes the app directly on the droplet's IP at port 30080. Just make sure the firewall allows it.

**Option B: Ingress with Traefik (sidebar — more advanced)**

k3s ships with Traefik as its built-in Ingress controller. If you want port 80 access (no port number in the URL), you can create an Ingress resource. This is optional and more complex:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ai-coderrank
  namespace: default
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: ai-coderrank
                port:
                  number: 3000
```

With Traefik Ingress, the app will be accessible at `http://YOUR_DROPLET_IP` on port 80 -- no port number needed. This is a nice-to-have but not required for the course exercises.

**Important**: Whichever option you chose, commit the Service/Ingress changes to `k8s/` and push to Git. Don't `kubectl apply` manually -- let ArgoCD handle it. This is GitOps:

```
Commit the updated service manifest and push it to the main branch. Let ArgoCD
sync it.
```

```bash
git add k8s/
git commit -m "feat: expose ai-coderrank via NodePort for public access"
git push origin main
```

Make sure the droplet's firewall allows traffic on port 30080. Ask Claude:

```
Help me check and update the DigitalOcean firewall rules for my droplet to allow
inbound traffic on port 80 and 30080.
```

Watch the ArgoCD UI — within a few minutes, it will detect the push and sync the new Service configuration to the cluster.

---

### Step 6: Push the Dark Theme — The Big Moment (~3 min)

> **Container registry**: This course standardizes on **GitHub Container Registry (GHCR)**. Your CI pipeline (Block 10) should push images to `ghcr.io/YOUR_USERNAME/ai-coderrank`, and your K8s deployment should reference that same image. If you skipped Block 10, you can push images manually with `docker push ghcr.io/YOUR_USERNAME/ai-coderrank:latest`.

This is it. The moment the whole course has been building toward.

If your dark theme changes from Block 4 aren't pushed yet, now is the time. Ask Claude:

```
Make sure all the dark theme changes are committed and push everything to the main
branch.
```

```bash
git add .
git commit -m "feat: dark theme implementation from Block 4"
git push origin main
```

Now watch. Open the ArgoCD UI side by side with your terminal.

1. ArgoCD detects the new commit (within 3 minutes by default, or instantly with a webhook)
2. It clones the updated repo
3. It diffs the new manifests against the cluster state
4. If the Docker image reference changed, it rolls out the new version
5. The status goes from **OutOfSync** to **Syncing** to **Synced**

> **Note**: If your K8s manifests reference a container image that gets built by CI (from Block 10), the full loop is: push code -> GitHub Actions builds image -> push image to registry -> update the image tag in `k8s/deployment.yaml` -> ArgoCD syncs the new image. If you're using a fixed image tag, you may need to update it. Ask Claude to help set up the image update flow.

---

### Step 7: Access the App From Your Browser (~1 min)

Open a browser. Navigate to:

```
http://YOUR_DROPLET_IP:30080
```

You should see ai-coderrank. With the dark theme. Running on Kubernetes. Deployed via GitOps. Accessible from the internet.

Take a breath. You built this.

If it's not loading, ask Claude to troubleshoot:

```
The app isn't loading at http://YOUR_DROPLET_IP:30080. Help me debug — check the
pods, service, NodePort, and firewall rules.
```

Claude will systematically check each layer: are pods running? Is the service routing correctly? Is the NodePort exposed? Are firewall rules open? This methodical debugging is exactly the kind of thing Claude excels at.

---

### Step 8: Monitor with /loop (~3 min)

Now let's use Claude Code as an operations tool. The `/loop` command runs a prompt repeatedly on an interval — perfect for monitoring.

In your Claude Code session:

```
/loop 30s check if ArgoCD sync is complete for ai-coderrank and report the sync
status and health
```

Claude will check every 30 seconds and report:

```
[12:01:30] Sync: Synced | Health: Healthy | Resources: 4/4 running
[12:02:00] Sync: Synced | Health: Healthy | Resources: 4/4 running
[12:02:30] Sync: Synced | Health: Healthy | Resources: 4/4 running
```

Now, in a separate terminal, make a small change and push it:

```bash
# Change the replica count or add a label — something small
git add k8s/
git commit -m "test: verify ArgoCD auto-sync with minor manifest change"
git push origin main
```

Watch the `/loop` output. Within a few minutes you should see:

```
[12:05:00] Sync: OutOfSync | Health: Healthy | Resources: 4/4 running
[12:05:30] Sync: Syncing  | Health: Progressing | Resources: updating...
[12:06:00] Sync: Synced   | Health: Healthy | Resources: 4/4 running
```

You just watched GitOps happen in real time. Push to Git, ArgoCD syncs, cluster updates. No manual intervention. The `/loop` command gave you a live dashboard in your terminal.

Press `Ctrl+C` (or type `stop`) to end the loop.

> **Pro tip**: `/loop` accepts any time interval -- `10s`, `1m`, `5m`. Use shorter intervals for active monitoring during deployments, longer intervals for background checks.

---

### Step 8b: Observability and Debugging (~5 min)

Now that you have a live deployment, let's build a basic observability practice. These commands will save you when something goes wrong in production.

#### Pod logs

The first thing to check when something breaks:

```bash
# Logs for the ai-coderrank deployment
kubectl logs deployment/ai-coderrank -n default

# Follow logs in real time (like tail -f)
kubectl logs deployment/ai-coderrank -n default -f

# Logs from a crashed or restarting pod (previous container)
kubectl logs deployment/ai-coderrank -n default --previous
```

Ask Claude to help interpret the logs:

```
Read the last 50 lines of logs from the ai-coderrank deployment and flag anything
that looks like an error, warning, or unexpected behavior.
```

#### Resource usage

Install the metrics-server so `kubectl top` works on k3s:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

> **Note**: On k3s, you may need to add `--kubelet-insecure-tls` to the metrics-server deployment args. Ask Claude to patch it if `kubectl top` returns errors.

Once metrics-server is running (give it a minute):

```bash
# CPU and memory usage per pod
kubectl top pods -n default

# CPU and memory usage per node
kubectl top nodes
```

This tells you whether your s-2vcpu-4gb droplet has headroom or is running hot.

#### Readiness and liveness probes

Check that your K8s manifests include health probes. Ask Claude:

```
Check all deployments in k8s/ for readiness and liveness probes. If any are
missing, add them. Use HTTP GET /health for the web service (or TCP on port 3000
if no health endpoint exists).
```

Probes tell Kubernetes when your app is ready to accept traffic (readiness) and when it's stuck and needs a restart (liveness). Without them, K8s has no way to detect a hung process.

#### ArgoCD health status

ArgoCD provides its own health model on top of Kubernetes:

```bash
# Quick status check
kubectl get application ai-coderrank -n argocd

# Detailed health for each resource
kubectl get application ai-coderrank -n argocd -o jsonpath='{.status.resources[*].health.status}' | tr ' ' '\n'
```

The ArgoCD UI also shows health per resource in the visual tree. Green = healthy, yellow = progressing, red = degraded.

#### Continuous monitoring with /loop

Combine these into a monitoring loop:

```
/loop 1m check ArgoCD sync status, pod health, and resource usage for ai-coderrank.
Report any issues.
```

This gives you a terminal-based ops dashboard that updates every minute. Use it during deployments, after configuration changes, or any time you're suspicious.

---

### Step 9: Set Up a Scheduled Health Check with /schedule (~3 min)

`/loop` is great for active monitoring, but what about when you're asleep? That's where `/schedule` comes in — it creates cloud-based tasks that run on a cron schedule.

```
/schedule create "ai-coderrank daily health check" --cron "0 9 * * *" \
  --prompt "Check the status of all pods in the default namespace. Verify the \
  ai-coderrank service is responding. Check ArgoCD sync status. If anything is \
  unhealthy, provide a summary of issues and suggested fixes."
```

This creates a scheduled agent that runs every day at 9:00 AM. It will:
1. Check pod status
2. Verify the service is responding
3. Check ArgoCD sync status
4. Report any issues

You can list your scheduled tasks:

```
/schedule list
```

And check the output of the last run:

```
/schedule show "ai-coderrank daily health check"
```

> **Use cases beyond health checks**: Nightly security scans (`/schedule` a prompt that checks for CVEs in your dependencies). Weekly cost reports. Daily log analysis. Any recurring operational task that you'd normally automate with a cron job + script can be a scheduled Claude agent instead.

---

### Step 10: Remote Control — Monitor From Your Phone (~2 min)

One last feature for the finale. Claude Code supports remote sessions — you can start a session on one machine and connect to it from another.

Start a long-running monitoring session:

```bash
claude --remote
```

This gives you a URL. Open that URL on your phone's browser. You now have access to the same Claude Code session from your phone.

Try it:
1. From your phone, ask Claude to check the pod status
2. From your phone, ask Claude to check the ArgoCD sync status
3. From your phone, trigger a `/loop 1m` to watch the cluster

This is genuinely useful in practice. You push a deployment from your laptop, close the lid, and monitor the rollout from your phone while you grab coffee. If something goes wrong, you can investigate and even run commands — all from a mobile browser.

> **When this matters**: Friday afternoon deployment. You push the change, start a `/loop` to monitor, then walk to the train. From your phone, you watch the rollout complete. No laptop required. That's operational peace of mind.

---

### Checkpoint: The Grand Milestone

Let's take stock of what you just accomplished:

**Infrastructure**:
- ArgoCD running on k3s
- `argocd/application.yaml` connecting your repo to the cluster
- Auto-sync with self-healing and pruning enabled
- NodePort exposing the app publicly on port 30080

**Workflow**:
- Push to Git -> ArgoCD syncs -> App updates automatically
- No manual `kubectl apply` needed
- Full audit trail in Git history

**Monitoring**:
- `/loop` for real-time sync monitoring
- `/schedule` for daily health checks
- Remote control for monitoring from any device

**The app**:
- ai-coderrank with dark theme
- Live on the internet at your droplet's public IP
- Deployed and managed entirely through GitOps

You started this course with `claude` and a blank terminal. You now have a complete, modern, production-grade development and deployment pipeline. Code -> CI -> GitOps -> Live.

Send the URL to a friend. Seriously. Show them. You built this.

---

### Troubleshooting

**ArgoCD shows "Unknown" sync status**
The repo might be private. You'll need to add credentials:
```bash
kubectl -n argocd create secret generic repo-ai-coderrank \
  --from-literal=url=https://github.com/YOUR_USER/ai-coderrank.git \
  --from-literal=username=YOUR_USER \
  --from-literal=password=YOUR_GITHUB_TOKEN
kubectl -n argocd label secret repo-ai-coderrank argocd.argoproj.io/secret-type=repository
```
Ask Claude to help generate a GitHub personal access token with `repo` scope.

**ArgoCD shows "Degraded" health**
Usually means pods are crashlooping. Check:
```bash
kubectl get pods -n default
kubectl logs deployment/ai-coderrank -n default
```
Ask Claude to diagnose the logs. Common issues: missing environment variables, wrong image tag, port mismatch.

**Can't access the app on the public IP**
Check the chain: Pod running? -> Service routing? -> NodePort open? -> Firewall open?
```bash
kubectl get pods -n default
kubectl get svc -n default          # Verify NodePort 30080 is listed
curl http://localhost:30080          # Test from the droplet itself
# Check firewall on DO console or via doctl — port 30080 must be allowed
```

**ArgoCD sync is slow**
By default, ArgoCD polls every 3 minutes. You can force a sync:
```bash
kubectl -n argocd patch application ai-coderrank \
  --type merge -p '{"metadata":{"annotations":{"argocd.argoproj.io/refresh":"hard"}}}'
```
Or configure a GitHub webhook for instant sync.

---

### Bonus Challenges

**Challenge 1: Webhook-triggered sync**
Configure a GitHub webhook so ArgoCD syncs instantly on push instead of polling every 3 minutes. Ask Claude to help set up the webhook URL and secret.

**Challenge 2: Rollback via Git**
Make a breaking change (wrong image tag), push it, watch ArgoCD deploy the broken version. Then `git revert` the commit, push again, and watch ArgoCD roll back. The entire rollback was a Git operation. No `kubectl rollout undo` needed.

**Challenge 3: Add ArgoCD notifications**
Set up ArgoCD notifications to send a Slack message or email when a sync completes or fails. Ask Claude to help configure the `argocd-notifications-cm` ConfigMap.

**Challenge 4: Multi-environment**
Create a `k8s/staging/` and `k8s/production/` directory structure. Create two ArgoCD Applications — one watching each directory. Ask Claude to help restructure the manifests.

---

> **What a ride.** You've gone from `claude` to GitOps. From reading code to deploying it live on the internet. From one tool to an entire ecosystem. The next and final block covers advanced patterns and what's next — but if you stopped right here, you'd already have a complete, production-ready workflow. Well done.
