---
title: "Adversary Simulation & Detection Engineering"
summary: Mapping attack paths, detection coverage gaps, and defensive architecture through diagram-driven analysis
status: draft
portfolio: true
started: 2026-05
tags: [detection-engineering, adversary-simulation, threat-modeling, mermaid, purple-team]
---

# Adversary Simulation & Detection Engineering

## Summary

This research looks at how to model adversary behavior, simulate attack paths, and map detection coverage using structured diagrams. The goal is to build a reusable methodology for evaluating defensive posture through visual threat modeling — not just as a one-time exercise, but as something that can evolve as the attack surface changes.

## Why Diagrams?

Detection engineering suffers from the same problem as every other documentation-heavy discipline: it is hard to keep static text aligned with a dynamic environment. Diagrams help here because they force precision. A flowchart that branches on detection logic reveals gaps that prose can hide. A pie chart of telemetry coverage tells you immediately where you are blind.

Mermaid is the medium because it is text-based, version-controllable, and can live alongside the code and config it describes.

## Coverage Breakdown

The first thing to understand is how detection resources are currently distributed across the kill chain. Without this, you cannot prioritize what to simulate.

```mermaid
pie title Detection Coverage by Kill Chain Phase
    "Initial Access" : 25
    "Execution" : 18
    "Persistence" : 12
    "Privilege Escalation" : 10
    "Defense Evasion" : 8
    "Credential Access" : 7
    "Discovery" : 5
    "Lateral Movement" : 4
    "Collection" : 3
    "Exfiltration" : 3
    "Command & Control" : 5
```

This makes the imbalance obvious: most detections sit at the front of the chain. Later stages — lateral movement, collection, exfiltration — are severely under-covered. That is exactly where real damage happens.

## Attack Path Simulation

A simulated attack path helps validate whether the existing detections actually fire in the right order. Below is a common initial access to privilege escalation chain.

```mermaid
graph TD
    A[Phishing Email] -->|User clicks link| B(C2 Staging Server)
    B -->|Drops loader| C{Execution Method}
    C -->|PowerShell| D[Inline Shellcode]
    C -->|MSI Installer| E[DLL Side-Loading]
    C -->|WMI| F[Remote Script Execution]
    D --> G{Defender Check}
    E --> G
    F --> G
    G -->|AMSI Bypass| H[Beacon Loaded]
    G -->|Detected| I[Alert Generated]
    H --> J[Process Injection]
    J --> K{Token Type}
    K -->|Medium IL| L[Token Steal]
    K -->|High IL| M[Service Install]
    L --> N[Privilege Escalation]
    M --> N
    N --> O[Lsass Dump]
    O --> P{Credential Format}
    P -->|NTLM Hash| Q[Pass-the-Hash]
    P -->|Plaintext| R[Pivot to Next Host]
    Q --> R
    R --> S[Lateral Movement via WinRM]
    S --> T{Target Reachable?}
    T -->|Yes| C
    T -->|No| U[Data Staging]
    U --> V[Exfiltration over HTTPS]
```

This graph is useful because it reveals where detections are expected to fire. If you have no alerts at the AMSI bypass step but claim to detect PowerShell abuse, you have a blind spot.

## Detection Pipeline Sequence

The following sequence diagram models how telemetry flows from endpoint to analyst console. It exposes delays and transformation points where data can be lost.

```mermaid
sequenceDiagram
    participant E as Endpoint
    participant C as Collector
    participant Q as Event Queue
    participant P as Processing Pipeline
    participant D as Detection Engine
    participant T as Threat Intel
    participant A as Analyst

    E->>C: ETW Event 0x1001
    E->>C: Sysmon Event 1
    E->>C: PowerShell Log
    C->>Q: Forward batch
    Q->>P: Dequeue event
    P->>P: Normalize fields
    P->>P: Enrich with context
    P->>D: Correlate rules
    D->>T: Query IOCs
    T-->>D: Match found
    D->>D: Risk score calculation
    alt Score > Threshold
        D->>A: Alert created
        A->>A: Triage investigation
        A->>E: Isolate host
    else Score <= Threshold
        D->>Q: Re-queue for later
    end
```

The key insight here is the enrichment and correlation steps. If the pipeline drops events during normalization (malformed fields, missing timestamps), the detection engine never sees them. That is a silent failure mode.

## Dependency Graph

Below is a dependency graph of the detection stack itself. This helps visualize what happens when a single component fails.

```mermaid
graph LR
    subgraph Telemetry
        T1[ETW] --> N1[Event Collector]
        T2[Sysmon] --> N1
        T3[AuditD] --> N1
    end

    subgraph Transport
        N1 --> M1[Kafka]
        M1 --> M2[Logstash]
    end

    subgraph Storage
        M2 --> S1[Elasticsearch]
        M2 --> S2[S3 Archive]
    end

    subgraph Detection
        S1 --> D1[Sigma Rules]
        S1 --> D2[ML Model]
        D1 --> A1[Alert Manager]
        D2 --> A1
    end

    subgraph Response
        A1 --> R1[SOAR Playbook]
        R1 --> R2[API-based Remediation]
        R1 --> R3[Ticket Creation]
    end

    T1 -.->|Fail| F1[Missing Process Events]
    T2 -.->|Fail| F2[No Network Connections]
    M1 -.->|Degraded| F3[Batched Alerts Only]
    S1 -.->|Down| F4[No Historical Queries]
```

The dotted failure paths (`.->`) show single points of failure. If the transport layer (Kafka) degrades, everything downstream is delayed. If Elasticsearch goes down, detection becomes blind.

## State Machine: Session Lifecycle

A session — whether benign or malicious — follows predictable state transitions. Understanding these helps write stateful detection rules instead of relying on single-event matching.

```mermaid
stateDiagram-v2
    [*] --> Initialized
    Initialized --> Authenticated: Logon Event 4624
    Initialized --> Failed: Logon Failure 4625
    Authenticated --> Active: Process Create 4688
    Authenticated --> Idle: No Activity > 15min
    Active --> Privileged: Token Elevation
    Active --> Suspicious: Anomaly Detected
    Suspicious --> Investigated: Alert Triggered
    Investigated --> Isolated: Analyst Action
    Investigated --> Dismissed: False Positive
    Privileged --> Lateral: Network Connection
    Lateral --> Exfiltrating: High Data Volume
    Exfiltrating --> Blocked: Auto-Response
    Idle --> Terminated: Session End
    Terminated --> [*]
    Blocked --> [*]
    Isolated --> [*]
    Failed --> [*]
```

Stateful detection is more powerful than threshold-based alerting because it understands context. A network connection from a session in the "Idle" state is more suspicious than one from "Active."

## Current State

This is an early draft. The diagrams are the main output so far — they function as both analysis tools and documentation. The next step is to validate these models against real detection rules and telemetry sources.

## Next Steps

- Map each graph node to a concrete Sigma rule or detection query
- Identify gaps where no detection exists for a node
- Build a small tool that generates these diagrams from live detection config
- Test the attack path graph in a lab environment with real adversary emulation

## Open Questions

- How often should these diagrams be regenerated from live data?
- Can the dependency graph be auto-generated from infrastructure-as-code?
- Is there a way to derive coverage percentages in the pie chart from actual alert volume instead of manual estimation?
