# 🗂️ Clasificación de Datos Críticos  
## Marco de decisión para la plataforma ISP + Hosting Estatal

---

## 🎯 Objetivo

Definir **qué datos son más importantes**, **cómo deben protegerse**,  
y **qué nivel de inversión requiere cada tipo_pct de información**.

Esta clasificación permite:
- priorizar infraestructura,
- definir backups y continuidad,
- reducir riesgos legales,
- justificar decisiones técnicas y presupuestales.

---

## 🧱 Principio clave

> **No todos los datos valen lo mismo.**  
> La arquitectura se diseña protegiendo **primero lo que más daño causaría perder**.

---

## 🧭 Niveles de clasificación propuestos

| Nivel | Nombre | Descripción |
|------|--------|------------|
| **Nivel 1** | **Datos Críticos** | Pérdida inaceptable: afecta operación, contratos o gobierno |
| **Nivel 2** | **Datos Importantes** | Afectan servicio y reputación |
| **Nivel 3** | **Datos Operativos** | Impacto moderado |
| **Nivel 4** | **Datos No Críticos** | Impacto bajo |

---

## 🟥 Nivel 1 – Datos Críticos

### Incluye:
- Bases de datos del gobierno
- Información personal de usuarios
- Grabaciones del call center
- Contratos y facturación
- Logs de seguridad y auditoría

### Riesgo si se pierden:
- Riesgo legal grave
- Incumplimiento contractual
- Pérdida de confianza institucional

### Requerimientos técnicos:
- Replicación activa
- Backups 3-2-1
- Repositorio inmutable (WORM)
- Copia offsite
- Cifrado
- Control de accesos estricto
- RPO ≤ 15 minutos
- RTO ≤ 30 minutos

---

## 🟧 Nivel 2 – Datos Importantes

### Incluye:
- Código fuente
- Configuraciones
- Historial de tickets
- Métricas de clientes

### Requerimientos:
- Backups diarios
- Snapshots
- Copia externa
- RPO ≤ 1 hora
- RTO ≤ 2 horas

---

## 🟨 Nivel 3 – Datos Operativos

### Incluye:
- Logs históricos antiguos
- Reportes internos
- Archivos de soporte

### Requerimientos:
- Backups semanales
- Retención limitada
- RPO ≤ 24 horas
- RTO ≤ 8 horas

---

## 🟩 Nivel 4 – Datos No Críticos

### Incluye:
- Material temporal
- Pruebas
- Archivos obsoletos

### Requerimientos:
- Backups opcionales
- Retención corta
- RPO/RTO no críticos

---

## 🧩 Cómo se implementa en la arquitectura

| Capa | Nivel de datos | Estrategia |
|----|----|----|
Bases de datos | Nivel 1 | Replicación + PITR + backup inmutable |
Storage ROSE | Nivel 1–2 | Snapshots + cifrado |
Backups | Todos | 3-2-1 + offsite |
Cluster apps | Nivel 2–3 | Backups regulares |
Archivos internos | Nivel 3–4 | Retención controlada |

---

## ⚖️ Opciones de implementación

### Opción A – Conservadora (recomendada inicial)
Protege Nivel 1 con máxima inversión, los demás con protección estándar.

### Opción B – Alta seguridad
Nivel 1 y 2 con protección fuerte y replicación cruzada.

### Opción C – Máxima resiliencia (gobierno + SLA fuerte)
Todos los niveles con políticas estrictas y DR site completo.

---

## 🧠 Recomendación técnica

Para el escenario actual (10–12 clientes, uno gubernamental):

> **Implementar Opción A ahora, preparar arquitectura para Opción B/C.**

Esto minimiza costo inicial sin comprometer el crecimiento.

---

## 🧾 Decisiones que debe tomar el cliente

1. ¿Qué sistemas entran en **Nivel 1**?
2. ¿Qué RTO/RPO acepta para cada nivel?
3. ¿Cuánto riesgo legal está dispuesto a asumir?
4. ¿Se requiere DR site en el primer año o en fase 2?

---

## 🏁 Conclusión ejecutiva

Clasificar datos críticos es el **primer paso para construir una infraestructura seria**.  
Sin esta clasificación, cualquier arquitectura queda incompleta.

---

