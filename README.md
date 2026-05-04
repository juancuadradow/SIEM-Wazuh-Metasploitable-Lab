# 🛡️ Implementación de SIEM Wazuh para Monitorización de Sistemas Legacy

## 📋 Descripción
Este proyecto demuestra la implementación de un entorno de monitorización de seguridad (SIEM) utilizando **Wazuh** para supervisar activos vulnerables y antiguos (**Metasploitable 2**), donde no es posible instalar agentes modernos debido a limitaciones de compatibilidad del sistema operativo.

## 🏗️ Arquitectura del Laboratorio
Para este entorno se han configurado tres máquinas virtuales en una red aislada:
- **Atacante:** Kali Linux (10.0.0.1)
- **Defensa (SIEM):** Wazuh Manager (10.0.0.4)
- **Víctima (Legacy):** Metasploitable 2 (10.0.0.5)
  
#### ⚙️ Configuración de IP inicial en Wazuh Server  
![Configuración de IP inicial en Wazuh Server](image.png)

#### ✅ Verificación de IP 10.0.0.4 en Wazuh Server
![Verificación de IP 10.0.0.4 en Wazuh Server](image2.png)

#### 🖥️ Verificación del estado de Wazuh Dashboard
![Estado de Wazuh Dashboard](image3.png)
> **Nota técnica:** En la captura superior se observa el flujo de logs de OpenSearch en formato JSON. Estos registros confirman la correcta sincronización entre el indexador y la interfaz visual, garantizando que las alertas generadas por Metasploitable 2 se visualicen sin retraso (latencia mínima).

## 🛠️ Desafío Técnico: Monitorización Sin Agente

#### 🌐 Acceso a la Interfaz Web de Wazuh
![Login de Wazuh Dashboard](image4.png)

#### ⚙️ Configuración del reenvío de logs en Metasploitable 2
![Comando de edición](image6.png)

![Configuración Syslog](image7.png)

> **Explicación:** Se utilizó el editor `nano` para añadir la instrucción `*.* @10.0.0.4:514`. Esto indica al sistema legacy que envíe **todos** los logs (`*.*`) al servidor Wazuh (`10.0.0.4`) a través del puerto estándar `514`.

> **Verificación de conectividad:** Se confirma el acceso exitoso al panel de control desde la máquina atacante/analista (Kali Linux) apuntando a la IP `10.0.0.4` a través de HTTPS.

El reto principal fue establecer visibilidad sobre un sistema "Legacy" que no soporta el agente de Wazuh actual.
- **Solución:** Configuración de un canal de comunicación vía **Syslog (UDP/514)**.
- **Configuración en Víctima:** Se redirigieron todos los logs del sistema hacia el SIEM mediante la edición del archivo `/etc/rsyslog.conf`.

---

## ⚠️ Análisis de Incidentes y Hallazgos

#### 📊 Dashboard de Seguridad: Visualización de Eventos

![Dashboard de Wazuh con alertas](image5.png)

> **Evidencia técnica:** En el panel superior se observa que, aunque no hay agentes instalados (0 agents registered), el SIEM ya ha procesado **12 alertas de severidad media** y **106 de severidad baja**. Esto confirma que la ingesta de datos vía Syslog desde el sistema Legacy es exitosa y permite iniciar el análisis de comportamiento.
> 
Durante las pruebas de intrusión con **Nmap**, se identificó un comportamiento crítico:
1. **Saturación del Servicio:** Un escaneo agresivo (`-A`) provocó la saturación momentánea del demonio de registro en la víctima.
2. **Detección de Continuidad:** Wazuh detectó y alertó sobre el reinicio del servicio `syslogd` (Alerta Nivel 5), evidenciando un impacto directo en el sistema monitorizado tras un escaneo de red de alta intensidad.

---

## 🚀 Conclusión
Este laboratorio demuestra mi capacidad para:
- Configurar arquitecturas de red seguras y monitorización centralizada.
- Solventar limitaciones de compatibilidad en sistemas antiguos mediante protocolos estándar (Syslog).
- Analizar logs de sistema para identificar anomalías de seguridad incluso ante la ausencia de firmas específicas de ataque.
