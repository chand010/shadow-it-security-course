# Lesson 02 — Authorized Discovery and Evidence Quality

**Module:** 02 — Discovery and visibility  
**Session:** 2026-10-31  
**Status:** Course material published; practical work not verified  
**Environment:** Ubuntu, 16 GB RAM, Docker available; no new service or package is required

## Learning objectives

By the end of this lesson, you should be able to:
1. Write a narrow discovery scope with explicit exclusions.
2. Identify at least six sources that can reveal possible Shadow IT.
3. Explain why one telemetry signal rarely proves organizational use or data exposure.
4. Distinguish discovery leads, validated observations and confirmed findings.
5. Analyze synthetic events using Python's standard library.
6. Produce a defensible evidence register for a fictional court assessment.

## 1. Discovery begins with authority and scope

Before collecting data, document:
- **Authority:** who approved the assessment and under what mandate.
- **Purpose:** the question the assessment is meant to answer.
- **In-scope assets:** named systems, accounts, logs, dates and departments.
- **Out-of-scope assets:** personal accounts, unrelated devices, third-party networks and systems not covered by approval.
- **Collection method:** the minimum data needed to answer the question.
- **Handling:** who can access evidence, where it is stored, how integrity is checked and when it is deleted.
- **Stop conditions:** unexpected sensitive data, out-of-scope activity, service impact or an instruction from the authorizing owner to stop.

For this training lab, the only in-scope assets are your own Ubuntu machine, files created for this lesson and any local demo container you start yourself. Your home LAN's other devices, employer/court systems, other people's accounts and public networks are out of scope.

**Never run network discovery, packet capture, DNS monitoring or endpoint collection against a court or third-party network without written authorization and a defined collection plan.**

## 2. Discovery sources and what they can prove

Shadow IT discovery is a correlation exercise. Different sources answer different questions.

| Source | Useful signal | What it does not prove by itself |
|---|---|---|
| Procurement / finance / expense records | SaaS purchase, renewal, reimbursement or card charge | That the service is currently used or contains court data |
| Identity provider / SSO logs | App sign-in, federation, OAuth consent, user and timestamp | That all access is visible; non-SSO accounts may be absent |
| DNS resolver logs | A device or account queried a domain | A successful login, upload or organizational purpose |
| Secure web gateway / proxy | Web access metadata, destination and sometimes user attribution | Encrypted content or exact data transferred in every case |
| Endpoint inventory / browser extensions | Installed clients, extensions or processes | That a service is approved or that data was uploaded |
| CASB / cloud discovery | Cloud-service usage signals and policy events | Complete visibility if traffic bypasses the control |
| SaaS admin / audit logs | Sharing, sign-in, admin changes, file activity | Activity outside the log's retention or coverage |
| Staff and business-owner interviews | Business purpose, workflow and owner leads | Technical configuration or complete user activity |
| Contracts / vendor register | Contract owner, terms, renewal and approved purpose | Actual usage or secure configuration |

A useful assessment triangulates independent sources. For example, an expense entry plus an identity-provider OAuth consent event plus a business-owner confirmation is stronger evidence of organizational use than a DNS query alone.

### Three evidence states

- **Lead:** a clue that deserves validation (for example, a DNS query or staff report).
- **Validated observation:** corroborated evidence that a named service is used for a stated organizational purpose.
- **Confirmed finding:** evidence supports a specific control failure or exposure (for example, an audit log and controlled test confirm that a confidential synthetic document is accessible through an unauthenticated public link).

Do not upgrade a lead to a finding merely because it looks suspicious.

## 3. Network visibility: understand the limits

Network telemetry can help identify cloud services, but it has blind spots:
- DNS may show a lookup without a successful connection.
- TLS normally protects application content; a network observer may see metadata but not file contents.
- VPNs, encrypted DNS, proxies, split tunnelling and mobile networks can change what is visible.
- Shared cloud hosting and content delivery networks can make domain-to-vendor attribution uncertain.
- A browser visit to a service does not prove that a user uploaded organizational information.

### Tool roles

**Wireshark / TShark**
- Wireshark is a packet analyzer for examining captured traffic.
- It has separate capture-filter and display-filter languages. A capture filter reduces what is recorded; a display filter changes what is shown from an existing capture. For example, the display filter **dns** shows DNS packets, while **tcp.port == 443** narrows displayed TCP packets to port 443.
- Use it only on an authorized interface or a lab-generated capture. Packet captures may contain personal or confidential information; protect and minimize them.

**Zeek**
- Zeek analyzes network traffic and produces structured logs. Common outputs include conn.log, dns.log, http.log and ssl.log.
- It is useful for summarizing connections and DNS activity from an authorized packet capture or sensor.
- A Zeek connection record is network evidence, not proof that a user intentionally used a SaaS service.

**Wazuh**
- Wazuh agents can collect endpoint and application logs and forward them for centralized analysis.
- It can contribute endpoint and security-event context, but it does not automatically provide a complete inventory of every SaaS application or every cloud file action.
- Later in the course we will use Wazuh for monitoring and alert triage. Do not deploy a full manager/indexer stack on this 16 GB machine for today's exercise.

**CASB and SaaS management**
- CASB discovery can correlate cloud traffic and identity signals, depending on deployment and coverage.
- SaaS management platforms focus on application inventory, ownership, contracts, usage and licenses.
- These tools complement, rather than replace, business-owner validation and evidence handling.

## 4. Enterprise and Gartner lens

A practical enterprise discovery programme combines:
1. identity and access telemetry;
2. network and endpoint visibility;
3. procurement, expense and contract data;
4. cloud audit events;
5. business-owner interviews and service-desk records.

A CASB may help with cloud-use discovery and policy enforcement. A SaaS management platform may help reconcile subscriptions, owners and renewals. An SSPM tool is more focused on SaaS security configuration. The correct category depends on the control gap.

**Gartner research concept — Market Guide:** use a Market Guide to understand a market's scope, common use cases and vendor landscape. Read the report's publication date, definitions, inclusion criteria and limitations. A Market Guide is not a certification, exhaustive vendor list or proof that a product meets your requirements. In Module 07, we will use Gartner research alongside a weighted, requirements-based proof of concept.

## 5. Lab A — record the local baseline without scanning a network

Create a private working folder:

~~~
mkdir -p ~/shadow-it-lab/lesson-02
chmod 700 ~/shadow-it-lab/lesson-02
cd ~/shadow-it-lab/lesson-02
~~~

Check which tools are already present; do not install anything just to complete this lesson:

~~~
for tool in python3 jq wireshark tshark zeek wazuh-agent; do
  if command -v "$tool" >/dev/null 2>&1; then
    printf '%-14s %s\n' "$tool" "available"
  else
    printf '%-14s %s\n' "$tool" "not installed"
  fi
done

date -Is
free -h
docker ps --format 'table {{.Names}}\t{{.Image}}\t{{.Status}}'
~~~

Keep any host-specific output private. Do not publish IP addresses, hostnames, usernames, MAC addresses, raw logs or screenshots that expose unrelated services.

## 6. Lab B — analyze a synthetic discovery dataset

This exercise uses fictional .example domains and fictional identities. It does not generate network traffic or contact any service.

Create the event file:

~~~
cat > synthetic-discovery-events.jsonl <<'EOF'
{"time":"2026-10-31T08:10:00Z","source":"dns","principal":"clerk.demo","destination":"cloudbox-demo.example","event":"query","note":"DNS query only; no proof of login or upload"}
{"time":"2026-10-31T08:14:00Z","source":"proxy","principal":"clerk.demo","destination":"cloudbox-demo.example","event":"web_access","note":"Fictional proxy event; encrypted content not inspected"}
{"time":"2026-10-31T08:18:00Z","source":"identity","principal":"clerk.demo","destination":"cloudbox-demo.example","event":"oauth_consent","note":"Fictional consent event; requested scopes require review"}
{"time":"2026-10-31T08:25:00Z","source":"expense","principal":"records.manager.demo","destination":"cloudbox-demo.example","event":"subscription_charge","note":"Fictional reimbursement record; owner and business purpose not yet confirmed"}
{"time":"2026-10-31T09:02:00Z","source":"dns","principal":"admin.demo","destination":"meetnow-demo.example","event":"query","note":"Fictional approved meeting service"}
{"time":"2026-10-31T09:04:00Z","source":"identity","principal":"admin.demo","destination":"meetnow-demo.example","event":"sso_login","note":"Fictional SSO event"}
{"time":"2026-10-31T09:35:00Z","source":"endpoint","principal":"procurement.demo","destination":"quicksign-demo.example","event":"browser_extension_seen","note":"Fictional extension inventory; business purpose unknown"}
{"time":"2026-10-31T09:41:00Z","source":"identity","principal":"procurement.demo","destination":"quicksign-demo.example","event":"oauth_consent","note":"Fictional consent event; granted permissions not yet reviewed"}
{"time":"2026-10-31T10:00:00Z","source":"dns","principal":"system.demo","destination":"updates.vendor.example","event":"query","note":"Single DNS query; may be a software update check"}
EOF
~~~

Create a small parser using Python's standard library:

~~~
cat > summarize_events.py <<'PY'
import json
from collections import defaultdict, Counter
from pathlib import Path

path = Path("synthetic-discovery-events.jsonl")
groups = defaultdict(list)

with path.open(encoding="utf-8") as handle:
    for line_number, line in enumerate(handle, start=1):
        if not line.strip():
            continue
        try:
            event = json.loads(line)
        except json.JSONDecodeError as exc:
            raise SystemExit("Invalid JSON on line " + str(line_number) + ": " + str(exc))
        for field in ("source", "principal", "destination", "event"):
            if field not in event:
                raise SystemExit("Missing " + field + " on line " + str(line_number))
        groups[event["destination"]].append(event)

print("SYNTHETIC DISCOVERY SUMMARY — leads only, not incident findings")
for destination, events in sorted(groups.items()):
    sources = Counter(item["source"] for item in events)
    principals = sorted({item["principal"] for item in events})
    event_types = sorted({item["event"] for item in events})
    print()
    print("Service: " + destination)
    print("  Events: " + str(len(events)))
    print("  Sources: " + ", ".join(name + "=" + str(count) for name, count in sorted(sources.items())))
    print("  Principals: " + ", ".join(principals))
    print("  Event types: " + ", ".join(event_types))
    print("  Next step: validate owner, purpose, scope and source records.")
PY

python3 summarize_events.py
~~~

### Investigation task

For each fictional service, answer:
1. What source types support the lead?
2. Is there evidence of a sign-in, organizational purchase, granted app permission or merely a DNS lookup?
3. Which source record should be requested next?
4. Who can validate the business purpose and data owner?
5. What can you conclude now, and what must remain unknown?

Expected reasoning:
- **CloudBox Demo** has several independent lead types. Validate the identity consent details, expense record and business owner; this still does not prove a confidential file was uploaded.
- **QuickSign Demo** has endpoint and identity signals. Review the actual OAuth scopes, publisher, consent actor and business purpose.
- **MeetNow Demo** has DNS and SSO events and is marked as approved in this fictional exercise; verify that the approval and account records are current.
- **updates.vendor.example** has only one DNS event. Do not classify it as Shadow IT from that signal alone.

The parser deliberately does not assign a risk score. Analysts must assess source reliability, corroboration, scope and impact before making a finding.

## 7. Lab C — build an evidence register

Create a blank register:

~~~
cat > evidence-register.csv <<'EOF'
evidence_id,lead,source,collection_time_utc,scope_authority,collector,integrity_reference,limitation,next_validation_step,status
E-001,CloudBox Demo,DNS synthetic,2026-10-31T08:10:00Z,Training dataset only,Analyst Demo,synthetic-no-hash,DNS query does not prove login or upload,Correlate identity and proxy events,Lead
E-002,CloudBox Demo,Identity synthetic,2026-10-31T08:18:00Z,Training dataset only,Analyst Demo,synthetic-no-hash,Consent event does not prove data access,Review scopes and app publisher,Lead
EOF
~~~

For real authorized work, record the actual collection method, source system, time zone, collector, access controls and a cryptographic hash where appropriate. Never invent a hash or integrity reference. Preserve original evidence separately from working copies and follow the organization's evidence-retention and legal-hold requirements.

## 8. Fictional court scenario

A fictional court division has three leads:
- CloudBox Demo appears in DNS, proxy, identity-consent and expense data.
- QuickSign Demo appears in endpoint inventory and identity-consent data.
- updates.vendor.example appears only in one DNS record.

The court security lead asks: "Which service is a confirmed breach?"

Your professional answer should be: **the current dataset does not establish a confirmed breach.** CloudBox Demo and QuickSign Demo warrant prioritized validation because multiple independent sources indicate possible organizational use and app authorization. The single DNS event for the update domain is a weak lead. Request the relevant audit records and owner confirmations under approved scope before reaching a finding.

Prepare a one-page discovery note containing:
- scope and authority;
- discovery sources used;
- lead summary and confidence rationale;
- evidence gaps;
- next collection request;
- privacy and evidence-handling safeguards.

## 9. Knowledge check

Answer in your own words:
1. Why must authorization and scope be defined before discovery?
2. What can a DNS query establish, and what can it not establish?
3. Explain the difference between a Wireshark capture filter and display filter.
4. What is one useful role for Zeek and one limitation of its connection logs?
5. How can Wazuh contribute to visibility, and why is it not a complete SaaS inventory by itself?
6. Which fictional service would you validate first, and what two independent records would you request?

## References

- [Wireshark User's Guide — filtering packets](https://www.wireshark.org/docs/wsug_html/)
- [Zeek Quick Start Guide](https://docs.zeek.org/en/master/quickstart.html)
- [Wazuh documentation — log data collection](https://documentation.wazuh.com/current/user-manual/capabilities/log-data-collection/index.html)
- [Gartner — Magic Quadrant research methodology](https://www.gartner.com/en/research/methodologies/magic-quadrants-research)

## Completion criteria

Lesson 02 remains **In progress** until:
- the local tool-availability check has been run;
- the synthetic event parser has run and its output has been reviewed;
- the evidence register has been completed with fictional evidence only;
- the six knowledge-check questions have been answered;
- a sanitized discovery note has been saved.

Do not install Zeek, Wazuh or a full monitoring stack for this lesson. The next lesson is **Module 03 — Cloud storage security**, using an isolated Nextcloud tenant only after the learner's Docker baseline and existing containers have been reviewed.
