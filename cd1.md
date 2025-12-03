```mermaid
flowchart TB
    %% Define subgraphs for clarity
    subgraph OnPrem["On-Prem Production Environment"]
        APP[Application Servers]
        DB[(Primary Database Servers)]
        RMAN[RMAN Backup Jobs]
        VMSNAP[VM/Host Snapshots]
        BACKUP_REPO[(Local Backup Repository)]
    end

    subgraph AzureCold["Azure Cold DR Environment (Compute Off)"]
        ASRVAULT[(Azure Backup / Recovery Vault)]
        AZSTORAGE[(Azure Storage - Cool Tier)]
        DRVM[(DR VMs - Powered Off)]
        DRDB[(DR Database Servers - Powered Off)]
    end

    subgraph Event["Ransomware Attack Event"]
        RANSOMWARE[Ransomware Detected On-Prem]
        ISOLATE[Isolate On-Prem Network]
        LOCK_BACKUPS[Lock & Version Backups]
        NOTIFY[Notify Security & DR Team]
    end

    subgraph Recovery["Azure DR Restore Process"]
        RESTORE_DB[Restore Clean DB Backup to Azure]
        RESTORE_VM[Provision & Restore DR VMs in Azure]
        VALIDATE[Validate Clean State]
        DNS_CUTOVER[DNS Cutover to Azure]
        RESUME[Resume Business Operations in Azure]
    end

    %% Regular flow
    DB --> RMAN --> BACKUP_REPO
    BACKUP_REPO -->|Daily Sync| ASRVAULT
    BACKUP_REPO -->|Weekly Offsite| AZSTORAGE

    %% Ransomware event triggers
    RANSOMWARE --> ISOLATE --> LOCK_BACKUPS --> NOTIFY

    %% Azure recovery triggered
    LOCK_BACKUPS --> RESTORE_DB
    ASRVAULT --> RESTORE_DB
    AZSTORAGE --> RESTORE_DB

    RESTORE_DB --> RESTORE_VM --> VALIDATE --> DNS_CUTOVER --> RESUME
