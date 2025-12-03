```mermaid
flowchart TB

    %% On-Premises Environment
    subgraph OnPrem["On-Prem Production Environment"]
        APP[App Servers]
        DB[(Primary Databases)]
        RMAN[RMAN Backup Jobs]
        SNAP[VM Snapshots / File System Backups]
        BACKUP_REPO[(Local Backup Repository)]
    end

    %% Azure Cold DR Backup Storage
    subgraph AzureBackup["Azure Backup & Storage (Always On – Low Cost)"]
        ASRVAULT[(Azure Recovery Vault)]
        AZSTORAGE[(Azure Storage - Cool/Archive Tier)]
    end

    %% Ransomware Event
    subgraph Event["Ransomware Event"]
        RANSOMWARE[Ransomware Detected On-Prem]
        ISOLATE[Isolate On-Prem Network]
        IMMUTABLE[Lock/Seal Backups (Immutable)]
        NOTIFY[Notify Security/SOC & DR Leadership]
        FORENSICS[Begin Forensic Snapshot Collection]
    end

    %% Azure DR Provisioning
    subgraph Provision["Azure DR Provisioning"]
        RG[Create DR Resource Group]
        VNET[Deploy DR Virtual Network + Subnets]
        NSG[Apply NSGs + Zero-Trust Firewall Rules]
        KV[Restore Secrets to Azure Key Vault]
        IMAGES[Prepare Golden Images / VM Templates]
    end

    %% Azure Restore Layer
    subgraph Restore["Azure Compute + Data Restore"]
        DRDB[(DR Database Server - Restore Backup)]
        DRAPP[DR App Servers - Rebuild/Restore]
        CONFIG[Restore Config Files / Middleware]
        IDENTITY[Rebuild Service Principals / Managed Identity]
        LB[Deploy Azure Load Balancer / App Gateway]
    end

    %% DR Validation & Cutover
    subgraph DRLive["DR Validation & Go-Live"]
        VALIDATE[Application Validation & Security Scan]
        DNS_CUTOVER[DNS Cutover → Azure]
        OPERATIONS[Resume Business Operations in Azure]
    end

    %% Regular backup flow
    DB --> RMAN --> BACKUP_REPO
    BACKUP_REPO -->|Daily Backup Replication| ASRVAULT
    BACKUP_REPO -->|Weekly Offsite Copy| AZSTORAGE
    SNAP --> BACKUP_REPO

    %% Attack workflow
    RANSOMWARE --> ISOLATE --> IMMUTABLE --> NOTIFY --> FORENSICS

    %% Provisioning after backups locked
    IMMUTABLE --> RG --> VNET --> NSG --> KV --> IMAGES

    %% Restore logic
    ASRVAULT --> DRDB
    AZSTORAGE --> DRDB
    IMAGES --> DRAPP
    KV --> CONFIG
    DRDB --> DRAPP
    DRAPP --> CONFIG
    CONFIG --> IDENTITY --> LB

    %% Final flow
    LB --> VALIDATE --> DNS_CUTOVER --> OPERATIONS
