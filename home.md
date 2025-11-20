flowchart TD

    A[Oracle Exadata] --> B[RMAN backups created]
    B --> C[Local backup storage]
    C --> D[Upload to Azure Blob Storage]

    D --> E[Azure VM (Cold Standby)]
    E --> F[Download backups from Blob Storage]
    F --> G[RMAN: catalog backups]
    G --> H[RMAN: restore database]
    H --> I[RMAN: recover database]

    I --> J[DR Database Online in Azure]
