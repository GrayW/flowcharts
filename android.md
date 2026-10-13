```mermaid
%%{init: {
  'theme': 'base',
  'themeVariables': {
    'fontFamily': 'Helvetica, Arial, sans-serif',
    'fontSize': '17px',
    'primaryColor': '#ffffff',
    'primaryTextColor': '#1e293b',
    'primaryBorderColor': '#94a3b8',
    'lineColor': '#64748b',
    'clusterBkg': '#f8fafc',
    'clusterBorder': '#cbd5e1',
    'edgeLabelBackground': '#ffffff'
  },
  'flowchart': { 'curve': 'basis', 'nodeSpacing': 45, 'rankSpacing': 60, 'padding': 26, 'wrappingWidth': 300, 'htmlLabels': true }
}}%%
flowchart TD
  START(["New or existing Android device"]) --> Q1{"How is the device being<br/>enrolled?"}

  REQ["PREREQUISITE for ALL paths:<br/>Android Enterprise connected<br/>to Fleet (managed Google Play,<br/>Android Management API)"]

  %% ---- PATH 1: fully managed (COBO) ----
  Q1 -->|"PATH 1 — company-owned,<br/>fully managed device (COBO)"| COBO{"How is it provisioned?"}
  COBO -->|"Zero-touch enrollment:<br/>device configured in the<br/>Android zero-touch portal<br/>(Fleet provides the<br/>DPC extras JSON)"| ZT["New, unboxed device:<br/>provisions itself during<br/>first-run setup, no user<br/>action needed"]
  COBO -->|"QR code (device not<br/>zero-touch enabled):<br/>new or FACTORY-RESET device,<br/>tap welcome screen 6 times,<br/>scan Fleet's QR code<br/>(fully_managed=true)"| QR["User scans QR, connects<br/>to Wi-Fi, follows the<br/>setup wizard"]
  COBO -->|"Knox Mobile Enrollment:<br/>Samsung zero-touch variant"| KNOX["Samsung device provisions<br/>itself on first run / reset<br/>(via Knox portal assignment)"]
  ZT --> PENDING["Hosts awaiting automatic<br/>enrollment appear in Fleet<br/>as 'Pending'"]
  QR --> COBOOUT
  KNOX --> PENDING
  PENDING --> COBOOUT(["Outcome: MDM status<br/>'On (company-owned)' —<br/>IT fully controls settings,<br/>apps and data; IT CAN wipe<br/>the entire device"])
  ZT --> COBOOUT

  %% ---- PATH 2: work profile (BYOD) ----
  Q1 -->|"PATH 2 — personally-owned<br/>device (BYOD / BYO):<br/>work profile"| WP["IT admin shares an enrollment<br/>link (or QR code) — this URL<br/>does NOT include<br/>fully_managed=true"]
  WP --> WP2["User opens link in the device<br/>browser → taps 'Enroll' →<br/>accepts prompts to set up<br/>a work profile"]
  WP2 --> WP3["Work profile created: work<br/>apps show a briefcase icon;<br/>company data stays inside<br/>the profile"]
  WP3 --> WPOUT(["Outcome: MDM status<br/>'On (personal)' —<br/>data separation; IT manages<br/>ONLY the work profile:<br/>can lock the work profile and<br/>clear its passcode; CANNOT wipe<br/>the whole device or lock the<br/>user out of personal data.<br/>Unenrolling removes ONLY the<br/>work profile (AMAPI WIPE).<br/>Note: after unenrollment the<br/>status PERSISTS as<br/>'On (personal)' (vs 'Off' for<br/>company-owned)"])

  %% ---- Agent note ----
  AGENT["Fleet's Android agent app is<br/>installed silently via managed<br/>Google Play after enrollment —<br/>it adds setup experience and<br/>telemetry but does NOT change<br/>the MDM enrollment status"]
  AGENT -.-> COBOOUT
  AGENT -.-> WPOUT

  REQ -.-> Q1

  %% ---- No agent-based enrollment ----
  Q1 -.->|"NO manual enrollment /<br/>adb / standalone-agent path —<br/>all Android enrollment goes<br/>through the Android<br/>Management API"| AGENT

  %% Legend
  subgraph LEGEND["How to read"]
    L1["Solid = enrolment flow"] --- L2["Dashed = optional /<br/>intermediate state / caveat"]
  end

  classDef default fill:#ffffff,stroke:#94a3b8,stroke-width:1.5px,color:#1e293b,rx:10,ry:10
  classDef start fill:#1e293b,stroke:#0f172a,stroke-width:2px,color:#ffffff
  classDef decision fill:#ede9fe,stroke:#7c3aed,stroke-width:2px,color:#3b0764
  classDef pending fill:#fef9c3,stroke:#ca8a04,stroke-width:1.5px,color:#422006
  classDef full fill:#d1fae5,stroke:#059669,stroke-width:2.5px,color:#064e3b
  classDef byod fill:#dbeafe,stroke:#2563eb,stroke-width:2.5px,color:#1e3a8a
  classDef warn fill:#fee2e2,stroke:#dc2626,stroke-width:1.5px,color:#7f1d1d,stroke-dasharray:5 3

  class START start
  class Q1,COBO decision
  class PENDING,REQ pending
  class COBOOUT full
  class WPOUT byod
  class AGENT warn
  class L1,L2 pending

  linkStyle default stroke:#64748b,stroke-width:1.6px
  style LEGEND fill:#f8fafc,stroke:#cbd5e1,stroke-width:1px,color:#475569
  ```
