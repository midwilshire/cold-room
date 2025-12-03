```mermaid
flowchart TB

%% ======================================
%% ON-PREM LAYER
%% ======================================
subgraph OnPrem ["On-Prem Environment"]
    OPDB[Primary Oracle DB]
    OPEXADATA[Exadata or Oracle RAC]
    OPGGExtract[GoldenGate Extract Process]
    OPDGPrimary[Data Guard Primary Role]
    OPNet[On-Prem Firewall and Network]
end

%% ======================================
%% REPLICATION CHANNELS
%% ======================================
subgraph Replication ["Replication Channels"]
    GGTrail[GoldenGate Trail Files]
    DGLCR[Redo Log Shipping Async]
end

%% ======================================
%% AZURE LAYER - ORACLE DATABASE @ AZURE
%% ======================================
subgraph AzureOracle ["Oracle Database at Azure"]
    AZExadata[Azure Exadata DB Service]
    AZStandbyDB[Standby Oracle DB]
    AZGGReplicator[GoldenGate Replicat Process]
    AZDGStandby[Data Guard Standby Role]
    AZMon[Oracle Monitoring and Health Checks]
end

%% ======================================
%% NETWORK + SECURITY
%% ======================================
subgraph NetworkSecurity ["Networking and Security"]
    Interconnect[Azure OCI Interconnect]
    ExpressRoute[ExpressRoute or VPN]
    AzureFirewall[Azure Firewall]
    AAD[Azure AD Authentication]
end

%% ======================================
%% DATA FLOW CONNECTIONS
%% ======================================
OPDB --> OPGGExtract --> GGTrail --> AZGGReplicator --> AZStandbyDB
OPDB --> OPDGPrimary --> DGLCR --> AZDGStandby --> AZStandbyDB

OnPrem --> OPNet --> ExpressRoute --> Interconnect --> AzureOracle

AAD --> AzureOracle
AzureFirewall --> AzureOracle
