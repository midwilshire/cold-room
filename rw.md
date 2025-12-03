```mermaid
flowchart TB

%% ============================
%% ON-PREM ENVIRONMENT
%% ============================
subgraph OnPrem["On-Prem Environment"]
    subgraph Clients["Client Access"]
        THICK[Thick Client Application]
        RDS[Terminal Services for Thick Client]
        THIN[Thin Client Browser React App]
    end

    subgraph OnPremApp["On-Prem Application Layer"]
        API[API Server for Thin Client]
        APP_LOGIC[Business Logic Services]
    end

    subgraph OnPremIdentity["On-Prem Identity"]
        AD[Active Directory]
        ADCONNECT[AD Connect Sync]
    end

    subgraph OnPremDB["On-Prem Database"]
        ORADB[Oracle Database]
        RMAN[RMAN Backup Jobs]
    end
end

%% ============================
%% AZURE ENVIRONMENT
%% ============================
subgraph Azure["Azure Environment"]
    
    subgraph Identity["Identity Layer"]
        ENTRA[Microsoft Entra ID]
    end

    subgraph Network["Azure Network Hub and Spoke"]
        HUB[Azure Hub VNET]
        SPOKEAPP[Application Spoke VNET]
        SPOKEDR[DR Spoke VNET]
        FW[Azure Firewall]
    end

    subgraph AppTier["Azure Application Tier"]
        APIDR[API DR Server]
        APPDR[App DR Services]
    end

    subgraph DRTier["Azure DR Database"]
        ORADR[Oracle Database at Azure]
        STORAGE[Azure Storage Backup]
        VAULT[Azure Recovery Vault]
    end

end

%% ============================
%% CONNECTIONS
%% ============================

%% Client access flows
THICK --> RDS
RDS --> APP_LOGIC
THIN --> API

%% App to DB
APP_LOGIC --> ORADB
API --> ORADB

%% Identity sync
AD --> ADCONNECT --> ENTRA

%% VPN/ExpressRoute
OnPrem --> HUB

%% Hub to spokes
HUB --> SPOKEAPP
HUB --> SPOKEDR
FW --> SPOKEAPP
FW --> SPOKEDR

%% DR data flows
RMAN --> STORAGE
RMAN --> VAULT

%% DR DB restore
STORAGE --> ORADR
VAULT --> ORADR

%% DR app rebuild
ORADR --> APIDR
ORADR --> APPDR
APIDR --> THIN
APPDR --> THICK
