```mermaid
flowchart TB

    %% On-Prem Environment
    subgraph OnPrem["On-Prem Production"]
        APP[App Servers]
        DB[Primary Databases]
        RMAN[RMAN Backup Jobs]
        SNAP[VM Snapshots]
        BACKUP_REPO[Local Backup Repository]
    end

    %% Azure Backup
    subgraph AzureBackup["Azure Backup Storage"]
        ASRVAULT[Azure Recovery Vault]
        AZSTORAGE[Azure Storage Archive]
    end

    %% Ransomware Event
    subgraph Event["Ransomware Event Response"]
        RANSOMWARE[Ransomware Detected]
        ISOLATE[Isolate On-Prem Network]
        IMMUTABLE[Seal Backups Immutable]
        NOTIFY[Notify Security Team]
        FORENSICS[Begin Forensic Capture]
    end

    %% Azure Provisioning
    subgraph Provision["Azure DR Provisioning"]
        RG[Create DR Resource Group]
        VNET[Deploy DR VNET and Subnets]
        NSG[Apply NSG and Firewall Rules]
        KV[Restore Azure Key Vault Secrets]
        IMAGES[Prepare Golden VM Images]
    end

    %% Restore Stage
    subgraph Restore["Azure Restore Operations"]
        DRDB[Restore Database Server]
        DRAPP[Deploy DR App Servers]
        CONFIG[Restore Config and Middleware]
        IDENTITY[Rebuild Service Identities]
        LB[Deploy Load Balancer]
    end

    %% Validation
    subgraph DRLive["DR Validation and Cutover"]
        VALIDATE[Validate Application]
        DNS_CUTOVER[DNS Cutover to Azure]
        OPERATIONS[Resume Operations in Azure]
    end

    %% Backup flow to Azure
    DB --> RMAN --> BACKUP_REPO
    BACKUP_REPO --> ASRVAULT
    BACKUP_REPO --> AZSTORAGE
    SNAP --> BACKUP_REPO

    %% Incident response flow
    RANSOMWARE --> ISOLATE --> IMMUTABLE --> NOTIFY --> FORENSICS

    %% Azure provisioning flow
    IMMUTABLE --> RG --> VNET --> NSG --> KV --> IMAGES

    %% Restore flow
    ASRVAULT --> DRDB
    AZSTORAGE --> DRDB
    IMAGES --> DRAPP
    KV --> CONFIG
    DRDB --> DRAPP
    DRAPP --> CONFIG
    CONFIG --> IDENTITY --> LB

    %% Final validation
    LB --> VALIDATE --> DNS_CUTOVER --> OPERATIONS
