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
  START(["New or existing Windows PC"]) --> Q1{"How is the device being<br/>enrolled?"}

  REQ["PREREQUISITE for MDM paths:<br/>Windows MDM turned on in Fleet<br/>(WSTEP certificate + key).<br/>Windows Pro / Enterprise /<br/>Education required —<br/>Windows Home CANNOT enroll"]

  %% ---- PATH 1: Entra ID automatic enrollment at OOBE ----
  Q1 -->|"PATH 1 — Entra ID automatic<br/>enrollment at OOBE<br/>(Premium; the base zero-touch<br/>flow — Autopilot is a<br/>customization on top)"| ENTRA["User picks 'Set up for<br/>work or school' at OOBE and<br/>signs in with Microsoft<br/>Entra ID (Fleet connected<br/>to Entra ID, automatic<br/>enrollment configured)"]
  ENTRA --> ESP["Enrollment Status Page holds<br/>the device while setup<br/>experience applies software<br/>and profiles before the<br/>user reaches the desktop"]
  ESP --> ENTRAOUT(["Outcome: MDM status<br/>'On (company-owned)' —<br/>device joins Entra ID +<br/>enrolls in Fleet MDM.<br/>Each enrolling user needs<br/>an Entra ID P1 license<br/>(Intune license NOT needed;<br/>Autopilot profiles are<br/>authored in the Intune<br/>admin center)"])

  %% ---- PATH 1b: Autopilot branch ----
  Q1 -->|"PATH 1b — + Windows Autopilot:<br/>device registered (hardware<br/>hash CSV or OEM / reseller)<br/>+ deployment profile assigned.<br/>USER-DRIVEN MODE ONLY<br/>(self-deploying and<br/>pre-provisioning NOT supported)"| AP["On first boot / reset,<br/>OOBE contacts the Autopilot<br/>deployment service —<br/>existing PCs only pick this<br/>up after a reset / reimage,<br/>since the profile applies<br/>during OOBE"]
  AP --> AP2["User signs in with<br/>Microsoft Entra ID —<br/>device joins Entra ID and<br/>enrolls in Fleet MDM"]
  AP2 --> ESP
  AP -.->|"OPTIONAL Graph sync (Premium):<br/>app registration with<br/>DeviceManagementServiceConfig.<br/>Read.All; syncs every 5 min;<br/>matched by serial number,<br/>placeholder serials skipped"| PENDING["Host appears in Fleet<br/>as 'Pending' before<br/>enrollment"]

  %% ---- PATH 2: manual fleetd MSI ----
  Q1 -->|"PATH 2 — manual software<br/>deployment"| PKG["Generate installer:<br/>fleetctl package --type msi<br/>--fleet-url=... --enroll-secret=...<br/>(optional --fleet-desktop)"]
  PKG --> PKG2["Run MSI on the device,<br/>or deploy via provisioning<br/>package to preinstall fleetd<br/>before the user finishes<br/>Windows setup (aimed at<br/>devices that DON'T join Entra)"]
  PKG2 --> PKG3["Installs orbit + osqueryd<br/>(+ Fleet Desktop);<br/>osquery telemetry flows<br/>immediately"]
  PKG3 --> AGENTOUT(["Outcome: osquery-enrolled host.<br/>With 'Turn on MDM programmatically'<br/>enabled (documented best practice),<br/>Fleet turns on MDM for these hosts —<br/>BUT Windows MDM enrollment completes<br/>in a USER context: status stays 'Off'<br/>at the lock screen until someone signs<br/>in (Fleet retries and completes within<br/>~30 seconds of the next sign-in; MDM<br/>cert in SYSTEM store). With it disabled,<br/>MDM stays 'Off' — use PATH 4 instead.<br/>fleetd-first hosts keep the fleet from<br/>their enroll secret, not the default fleet"])

  %% ---- PATH 3: plain osquery ----
  Q1 -->|"PATH 3 — DIY osqueryd"| OSQ["Run osqueryd with flags:<br/>--enroll_secret_path,<br/>--tls_hostname, --tls_server_certs,<br/>--enroll_tls_endpoint=/api/osquery/enroll,<br/>plus TLS plugins:<br/>--config_plugin=tls, --logger_plugin=tls,<br/>--distributed_plugin=tls"]
  OSQ --> OSQOUT(["Outcome: osquery-enrolled host,<br/>no auto-updates, no orbit,<br/>no Fleet Desktop;<br/>MDM status 'Off'"])

  %% ---- PATH 4: manual MDM via Settings ----
  Q1 -->|"PATH 4 — manual MDM enrollment<br/>(user is signed in;<br/>also the fallback when<br/>'Turn on MDM programmatically'<br/>is disabled)"| WM["User: Settings &gt; Accounts &gt;<br/>Access work or school &gt;<br/>Connect — enters work email;<br/>Fleet discovered via Entra ID<br/>(Fleet collects the user's email<br/>as IdP username)"]
  WM --> WM2["User prompted to set up<br/>Windows Hello and add a PIN"]
  WM2 --> WMOUT(["Outcome: MDM status<br/>'On (manual)'. Default MDM<br/>fleet applies when hosts<br/>enroll in MDM BEFORE fleetd<br/>is installed (the Autopilot<br/>order)"])

  %% ---- PATH 5: migrating from Intune / other MDM ----
  Q1 -->|"PATH 5 — migrating from<br/>Microsoft Intune / other<br/>Windows MDM"| MIG["1. Set up Windows MDM in Fleet<br/>2. Install Fleet's agent (fleetd)<br/>3. Enable automatic migration<br/>in Fleet settings"]
  MIG --> MIG2["Fleet sends a notification<br/>to each host to initiate<br/>migration"]
  MIG2 --> MIGOUT(["Outcome: enrolled in Fleet MDM.<br/>MAY initially report MDM not 'On'<br/>due to leftover enrollment data or<br/>third-party management agents —<br/>run fix-windows-mdm-migration.ps1<br/>and reboot to resolve. Users may<br/>hit enrollment errors or locked<br/>local accounts if issues occur"])

  REQ -.-> ENTRA
  REQ -.-> AP
  REQ -.-> WM

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
  classDef manual fill:#ffedd5,stroke:#ea580c,stroke-width:2.5px,color:#7c2d12
  classDef warn fill:#fee2e2,stroke:#dc2626,stroke-width:1.5px,color:#7f1d1d,stroke-dasharray:5 3

  class START start
  class Q1 decision
  class PENDING,REQ pending
  class ENTRAOUT,MIGOUT full
  class AGENTOUT,OSQOUT agent
  class WMOUT manual
  class L1,L2 pending

  linkStyle default stroke:#64748b,stroke-width:1.6px
  style LEGEND fill:#f8fafc,stroke:#cbd5e1,stroke-width:1px,color:#475569
  ```
