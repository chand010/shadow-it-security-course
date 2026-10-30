# Lesson 01 — Shadow IT Fundamentals and Safe First Discovery

**Module:** 01 — Shadow IT fundamentals  
**Session:** 2026-10-30  
**Status:** In progress until the learner verifies the lab  
**Environment target:** Ubuntu Linux, 16 GB RAM, Docker; no additional platform is required

## Learning objectives

By the end of this lesson, you should be able to:

1. Explain Shadow IT, shadow SaaS, shadow data and shadow AI.
2. Distinguish an unknown or unapproved service from a confirmed security incident.
3. Describe a proportionate discovery-to-governance workflow.
4. Collect a small, authorized inventory of your own Ubuntu/Docker lab.
5. Triage a fictional court-sector SaaS inventory while recording evidence gaps.

## 1. Core concepts

**Shadow IT** is technology used for organizational work without the knowledge, approval or oversight of the functions responsible for IT, security, procurement or data governance. It may include SaaS subscriptions, personal cloud accounts, browser extensions, unmanaged devices, integrations, AI tools and locally hosted services.

Related terms:

- **Shadow SaaS:** cloud applications or subscriptions used outside the approved SaaS governance process.
- **Shadow data:** organizational information copied, stored or shared outside approved data locations and controls.
- **Shadow AI:** AI services or features used without an approved process for data handling, privacy, security and human review.

An unapproved service is not automatically malicious or compromised. It is a visibility and governance concern that needs validation. Staff may adopt a tool because the approved process is slow, the required capability is missing, or a business deadline is urgent. Find the unmet business need before recommending a control.

### A practical assessment workflow

1. **Discover:** identify a possible application, account, device, integration or data flow using authorized sources.
2. **Validate:** confirm that the service is real, used for organizational work, and within the assessment scope.
3. **Understand:** identify the business owner, data owner, purpose, users, data types and dependencies.
4. **Assess:** examine identity, sharing, configuration, vendor, privacy, legal and availability risks.
5. **Respond:** onboard, remediate, restrict, replace or retire the service proportionately.
6. **Govern:** assign an owner, approval path, periodic review, logging and offboarding process.

Discovery is not the same as risk confirmation. A DNS lookup, expense entry or staff report can be a lead; it is not by itself proof that confidential information was exposed.

## 2. Risk areas in a court environment

| Risk area | Fictional example | Evidence to seek |
|---|---|---|
| Confidentiality | A case-related file is placed in a personal drive | Data classification, sharing settings and access logs |
| Identity | A former contractor still has an account | Identity lifecycle and last-login records |
| Privilege | A departmental user is an application administrator | Role assignment and privileged activity logs |
| Third-party access | An unreviewed app receives mailbox or drive permissions | OAuth consent, scopes and app owner |
| Availability | A critical workflow depends on one employee's personal account | Process owner, recovery path and export capability |
| Governance | Nobody owns a SaaS subscription | Procurement record, business owner and renewal date |
| AI data handling | Staff paste confidential text into an unapproved AI service | Approved-use policy, data flow and service terms |

Use the least intrusive evidence that can answer the assessment question. Do not inspect personal accounts or real court systems without written authorization and a defined scope.

## 3. Enterprise tool map — orientation only

These are categories, not interchangeable products:

- **CASB:** discovers and applies controls to cloud-service use, often through API or inline integrations.
- **SSE:** converges security services for access to web, cloud and private applications.
- **SASE:** combines networking capabilities with cloud-delivered security capabilities.
- **SSPM:** assesses the security posture and configuration of SaaS applications.
- **SaaS management platform:** helps maintain application, owner, contract, license and usage inventories.
- **DLP:** detects or prevents inappropriate movement or disclosure of sensitive data.
- **IAM/PAM:** governs identities, authentication, authorization and privileged access.

Illustrative products previously placed on the course map include Microsoft Defender for Cloud Apps, Netskope, Zscaler, Obsidian Security, ServiceNow, Zylo, Torii and Microsoft Purview. Product scope and packaging can change. These examples are not endorsements and do not imply a current Gartner ranking. We will compare products against court use cases in later modules.

### Gartner research concept

Gartner is a research and advisory firm, not a security platform. A **Magic Quadrant** compares providers within a defined market using **Ability to Execute** and **Completeness of Vision**, placing them in Leaders, Challengers, Visionaries or Niche Players. Treat it as a starting point, not a procurement decision: the report is market- and date-specific, and a leader may still be a poor fit for a particular court requirement. Later, use **Critical Capabilities** and your own requirements to compare products against specific use cases.

Official reading: [Gartner — Magic Quadrant research methodology](https://www.gartner.com/en/research/methodologies/magic-quadrants-research) and [Gartner — Magic Quadrant FAQ](https://www.gartner.com/en/about/magic-quadrant-faq).

## 4. Lab A — establish your own Ubuntu/Docker baseline

This is a local inventory exercise. Run it only on your own machine or a machine you are authorized to administer. Do not scan a workplace network.

Create a lesson folder:

```bash
mkdir -p ~/shadow-it-lab/lesson-01
cd ~/shadow-it-lab/lesson-01
```

Record the baseline:

```bash
date -Is
lsb_release -ds
uname -m
free -h
docker version
docker ps -a --format 'table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}'
docker network ls
ss -ltn
```

What each check tells you:

- OS and architecture help establish compatibility.
- Memory indicates how much capacity is available for later labs.
- Docker client/server versions confirm whether the daemon is reachable.
- Container and network lists show existing local workloads and networks.
- Listening TCP sockets show local services that may need an owner or explanation.

**Important:** do not paste unredacted hostnames, IP addresses, port mappings, usernames or unrelated container names into the public repository. Do not stop or modify an existing container just because you do not recognize it. Investigate ownership first. If Docker returns a permission error, record it; do not casually add your account to the Docker group because Docker administration is effectively root-level access.

## 5. Lab B — simulate an unapproved local web service

This small Nginx container represents a fictional department spinning up a service outside the normal request process. It is deliberately bound to loopback, resource-limited and disposable. It does not represent a real external exposure.

First check that the lab name and port are unused:

```bash
docker ps -a --filter name=^/shadowit-demo-01$
ss -ltn | grep ':18088' || true
```

If a container with that name already exists, or port 18088 is already in use, do not overwrite it; choose a different name and unused high port for every command below.

Start the disposable service:

```bash
docker run --rm -d \
  --name shadowit-demo-01 \
  --memory=256m \
  --cpus=0.5 \
  --pids-limit=100 \
  -p 127.0.0.1:18088:80 \
  nginx:alpine
```

Verify only the local service:

```bash
docker ps --filter name=shadowit-demo-01
curl -I http://127.0.0.1:18088
docker inspect --format 'Image={{.Config.Image}} Ports={{json .NetworkSettings.Ports}}' shadowit-demo-01
ss -ltn | grep ':18088'
```

Expected result: the container is running, the local HTTP request returns an HTTP response, and the port binding shows 127.0.0.1 rather than 0.0.0.0. If the image cannot be pulled, stop here and record the error; do not weaken firewall or network settings to make it work.

Clean up:

```bash
docker stop shadowit-demo-01
docker ps -a --filter name=^/shadowit-demo-01$
```

Because the container was started with `--rm`, it should be removed after stopping. Verify that the final command returns no matching container. If any command fails, record the exact error with secrets and identifying details removed. Do not mark this lab verified until the checks and cleanup have been confirmed.

## 6. Lab C — triage a fictional SaaS inventory

Create a CSV containing synthetic data only:

```bash
cat > fictional-saas-inventory.csv <<'EOF'
application,department,business_owner,data_classification,approval_status,external_sharing,mfa,owner_confirmed
CloudBox Demo,Records,Fictional Records Manager,Confidential,Unapproved,Public links enabled,Unknown,Yes
MeetNow Demo,Administration,Fictional Admin Lead,Public,Approved,Restricted,Enforced,Yes
QuickSign Demo,Procurement,Fictional Procurement Lead,Confidential,Unknown,Unknown,Unknown,No
EOF

column -s, -t fictional-saas-inventory.csv
```

For each entry, write down:

1. What is known, and what is only reported?
2. Which evidence source could validate the application and its use?
3. Who should confirm the business purpose and data ownership?
4. Which setting or record should be checked next?
5. What business disruption could result from disabling it immediately?
6. What safe interim action could reduce risk while preserving legitimate work?

Use this triage principle:

- **Confirmed exposure** (for example, a verified public link to confidential data) needs prompt, scoped response.
- **Unapproved use** needs ownership and business-purpose validation, followed by onboarding, an approved alternative or a controlled exception.
- **Unknown settings** are evidence gaps. They are not proof that MFA is disabled or data has leaked.
- **No clear business need** may justify retirement, but plan migration, retention and continuity first.

Do not assign a final risk rating from this CSV alone. The dataset is deliberately incomplete; identify what evidence is missing.

## 7. Fictional court scenario

A court department uses CloudBox Demo because the approved collaboration process is slow. The inventory says public links are enabled and MFA is unknown. You have not yet seen an audit log or verified that any confidential file was shared.

Prepare a short initial finding using this structure:

- **Observation:** what the inventory actually states.
- **Risk hypothesis:** what could happen if the settings and data classification are confirmed.
- **Evidence gap:** what has not yet been verified.
- **Business impact:** why immediate shutdown could disrupt work.
- **Next action:** who should validate the service, settings and data flow, and by when.
- **Interim safeguard:** a proportionate measure that does not destroy evidence or interrupt lawful work.

Never use real court names, case numbers, staff identities, documents, IP addresses or credentials in this exercise or public repository.

## 8. Evidence and deliverables

Keep raw local outputs in your private lab folder. For the public GitHub repository, publish only a sanitized summary with:
- date and lesson number;
- commands attempted and whether each was verified;
- synthetic inventory findings;
- evidence gaps and next actions;
- no secrets, personal data, real institutional details or raw infrastructure inventory.

Suggested local notes file: `lesson-01-notes.md`. Record failed commands as failed; a documented blocker is useful progress.

## 9. Knowledge check

Answer in your own words:

1. Why is an unapproved SaaS service not automatically proof of a breach?
2. Give one example each of shadow SaaS, shadow data and shadow AI.
3. What is the difference between discovering an application and validating its risk?
4. Why should a court assessment identify the business need before recommending a shutdown?
5. What do Ability to Execute and Completeness of Vision mean in a Gartner Magic Quadrant?
6. In the fictional inventory, which item would you validate first, and what evidence would you request before making a final finding?

## Completion criteria

This lesson remains **In progress** until:
- the Ubuntu/Docker baseline has been recorded;
- the local demo has either been verified and cleaned up, or a blocker has been documented;
- the fictional SaaS inventory has been reviewed;
- the six knowledge-check questions have been answered;
- a sanitized note has been saved.

Next lesson: **Module 02 — Discovery and visibility**, covering authorization and scope, discovery sources, evidence quality, and a small local packet/log investigation.