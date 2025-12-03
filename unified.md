```mermaid
flowchart TB

%% ============================
%% ON-PREM ENVIRONMENT
%% ============================
subgraph OnPrem["On-Prem Production"]
    THICK[Thick Client App]
    RDS[Terminal Services]
    THIN[Thin Client Browser]
    API[API Servers]
    ORADB[Oracle Database]
    AD[Active Directory]
    ADCONNECT[AD Connect Sync]
    RMAN[RMAN Backup Jobs]
end

%% ============================
%% AZURE PLATFORM BASELINE
%% ============================
subgraph Azure["Azure Platform"]
    HUB[Hub VNET]
    SPOKEAPP[App Spoke VNET]
    SPOKEDR[DR Spoke VNET]
    FW[Azure Firewall]
    KV[Key Vault]
    IMG[Golden VM Images]
    TF[Terraform Templates]
    STORAGE[Azure Storage Immutable]
    VAULT[Recovery Vault]
    ENTRA[Entra ID]
end

%% ============================
%% COLD DR WORKFLOW
%% ============================
subgraph Cold["Cold DR Workflow"]
    C1[Upload RMAN Backups to Azure]
    C2[Validate Backups in Isolated Subnet]
    C3[Deploy App Servers On Demand]
    C4[Deploy Oracle DB On Demand]
    C5[Restore RMAN Backup to Oracle in Azure]
    C6[Cutover DNS]
end

%% ============================
%% WARM DR WORKFLOW
%% ============================
subgraph Warm["Warm DR Workflow"]
    W1[Provision Oracle Standby in Azure]
    W2[Provision DR App Tier Powered Off]
    W3[Replication via GoldenGate or DataGuard]
    W4[Power On DR App Tier]
    W5[Switch Standby to Primary]
    W6[DNS Failover]
end

%% ============================
%% HOT DR WORKFLOW
%% ============================
subgraph Hot["Hot DR Workflow"]
    H1[Fully Live Replication]
    H2[Active Data Guard]
    H3[Active App Servers Warm Standby]
    H4[Traffic Managed by Global LB]
end

%% ============================
%% CONNECTIONS
%% ============================

%% Client flows
THICK --> RDS --> API
THIN --> API
API --> ORADB

%% Identity flows
AD --> ADCONNECT --> ENTRA

%% Network flows
OnPrem --> HUB --> FW --> SPOKEAPP
HUB --> SPOKEDR

%% Backup flows
RMAN --> STORAGE
RMAN --> VAULT

%% Cold DR
STORAGE --> C2 --> C3 --> C4 --> C5 --> C6

%% Warm DR
STORAGE --> W1
W1 --> W3 --> W2
W2 --> W4 --> W5 --> W6

%% Hot DR
W3 --> H1 --> H2 --> H3 --> H4
