# Module 01 — Shadow IT Fundamentals

## Learning objectives
By the end of this module, you should be able to define Shadow IT, distinguish it from approved IT, explain why employees adopt unsanctioned applications, identify risks without assuming every unsanctioned service is malicious, and describe proportionate discovery and governance.

## 1. What is Shadow IT?
Shadow IT refers to hardware, software, cloud services, applications, integrations or technology processes used within an organization without the knowledge, approval or oversight of responsible IT or security governance functions.

Examples:
- A staff member stores work documents in a personal cloud drive.
- A department purchases a SaaS subscription without procurement review.
- An employee shares files through a personal account.
- A team connects an unreviewed third-party application to an organizational account.
- Staff use an AI service to summarize documents without an approved data-handling process.

A service can be legitimate and secure in general while still being inappropriate for an organization's data or legal obligations.

## 2. Why Shadow IT happens
Common drivers include slow provisioning, difficult approved tools, collaboration needs, separate departmental budgets, limited understanding of data classification, easy SaaS registration and business needs evolving faster than formal IT processes. Investigate the unmet business need rather than simply blaming users.

## 3. Risk areas

| Risk | Example | Potential control |
|---|---|---|
| Confidentiality | Sensitive file shared through a public link | Sharing restrictions and DLP |
| Identity | Former employee retains access | Central identity lifecycle |
| Privilege | Too many global administrators | Least privilege and PAM |
| Compliance | Data stored without approved review | Data governance and legal review |
| Availability | Workflow depends on a personal account | Approved service and continuity plan |
| Visibility | Security team cannot see SaaS usage | SaaS inventory and discovery |
| Third party | Unreviewed app receives OAuth permissions | App consent governance |

Assess likelihood and impact using evidence. An application alone is not proof of a breach.

## 4. Fictional court scenario
A fictional court department uses an unapproved file-sharing service to exchange administrative documents because the approved collaboration workflow is slow. The service uses personal accounts, permits external sharing and has no central offboarding process.

Investigate:
- What business need is the department meeting?
- What types of information are shared?
- Who owns the data and service relationship?
- Are external links enabled?
- How are accounts removed when staff leave?
- What approved alternative could meet the need?

## 5. Practical exercise — synthetic SaaS inventory
Create a CSV locally:

    mkdir -p ~/shadow-it-lab
    cd ~/shadow-it-lab
    cat > fictional-saas-inventory.csv <<'EOF'
    application,department,business_owner,data_type,approval_status,external_sharing,mfa,notes
    CloudBox Demo,Records,Records Manager,Internal,Unapproved,Yes,Unknown,Training example only
    MeetNow Demo,Administration,Admin Lead,Public,Approved,No,Yes,Training example only
    QuickSign Demo,Procurement,Procurement Lead,Confidential,Unknown,Unknown,Unknown,Training example only
    EOF

Review: Which entries need follow-up? Which fields are unknown? What evidence is needed before assigning risk? What business process could be disrupted if the service were disabled immediately? Use no real court application names or data.

## 6. Deliverable
Write a one-page summary containing a definition, three drivers, five court-relevant risks, an authorized and privacy-respecting discovery approach, and two governance improvements.

## 7. Knowledge check
1. Is every unsanctioned application automatically malicious? Explain.
2. Why identify the business need?
3. What is the difference between application discovery and risk confirmation?
4. Why are unknown MFA and sharing settings evidence gaps?
5. Name two controls that help manage access after an employee leaves.

## Completion record
Status: Planned. Mark verified only after executing the exercise and reviewing findings.
