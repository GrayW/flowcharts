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
  START(["New or existing Windows PC"]) --> Q1{"How is the device being<br/>enrolled?"}

  %% ---- PATH 1: Windows Autopilot ----
  Q1 -->|"PATH 1 — Windows Autopilot:<br/>company-owned device<br/>registered with Autopilot<br/>(hardware hash CSV, or<br/>OEM / reseller registers it)"| AP["Autopilot deployment profile<br/>assigned (authored in Intune<br/>admin center). Fleet connects to<br/>Microsoft Graph to read the<br/>tenant's Autopilot registry"]
  AP --> PENDING["Host appears in Fleet<br/>as 'Pending' before<br/>enrollment"]
  PENDING --> AP2["On first boot / reset,<br/>OOBE contacts the Autopilot<br/>deployment service"]
  AP2 --> AP3["User signs in with<br/>Microsoft Entra ID —<br/>device joins Entra ID and<br/>enrolls in Fleet MDM"]
  AP3 --> AP4["Enrollment Status Page can<br/>hold the device while setup<br/>experience applies software<br/>and profiles; first user can<br/>be forced to STANDARD user<br/>(not local admin); BitLocker<br/>managed via MDM"]
  AP4 --> APOUT(["Outcome: MDM status<br/>'On (company-owned)' —<br/>REQUIRES Microsoft Entra ID;<br/>each enrolling user needs an<br/>Entra ID P1 license. Intune<br/>license NOT required to<br/>enroll in Fleet"])
  AP -.->|"Existing Windows 10/11 PC:<br/>register hardware hash<br/>(no wipe needed for<br/>Autopilot, but full OOBE<br/>flow runs at next reset)"| APOUT

  %% ---- PATH 2: manual fleetd MSI ----
  Q1 -->|"PATH 2 — manual software<br/>deployment"| PKG["Generate installer:<br/>fleetctl package --type msi<br/>--fleet-url=... --enroll-secret=...<br/>(optional --fleet-desktop)"]
  PKG --> PKG2["Run MSI on the device<br/>(or deploy via provisioning<br/>package before the user<br/>completes Windows setup)"]
  PKG2 --> PKG3["Installs orbit + osqueryd<br/>(+ Fleet Desktop)"]
  PKG3 --> AGENTOUT(["Outcome: osquery-enrolled host<br/>(telemetry, queries, policies) —<br/>MDM status 'Off' unless<br/>MDM turned on separately"])

  %% ---- PATH 3: plain osquery ----
  Q1 -->|"PATH 3 — DIY osqueryd"| OSQ["Run osqueryd with flags:<br/>--enroll_secret_path,<br/>--tls_hostname, --tls_server_certs,<br/>--enroll_tls_endpoint=/api/osquery/enroll,<br/>plus TLS plugins:<br/>--config_plugin=tls, --logger_plugin=tls,<br/>--distributed_plugin=tls"]
  OSQ --> OSQOUT(["Outcome: osquery-enrolled host,<br/>no auto-updates, no orbit,<br/>no Fleet Desktop;<br/>MDM status 'Off'"])

  %% ---- PATH 4: manual MDM via Settings ----
  Q1 -->|"PATH 4 — manual MDM<br/>enrollment without Autopilot<br/>(Fleet connected to<br/>Microsoft Entra ID)"| WM["User: Settings &gt; Accounts &gt;<br/>Access work or school &gt;<br/>Connect — enters work email;<br/>Fleet discovered via Entra ID<br/>(Fleet collects the user's email<br/>as IdP username)"]
  WM --> WM2["User prompted to set up<br/>Windows Hello and add a PIN"]
  WM2 --> WMOUT(["Outcome: MDM status<br/>'On (manual)'. CAVEAT: on<br/>Windows, MDM enrollment<br/>happens in the context of a<br/>signed-in user — status stays<br/>'Off' until an end user signs<br/>in (MDM certificate lives in<br/>the SYSTEM account's store)"])

  %% ---- PATH 5: migrating from Intune / other MDM ----
  Q1 -->|"PATH 5 — migrating from<br/>Microsoft Intune / other<br/>Windows MDM"| MIG["1. Set up Windows MDM in Fleet<br/>2. Install Fleet's agent (fleetd)<br/>3. Enable automatic migration<br/>in Fleet settings"]
  MIG --> MIG2["Fleet sends a notification<br/>to each host to initiate<br/>migration"]
  MIG2 --> MIGOUT(["Outcome: enrolled in Fleet MDM.<br/>MAY initially report MDM not 'On'<br/>due to leftover enrollment data or<br/>third-party management agents —<br/>run fix-windows-mdm-migration.ps1<br/>and reboot to resolve. Users may<br/>hit enrollment errors or locked<br/>local accounts if issues occur"])

  %% Legend
  subgraph LEGEND["How to read"]
    L1["Solid = enrolment flow"] --- L2["Dashed = optional /<br/>intermediate state / caveat"]
  end

  classDef default fill:#ffffff,stroke:#94a3b8,stroke-width:1.5px,color:#1e293b,rx:10,ry:10
  classDef start fill:#1e293b,stroke:#0f172a,stroke-width:2px,color:#ffffff
  classDef decision fill:#ede9fe,stroke:#7c3aed,stroke-width:2px,color:#3b0764
  classDef pending fill:#fef9c3,stroke:#ca8a04,stroke-width:1.5px,color:#422006
  classDef full fill:#d1fae5,stroke:#059669,stroke-width:2.5px,color:#064e3b
  classDef agent fill:#dbeafe,stroke:#2563eb,stroke-width:2.5px,color:#1e3a8a
  classDef warn fill:#fee2e2,stroke:#dc2626,stroke-width:1.5px,color:#7f1d1d,stroke-dasharray:5 3
  classDef manual fill:#ffedd5,stroke:#ea580c,stroke-width:2.5px,color:#7c2d12

  class START start
  class Q1 decision
  class PENDING pending
  class APOUT,MIGOUT full
  class AGENTOUT,OSQOUT agent
  class WMOUT manual
  class L1,L2 pending

  linkStyle default stroke:#64748b,stroke-width:1.6px
  style LEGEND fill:#f8fafc,stroke:#cbd5e1,stroke-width:1px,color:#475569