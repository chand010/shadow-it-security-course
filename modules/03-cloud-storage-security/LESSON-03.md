# Lesson 03 — Cloud Storage Security and Sharing Governance

**Module:** 03 — Cloud storage security  
**Session:** 2026-11-01  
**Status:** Lesson published; local deployment and tests are not verified  
**Environment:** Ubuntu, 16 GB RAM, Docker  
**Lab:** Isolated Nextcloud + MariaDB, fictional accounts and documents only

## Learning objectives

By the end of this lesson, you should be able to:

1. Map the lifecycle of a file in a cloud collaboration service.
2. Assess internal, group, federated and public-link sharing.
3. Explain why sync clients, file ownership, retention and backups matter.
4. Review a cloud-storage service using least privilege, MFA, auditability and data minimization.
5. Distinguish server-side encryption, end-to-end encryption and disk encryption.
6. Produce a defensible cloud-storage configuration review for a fictional court.

## 1. Cloud storage is a data-governance problem

Cloud storage is not just a place where files are uploaded. It is a lifecycle:

**Create → classify → store → access → share → synchronize → retain → archive or delete**

At every stage, ask:
- Who owns the data and the business process?
- What classification applies?
- Which identities and devices can access it?
- Can users create public links or invite external collaborators?
- Can recipients download, edit, reshare or synchronize files?
- What happens when an employee leaves?
- Do deleted files remain in trash, versions, logs or backups?
- Who can administer the service and retrieve data?

A self-hosted platform is not automatically secure or approved. It still needs an accountable owner, patching, identity controls, backups, monitoring, retention rules and incident response.

## 2. Sharing models and their risks

| Sharing model | Typical recipient | Main assessment questions |
|---|---|---|
| User-to-user | Named internal account | Is the recipient authorized? Is access least-privilege? |
| Group share | Members of a team | Is group membership reviewed and current? |
| Public link | Anyone who obtains the link | Is a password required? Is expiry enforced? Is the link read-only? |
| File drop | External contributor | Can the sender upload without seeing existing files? Is the workflow approved? |
| Federated share | User on another cloud instance | Is the remote organization trusted and governed? |
| Sync client | Desktop/mobile device | Are endpoints managed, encrypted and subject to remote wipe? |
| External storage mount | S3, SMB, SFTP, WebDAV, etc. | Who owns the backend? Can users add their own mounts? Can data be reshared? |

A public link is a bearer capability: possession of the link may be enough to access the resource, depending on its configuration. A password, expiration, read-only permission and revocation reduce risk, but they do not replace classification and authorization.

A File Drop is different from a normal public folder share: it can allow an unauthenticated visitor to upload without seeing the existing folder contents. Treat it as an externally accessible intake channel and validate file-handling controls.

### Nextcloud controls to inspect

In Administration → Sharing settings, review:
- whether link sharing is enabled;
- whether public uploads are allowed;
- whether password protection is enforced for public links;
- whether a default expiry is set and enforced;
- whether resharing is permitted;
- whether sharing is restricted to members of the same groups;
- whether particular groups are excluded from creating shares;
- default permissions for user and group shares;
- whether federated sharing is enabled and governed separately.

Nextcloud documents that group-sharing restrictions do not automatically constrain federated sharing. Do not assume one setting covers every sharing path.

## 3. Identity, ownership and retention

### Identity and access
- Require MFA for privileged and sensitive accounts; assess whether it is enforced or merely optional.
- Separate ordinary user accounts from administrator accounts.
- Keep the number of service administrators small and review their access.
- Use named accounts; avoid shared administrator credentials.
- Review app passwords, sync clients, OAuth integrations and external mounts.
- Remove or disable access promptly when users leave or change roles.

### Ownership and continuity
A file can be technically accessible but operationally orphaned. Record a business owner and a technical custodian. Plan for staff departures, account suspension, transfer of ownership and preservation of records. Nextcloud's documentation notes that files are tied to their owners and can be deleted when the owner is deleted; persistent shares or a dedicated service owner may be appropriate for continuity, subject to policy.

### Retention and legal hold
Deletion in the user interface is not necessarily immediate erasure. Files may remain in:
- trash bins;
- historical versions;
- activity and system audit logs;
- backups and snapshots;
- synchronized endpoint copies;
- external storage.

Nextcloud distinguishes per-user activity logs from the system audit log generated by the **admin_audit** app. Retention for the system audit log depends on log rotation. Design retention together with legal hold, records-management and privacy requirements. Never run cleanup commands on real records without authorization and an approved retention decision.

## 4. Encryption: choose controls for the threat

| Method | Protects mainly against | Important limitation |
|---|---|---|
| TLS in transit | Network interception between client and service | Does not protect data after it reaches the service |
| Disk/block encryption | Physical theft or offline access to storage media | Running server and authorized OS administrators can access mounted data |
| Server-side encryption (SSE) | Exposure at an external storage provider, depending on key design | Keys are held by the server; does not protect against a compromised server or malicious server administrator |
| End-to-end encryption (E2EE) | Server/storage administrators reading protected file content | Can limit server-side search, previews, sharing and recovery features; plan key recovery carefully |

Do not enable Nextcloud encryption casually in this lab. Its documentation warns that losing keys or the instance secret can cause permanent data loss. Back up first and test recovery. Encryption does not replace access control, audit logging, backups or governance.

## 5. Enterprise platforms and Gartner lens

Examples of platforms used for enterprise file collaboration and content management include:
- Microsoft SharePoint and OneDrive;
- Google Drive / Google Workspace;
- Box;
- Dropbox Business;
- Nextcloud.

Adjacent security and governance tools may add:
- CASB: cloud discovery, access policies and data controls;
- SSPM: SaaS configuration posture;
- DLP: sensitive-data detection and policy enforcement;
- IAM/PAM: identity lifecycle and privileged access;
- SaaS management: application ownership, usage, contracts and renewals.

**Gartner research example — Magic Quadrant for Document Management (published 28 April 2026):** the public abstract identifies providers including Box, Dropbox, Google and Microsoft. This is a document-management market view, not a security certification, not a complete list of cloud-storage products, and not proof that any provider meets a court's legal or technical requirements. Confirm the report's market definition, inclusion criteria, date and vendor cautions; then validate requirements through documentation, contract review and a controlled proof of concept.

A sensible evaluation should test:
1. data residency and contractual processing terms;
2. SSO, MFA, SCIM and privileged-access integration;
3. sharing restrictions and external-collaborator controls;
4. audit-event coverage, export and retention;
5. DLP/API integration and incident response;
6. encryption model and key ownership;
7. backup, recovery, legal hold and deletion behavior;
8. accessibility, availability, support and total cost.

## 6. Lab A — inspect before deploying

The earlier course setup proposed a Nextcloud lab, but no local command output has been supplied to verify it. **Do not start a second stack or overwrite an existing Compose file.**

Run this read-only preflight first:

~~~
docker compose ls
docker ps -a --format 'table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}'
docker volume ls
ss -ltn | grep -E ':(8081|18081)\b' || true
find ~/shadow-it-lab -maxdepth 2 -type f -printf '%P\n' 2>/dev/null | sort
free -h
~~~

Interpretation:
- If an existing Nextcloud project is listed, inspect and reuse it; do not create another.
- If the project exists but is stopped, inspect its Compose file and volumes before starting it.
- If a different service already uses port 8081, do not stop it. Resolve the port choice first.
- If there is no existing Nextcloud lab, continue with Lab B.
- Do not publish raw output containing local usernames, paths, IP addresses or unrelated services.

## 7. Lab B — create an isolated Nextcloud environment (only if no existing lab)

The repository contains **labs/nextcloud/compose.yaml**. It uses a pinned Nextcloud 35.0.2 Apache image, MariaDB 11.4, named volumes, a private Docker network, resource limits and a loopback-only web binding. The database is not published to the host.

Create a separate lesson directory:

~~~
mkdir -p ~/shadow-it-lab/lesson-03
chmod 700 ~/shadow-it-lab/lesson-03
cd ~/shadow-it-lab/lesson-03
curl -fsSLo compose.yaml https://raw.githubusercontent.com/chand010/shadow-it-security-course/main/labs/nextcloud/compose.yaml
~~~

Create local secrets. These values must remain on your machine:

~~~
umask 077
cat > .env <<EOF
MYSQL_ROOT_PASSWORD=$(openssl rand -hex 24)
MYSQL_PASSWORD=$(openssl rand -hex 24)
NEXTCLOUD_ADMIN_USER=ncadmin
NEXTCLOUD_ADMIN_PASSWORD=$(openssl rand -hex 24)
EOF
chmod 600 .env
printf '.env\n' > .gitignore
~~~

Validate the configuration without starting containers:

~~~
docker compose config --quiet
~~~

If this command fails, stop and investigate the error. Do not paste the contents of **.env** or a fully rendered Compose configuration into chat or GitHub.

Only after the preflight confirms that there is no existing Nextcloud project or port conflict:

~~~
docker compose pull
docker compose up -d
docker compose ps
docker stats --no-stream
curl -I http://127.0.0.1:8081
~~~

Open **http://127.0.0.1:8081** in a browser. The initial administrator account is configured from the local **.env**. If initialization is still in progress, wait and inspect sanitized service status/log excerpts rather than repeatedly recreating the stack.

The lab is intentionally isolated and local-only. It is not a production deployment and must never be exposed to the internet.

## 8. Lab C — create synthetic court identities and data

In Nextcloud, create these fictional users:
- **judge01** — read-only consumer of records;
- **clerk01** — records team member;
- **records01** — fictional records custodian;
- **itadmin01** — technical administrator;
- **external01** — fictional external reviewer.

Create groups:
- **records-team**: clerk01, records01;
- **court-readers**: judge01.

Create a folder named **Court_Confidential_Test** and a text file named **Case_2026_DEMO.txt** with this synthetic content only:

~~~
FICTIONAL TRAINING RECORD
Case reference: DEMO-2026-001
Subject: Synthetic document for cloud-sharing assessment
No real person, case, address or court information is included.
~~~

Test these cases:
1. Share the folder with **court-readers** using read-only access.
2. Confirm that a user outside the group cannot access the group share.
3. In this loopback-only lab, optionally create a temporary public link for the synthetic file with read-only access and no password/expiry to observe the baseline risk. Never use real data. Keep the link on your local machine.
4. Open the link in a private browser window on the same machine to confirm the actual behavior.
5. Revoke the test link.
6. In Administration → Sharing, enforce password protection and an expiry for any public link that is permitted. Disable public uploads and resharing if the fictional business requirement does not need them.
7. Repeat a controlled test and record whether the policy is enforced.

If you cannot verify that the web port is bound to **127.0.0.1**, skip the public-link test. Do not send the test link to anyone or publish it.

## 9. Lab D — inspect evidence and retention

If the instance is installed and healthy, inspect status and app availability:

~~~
docker compose exec -u www-data app php occ status
docker compose exec -u www-data app php occ app:list
~~~

Check the Administration → Logging and Activity views. Determine whether the **admin_audit** app is enabled; do not assume it is. Record whether the following events are visible:
- user creation;
- group membership change;
- internal share creation;
- public link creation and revocation;
- file upload and deletion;
- administrator setting change.

Do not use **occ config:list system --private**; it can disclose secrets. The ordinary **occ config:list system** is safer for diagnostics, but still review and redact output before sharing it.

Review the configured retention for trash, versions, activity and system logs. For this exercise, do not change retention values or delete evidence. Later modules will cover DLP, monitoring, legal hold and governance.

## 10. Fictional court assessment

Scenario: the fictional Records Department stores a synthetic confidential case file in CloudBox Demo. A public read-only link exists with no password and no enforced expiry. The file is synthetic, and no evidence yet shows that an unauthorized person accessed it.

Write a finding with:
- **Observation:** what configuration and test evidence actually show.
- **Risk:** anyone who obtains the link may be able to access the file.
- **Impact:** unauthorized disclosure of court information if the same configuration were used with real confidential records.
- **Evidence gap:** whether any real or unauthorized access occurred; do not claim a breach without evidence.
- **Recommendation:** restrict public sharing by default, enforce password and expiry where approved, use named recipients for confidential records, review group membership, enable appropriate audit logging and test revocation.
- **Owner and due date:** assign a responsible service owner and realistic remediation target.

Do not recommend immediate shutdown without assessing business continuity, record preservation, legal hold and a safe migration path.

## 11. Evidence checklist

Use **labs/nextcloud/SHARING_REVIEW_TEMPLATE.csv** for each test. Record:
- evidence ID and timestamp (with time zone);
- file classification and synthetic filename;
- owner and recipient scope;
- share type and permissions;
- password and expiry state;
- resharing and public-upload settings;
- result of the controlled access test;
- audit-log reference;
- limitation and next action.

Never record real share tokens, passwords, personal data or court records in the public repository.

## 12. Cleanup and safety

At the end of the exercise, stop only the project you created:

~~~
docker compose stop
docker compose ps
~~~

Named volumes persist after stopping. If you later choose to remove the lab, first confirm that the volumes contain only disposable synthetic data and that you no longer need the evidence. **docker compose down -v** deletes the project volumes and is intentionally not part of today's routine cleanup.

## Knowledge check

1. Why is a public link different from a named-user share?
2. What does File Drop allow, and what should be reviewed before enabling it?
3. Why can deleting a user's account affect file continuity?
4. Explain one difference between server-side encryption and end-to-end encryption.
5. Why must trash, file versions, logs and backups be considered in a retention assessment?
6. The synthetic file is accessible through a public link. What additional evidence is required before claiming that an unauthorized disclosure occurred?

## References

- [Nextcloud 35 — File sharing configuration](https://docs.nextcloud.com/server/stable/admin_manual/configuration_files/file_sharing_configuration.html)
- [Nextcloud 35 — Data retention](https://docs.nextcloud.com/server/stable/admin_manual/gdpr/data_retention.html)
- [Nextcloud 35 — Server-side encryption](https://docs.nextcloud.com/server/stable/admin_manual/configuration_files/encryption_configuration.html)
- [Nextcloud Docker image and Compose documentation](https://github.com/nextcloud/docker)
- [Docker Hub — supported Nextcloud image tags](https://hub.docker.com/_/nextcloud/)
- [Gartner Magic Quadrant for Document Management (28 April 2026)](https://www.gartner.com/en/documents/7773353)

## Completion criteria

Lesson 03 remains **In progress** until:
- the Docker preflight has been reviewed;
- an existing Nextcloud deployment has been safely reused or a new isolated stack has been deployed and verified;
- synthetic users, groups and a synthetic document have been created;
- internal sharing and public-link policy have been tested;
- relevant activity/audit visibility and retention settings have been reviewed;
- the sharing review template and court finding have been completed;
- the six knowledge-check questions have been answered.

The next lesson is **Module 04 — Identity, IAM and PAM**, including role matrices, MFA enforcement, joiner/mover/leaver controls and privileged-account review.
