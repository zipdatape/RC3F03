graph TB
    subgraph "Plataforma de Virtualización"
        VC[vCenter Server]
        VM1[VM 1]
        VM2[VM 2]
        VM3[VM N]
    end

    subgraph "Soluciones de Backup"
        VB[Veeam Backup]
        GH[Ghetto VCB<br/>Sistema Propio]
    end

    subgraph "Almacenamiento TrueNAS"
        TN[TrueNAS Server]
        subgraph "Repositorio Veeam"
            VR1[Versión 1]
            VR2[Versión 2]
        end
        subgraph "Repositorio Ghetto"
            GR1[Versión 1]
            GR2[Versión 2]
        end
        SPACE[Storage Disponible]
    end

    VC --> VM1
    VC --> VM2
    VC --> VM3

    VM1 -.->|Backup| VB
    VM2 -.->|Backup| VB
    VM3 -.->|Backup| VB

    VM1 -.->|Backup| GH
    VM2 -.->|Backup| GH
    VM3 -.->|Backup| GH

    VB -->|Se conecta a vCenter| VC
    GH -->|Se conecta a vCenter| VC

    VB -->|Almacena respaldos| VR1
    VB -->|Rotación| VR2
    
    GH -->|Almacena respaldos| GR1
    GH -->|Rotación| GR2

    VR1 --> TN
    VR2 --> TN
    GR1 --> TN
    GR2 --> TN
    SPACE --> TN

    style VC fill:#0072C6,color:#fff
    style VB fill:#00B336,color:#fff
    style GH fill:#FF6B35,color:#fff
    style TN fill:#0095D5,color:#fff
    style VR1 fill:#90EE90
    style VR2 fill:#90EE90
    style GR1 fill:#FFB347
    style GR2 fill:#FFB347
    style SPACE fill:#E0E0E0
