# 🛡️ Implementación de SIEM Wazuh para Monitorización de Sistemas Legacy

## 📋 Descripción
Este proyecto demuestra la implementación de un entorno de monitorización de seguridad (SIEM) utilizando **Wazuh** para supervisar activos vulnerables y antiguos (**Metasploitable 2**), donde no es posible instalar agentes modernos debido a limitaciones de compatibilidad del sistema operativo.

## 🏗️ Arquitectura del Laboratorio
Para este entorno se han configurado tres máquinas virtuales en una red aislada:
- **Atacante:** Kali Linux (10.0.0.1)
- **Defensa (SIEM):** Wazuh Manager (10.0.0.4)
- **Víctima (Legacy):** Metasploitable 2 (10.0.0.5)
![Configuración de IP inicial en Wazuh Server](image.png)
---

## 🛠️ Desafío Técnico: Monitorización Sin Agente
El reto principal fue establecer visibilidad sobre un sistema "Legacy" que no soporta el agente de Wazuh actual.
- **Solución:** Configuración de un canal de comunicación vía **Syslog (UDP/514)**.
- **Configuración en Víctima:** Se redirigieron todos los logs del sistema hacia el SIEM mediante la edición del archivo `/etc/rsyslog.conf`.

---

## ⚠️ Análisis de Incidentes y Hallazgos
Durante las pruebas de intrusión con **Nmap**, se identificó un comportamiento crítico:
1. **Saturación del Servicio:** Un escaneo agresivo (`-A`) provocó la saturación momentánea del demonio de registro en la víctima.
2. **Detección de Continuidad:** Wazuh detectó y alertó sobre el reinicio del servicio `syslogd` (Alerta Nivel 5), evidenciando un impacto directo en el sistema monitorizado tras un escaneo de red de alta intensidad.

---

## 🚀 Conclusión
Este laboratorio demuestra mi capacidad para:
- Configurar arquitecturas de red seguras y monitorización centralizada.
- Solventar limitaciones de compatibilidad en sistemas antiguos mediante protocolos estándar (Syslog).
- Analizar logs de sistema para identificar anomalías de seguridad incluso ante la ausencia de firmas específicas de ataque.
