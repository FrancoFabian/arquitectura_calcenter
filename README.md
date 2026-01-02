# 🏗️ Arquitectura Técnica
## Plataforma ISP + Hosting Estatal
**Ubicación principal:** San Martín Tilcajete  
**Horizonte:** 10–12 clientes en 12 meses 

---

## 🎯 Objetivo del sistema

Diseñar una infraestructura **estable, escalable y legalmente protegida** para:

- Operar como **proveedor de conectividad (ISP)**
- Ofrecer **hosting profesional de aplicaciones**
- Garantizar **continuidad ante apagones**
- Cumplir estándares de operación para **call center** y manejo de datos sensibles

---


## 🗺️ Mapa General de Arquitectura

```mermaid
graph TB
  %% =========================
  %% EXTERNO
  %% =========================
  Clients["Clientes Hosting (10–12)\nIncl. Gobierno del Estado\n[BR-10: Multi-tenant]"] --> Internet((Internet))
  Devs["Equipo DevOps / NOC\nAcceso remoto seguro\n[BR-07]\n[ISO27001 A.5.15]"] --> Internet

  %% =========================
  %% SEDE PRINCIPAL
  %% =========================
  subgraph Site["Sede Principal: San Martín Tilcajete (On-Prem DC/IDF)"]
    direction TB

    subgraph Power["Energía, continuidad y protección eléctrica"]
      UPS["UPS Online (Doble conversión) + Monitoreo\n[BR-01]\n[ISO27001 A.5.30]"]
      GEN["Planta/Generador + ATS (si SLA exige)\n[BR-02]\n[ISO22301 8.3/8.4]"]
      UPS --> GEN
    end

    ISP1["Enlace ISP #1\n[BR-03: Uplink primario]"]
    ISP2["Enlace ISP #2 (Failover)\n[BR-03: Uplink secundario]"]

    EDGE["MikroTik Edge/Core Router (CCR)\nBGP/OSPF, QoS, DDoS baseline\n[BR-04]\n[ISO27001 A.8.20]"]
    FW["Firewall/WAF (Perímetro)\nSegmentación + Políticas\n[BR-05]\n[ISO27001 A.8.20]"]
    SWCORE["Core Switch L3 + VLANs\nMgmt / DMZ / Apps / Datos / Voz\n[BR-06]\n[ISO27001 A.8.20]"]

    ISP1 --> EDGE
    ISP2 --> EDGE
    EDGE --> FW
    FW --> SWCORE

    %% -------------------------
    %% ZONA DMZ
    %% -------------------------
    subgraph DMZ["DMZ (Exposición controlada)"]
      direction TB
      LB["Load Balancer / Reverse Proxy\nTLS termination\n[BR-11]"]
      BASTION["Bastion / VPN Gateway\nAcceso Admin (MFA)\n[BR-07]\n[ISO27001 A.5.15]"]
      LB --- BASTION
    end
    SWCORE --> DMZ

    %% -------------------------
    %% ZONA APPS (CÓMPUTO)
    %% -------------------------
    subgraph APPS["Cluster de Cómputo x86 (Apps/Hosting)"]
      direction TB
      PX1["Proxmox/K8s Node-1 (x86)\n[BR-12: HA mínimo]"]
      PX2["Proxmox/K8s Node-2 (x86)\n[BR-12: HA mínimo]"]
      PX3["Proxmox/K8s Node-3 (x86)\n[BR-12: HA mínimo]"]

      RUNTIME["Runtime Apps\nNext.js / Astro / Angular SSR\nAPIs / Workers\n[BR-10: Multi-tenant]"]
      CICD["CI/CD Runner + Registry\nDeploy controlado\n[BR-08]"]

      PX1 --- PX2 --- PX3
      PX2 --> RUNTIME
      PX1 --> CICD
    end
    SWCORE --> APPS
    DMZ --> RUNTIME

    %% -------------------------
    %% ZONA DATOS (STORAGE/DB)
    %% -------------------------
    subgraph DATA["Capa de Datos (Producción)"]
      direction TB
      DB["DB Cluster (PostgreSQL)\nReplicación + PITR\n[BR-09]"]
      OBJ["S3 Interno (MinIO)\nAssets / Backups lógicos\n[BR-09]"]
      ROSE["MikroTik ROSE (RDS)\nNVMe Storage + NVMe/TCP iSCSI NFS\nData Services\n[BR-09]\n[ISO27001 A.8.13]"]

      DB --> ROSE
      OBJ --> ROSE
    end
    SWCORE --> DATA
    RUNTIME --> DB
    RUNTIME --> OBJ

    %% -------------------------
    %% OBSERVABILIDAD + LOGS
    %% -------------------------
    subgraph OBS["Observabilidad / Evidencia (NOC + Auditoría)"]
      direction TB
      LOGS["Central Logging (SIEM/Loki/ELK)\nRetención + Integridad\n[BR-13]\n[ISO27001 A.8.15]"]
      MON["Monitoreo (Prometheus/Grafana)\nAlertas NOC\n[BR-14]\n[ISO22301 9.1]"]
    end
    SWCORE --> OBS
    EDGE --> LOGS
    FW --> LOGS
    APPS --> LOGS
    DATA --> LOGS

    %% -------------------------
    %% BACKUP / DR (SEPARADO)
    %% -------------------------
    subgraph BACKUP["Backups y Recuperación (Separación obligatoria)"]
      direction TB
      BKS["Backup Server (Repo Inmutable/WORM)\nSnapshots + Copias + Restore tests\n[BR-15]\n[ISO27001 A.8.13]\n[ISO22301 8.4/8.5]"]
      OFFSITE["Offsite/DR (2ª ubicación o cloud)\nCopia fuera del sitio\n[BR-16]\n[ISO22301 8.3/8.4]"]
      BKS --> OFFSITE
    end
    DATA --> BKS
    APPS --> BKS

    %% -------------------------
    %% CALL CENTER (SISTEMAS SOPORTE)
    %% -------------------------
    subgraph CCC["Call Center (Operación)"]
      direction TB
      VOIP["PBX/VoIP + Grabaciones\n[BR-17]\n[ISO18295-1]"]
      CRM["CRM/Tickets\n[BR-18]\n[ISO18295-1]"]
    end
    SWCORE --> CCC
    CCC --> LOGS

  end

  %% =========================
  %% CONEXIÓN A SEDE
  %% =========================
  Internet --> ISP1
  Internet --> ISP2
```

---

## 🧾 Marco Normativo (ISO)

### ISO/IEC 27001:2022  [ver documentacion de ISO ](ISO27001_Implementacion.md)
- **A.5.15** Control de acceso
- **A.8.13** Respaldos de información
- **A.8.15** Registros y monitoreo
- **A.8.20** Seguridad de red

### ISO 22301:2019 [ver documentacion de ISO ](ISO22301_Implementacion.md)
- **8.3 – 8.5** Continuidad del negocio

### ISO 18295-1:2017 [ver documentacion de ISO ](ISO18295_Implementacion.md)
- Operación de centros de contacto

---

## 🧱 Componentes físicos requeridos

### Red y seguridad

| Equipo | Cantidad | Motivo | Precio MXN aprox |
|------|------|------|------|
| MikroTik CCR2116 | 1 | Core de red ISP | $18,000 – $21,000 |
| FortiGate 100F (Firewall empresarial) | 1 | Protección perimetral | $35,000 – $60,000 |
| Switch L3 10G | 1 | Segmentación de red | $9,000 – $12,000 |

### Energía

| Equipo | Cantidad | Motivo | Precio MXN |
|------|------|------|------|
| UPS Online 3000VA | 1–2 | Continuidad eléctrica | ~$53,000 |
| Generador + ATS | opcional | Continuidad extendida | $80,000+ |

### Cómputo

| Equipo | Cantidad | Motivo | Precio MXN |
|------|------|------|------|
| Servidor x86 rack | 3 | Cluster de aplicaciones | ~$110,000 c/u |
| NIC 10G | 3 | Alto rendimiento | $3,000 c/u |

### Almacenamiento y datos

| Equipo | Cantidad | Motivo | Precio MXN |
|------|------|------|------|
| MikroTik ROSE | 1 | Servicios de datos NVMe | ~$36,000 |
| NVMe enterprise | 6–12 | Bases y objetos | $6,000 – $12,000 c/u |

### Backups

| Equipo | Cantidad | Motivo | Precio MXN |
|------|------|------|------|
| Backup Server | 1 | Repositorio inmutable | $60,000+ |
| Offsite Backup | 1 | Recuperación ante desastre | variable |

---

## 🧭 Justificación de arquitectura

- La red, cómputo y datos están **separados** para evitar fallas en cascada.
- El cluster x86 permite crecimiento sin rediseñar el sistema.
- ROSE se usa como **capa de datos**, no como servidor principal de apps.
- Backups y DR reducen exposición legal y operativa.
- Cumple prácticas exigidas por **ISO 27001 / 22301 / 18295**.

---

##  Reglas de negocio clave

| Código | Regla |
|------|------|
| BR-01 | Ningún sistema crítico puede apagarse abruptamente |
| BR-03 | Enlaces de red redundantes |
| BR-08 | Todo despliegue pasa por CI/CD |
| BR-12 | Cluster mínimo de 3 nodos |
| BR-15 | Backups 3-2-1 obligatorios |
| BR-17 | Pruebas periódicas de restauración |

---

##  Revisar

1. [Ver documento : Definir RTO / RPO por servicio](RTO_RPO.md)  
2. [Ver docuemnto : Clasificar datos críticos](Clasificacion_Datos_Criticos.md)  
3. [Ver documento : Establecer SLAs por cliente](SLA_Guia_Implementacion.md)  
4. [Ver documento: Políticas de Seguridad](Politicas_Seguridad_Guia.md)
