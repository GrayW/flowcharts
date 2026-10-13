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
  START(["New or existing iPhone / iPad"]) --> Q1{"How is the device being<br/>enrolled?"}

  %% ---- PATH 1: ADE via Apple Business (company-owned) ----
  Q1 -->|"PATH 1 — ADE zero-touch:<br/>company-owned, bought via<br/>Apple / authorised reseller<br/>(serial in AB)"| ADE["Device assigned to Fleet<br/>MDM server in Apple Business"]
  ADE --> PENDING["Host syncs to Fleet<br/>as 'Pending'"]
  PENDING --> ADE2["On first boot + internet,<br/>activation servers point<br/>device at Fleet"]
  ADE2 --> ADE3["Setup Assistant runs —<br/>panes can be customised<br/>(Controls &gt; Setup experience &gt;<br/>Setup Assistant); IdP auth<br/>can be enforced"]
  ADE3 --> ADE4["Device becomes supervised;<br/>MDM profile non-removable<br/>(IsMDMRemovable = false by default;<br/>custom Setup Assistant profile<br/>can override)"]
  ADE4 --> ADE5["Fleet can auto-install VPP<br/>apps during enrollment<br/>(InstallApplication command;<br/>keep setup light, add more later)"]
  ADE5 --> ADEOUT(["Outcome: MDM status<br/>'On (automatic)' — supervised,<br/>no fleetd on iOS (MDM-only:<br/>29 vitals via DeviceInformation,<br/>incl. battery, cellular, backup,<br/>attestation). IT can wipe;<br/>LOCK REQUIRES SUPERVISION.<br/>Lock collects location.<br/>(shown as 'On (company-owned)')"])

  %% ---- PATH 2: Apple Configurator ----
  Q1 -->|"PATH 2 — existing device,<br/>not in AB: Apple Configurator<br/>adds serial to AB<br/>(needs physical access<br/>+ WIPE, then re-setup)"| CON["Device enters PATH 1<br/>ADE flow after re-setup"]
  CON --> ADE

  %% ---- PATH 3: ADUE (personal / BYOD) ----
  Q1 -->|"PATH 3 — Account-driven User<br/>Enrollment (personal / BYOD):<br/>user's own device"| ADUE["Admin one-time: connect AB,<br/>verify domain, connect IdP,<br/>create fleet for personal hosts.<br/>User: Settings &gt; General &gt; VPN &<br/>Device Management &gt; Sign in to<br/>Work or School Account (IdP<br/>credentials)"]
  ADUE --> ADUE2["iOS/iPadOS creates separate<br/>volumes for personal and<br/>work data (data separation)"]
  ADUE2 --> ADUEOUT(["Outcome: MDM status<br/>'On (personal)' — unsupervised,<br/>data separation; IT manages<br/>work settings/apps only;<br/>IT CANNOT wipe or lock.<br/>Privacy: personal BYOD hosts<br/>do NOT receive the extended<br/>MDM vitals; software inventory<br/>only for Fleet-installed apps"])

  %% ---- PATH 4: manual BYOD enroll link ----
  Q1 -->|"PATH 4 — manual BYOD<br/>enroll link<br/>(/enroll?enroll_secret=...<br/>&byod=true): personal device,<br/>no AB / ADUE setup"| MBYOD["User taps enroll link →<br/>Allow profile download →<br/>Settings &gt; Profile Downloaded →<br/>Install → Trust on Remote<br/>Management screen"]
  MBYOD --> MBYODOUT(["Outcome: MDM status<br/>'On (manual - personal)' —<br/>unsupervised; IT can only<br/>remotely remove work data,<br/>CANNOT wipe or lock.<br/>Device has no serial number<br/>listed in Fleet (BYOD privacy)"])

  %% ---- PATH 5: manual company-owned profile ----
  Q1 -->|"PATH 5 — manual company-owned<br/>profile install (not in AB,<br/>device stays company-owned)"| MCO["User follows manual profile<br/>install steps (same taps as<br/>PATH 4); user informed org<br/>can see and delete all<br/>device information"]
  MCO --> MCOOUT(["Outcome: MDM status<br/>'On (manual)' — typically<br/>unsupervised (unless Apple<br/>Configurator was used for<br/>initial setup); IT CAN wipe<br/>and enforce all MDM restrictions"])

  %% Legend
  subgraph LEGEND["How to read"]
    L1["Solid = enrolment flow"] --- L2["Dashed = optional /<br/>intermediate state / caveat"]
  end

  classDef default fill:#ffffff,stroke:#94a3b8,stroke-width:1.5px,color:#1e293b,rx:10,ry:10
  classDef start fill:#1e293b,stroke:#0f172a,stroke-width:2px,color:#ffffff
  classDef decision fill:#ede9fe,stroke:#7c3aed,stroke-width:2px,color:#3b0764
  classDef pending fill:#fef9c3,stroke:#ca8a04,stroke-width:1.5px,color:#422006
  classDef full fill:#d1fae5,stroke:#059669,stroke-width:2.5px,color:#064e3b
  classDef manual fill:#ffedd5,stroke:#ea580c,stroke-width:2.5px,color:#7c2d12
  classDef byod fill:#dbeafe,stroke:#2563eb,stroke-width:2.5px,color:#1e3a8a
  classDef warn fill:#fee2e2,stroke:#dc2626,stroke-width:1.5px,color:#7f1d1d,stroke-dasharray:5 3

  class START start
  class Q1 decision
  class PENDING pending
  class ADEOUT,MCOOUT full
  class ADUEOUT,MBYODOUT byod
  class CON warn
  class L1,L2 pending

  linkStyle default stroke:#64748b,stroke-width:1.6px
  style LEGEND fill:#f8fafc,stroke:#cbd5e1,stroke-width:1px,color:#475569
  ```
