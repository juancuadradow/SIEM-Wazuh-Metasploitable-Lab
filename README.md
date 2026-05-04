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

### 🛠️ Configuración en el Wazuh Manager (ossec.conf)

Para que el servidor acepte los logs remotos, fue necesario habilitar el recolector de **Syslog** dentro del archivo de configuración principal del Manager.

#### 1. Identificación del protocolo remoto
![Configuración nativa de Wazuh](image8.png)
> **Análisis:** Por defecto, el bloque `<remote>` está configurado para conexiones seguras de agentes en el puerto `1514/TCP`. Dado que los sistemas legacy no soportan el agente moderno, debemos habilitar un canal estándar.

#### 2. Habilitación del puerto de escucha Syslog (UDP/514)
![Configuración de recepción Syslog](image9.png)
> **Implementación:** Se añadió un nuevo bloque de configuración especificando:
> * **Connection:** `syslog` (permite la recepción de logs sin necesidad de agente).
> * **Port:** `514` (el puerto estándar para tráfico de logs).
> * **Protocol:** `udp` (elegido por su baja sobrecarga en sistemas antiguos).
> * **Allowed-ips:** `0.0.0.0/0` (en este entorno de laboratorio, permite recibir tráfico de toda la subred configurada).

El reto principal fue establecer visibilidad sobre un sistema "Legacy" que no soporta el agente de Wazuh actual.
- **Solución:** Configuración de un canal de comunicación vía **Syslog (UDP/514)**.
- **Configuración en Víctima:** Se redirigieron todos los logs del sistema hacia el SIEM mediante la edición del archivo `/etc/rsyslog.conf`.

## 🔍 Caso de Estudio: Autoprotección del SIEM (HIDS)

### 1.1 El Incidente (Escaneo Nmap)
Durante las pruebas de estrés, se realizó un escaneo agresivo sobre la IP `10.0.0.4`.

![Escaneo Nmap](image10.png)

### 1.2 Detección de Anomalía
El servidor detectó un pico inusual de actividad en los logs, demostrando su capacidad como **Host-based Intrusion Detection System (HIDS)** incluso cuando el objetivo es él mismo.

![Gráfica de eventos](image11.png)

Agente: wazuh-server (ID: 000). Esto indica que la alerta se generó internamente en el servidor.

Nivel de Regla (Rule Level): En la captura se observa un Nivel 3, pero durante un Nmap agresivo, este nivel escala según los scripts NSE que se activen.

Descripción de la regla: En el log detallado vemos PAM: Login session opened y actividades de sudo. Esto es porque Nmap, al intentar conectar con scripts agresivos, dispara eventos de autenticación y acceso al sistema.

## 🎯 Ataque Dirigido a Activo Externo (Metasploitable 2)

Tras validar la autoprotección del SIEM, se procede a lanzar el ataque al objetivo real de la infraestructura (10.0.0.5).

### 1.1 Ejecución y Troubleshooting de Visibilidad
Al lanzar un escaneo agresivo inicial con Nmap, se detectó un **"Detection Gap"**: Wazuh no registraba los eventos de la víctima en el Dashboard.

![Ataque Nmap a Victima](image12.png)

*   **Problema identificado:** El "chivato" (reenvío de logs) no estaba enviando datos de forma consistente.
*   **Solución aplicada:** Se forzó la configuración del canal mediante:

     `echo "*.* @10.0.0.4" | sudo tee -a /etc/syslog.conf`
![Comando echo](image13.png)
Comprobamos que se a plasmado en la maquina
![Comprobacion](image14.png)
Correcto
![Comprobacion](image15.png)

*   **Persistencia:** Se verificó la correcta escritura del archivo y se reinició el servicio para asegurar el flujo:
    `sudo /etc/init.d/sysklogd restart`
![Comando sudo reinicio de servicio](image16.png)

Una vez configurado el reenvío de logs en la víctima (10.0.0.5), se procede a realizar un escaneo agresivo con Nmap desde la máquina atacante Kali Linux

![Comando sudo cat](image17.png)

---

### 1.2 Monitorización en Tiempo Real (CLI)
Para confirmar la comunicación sin depender de la latencia de la interfaz gráfica, se utilizó la línea de comandos en el servidor Wazuh.

![Monitorización en tiempo real](image18.png)

> **Evidencia:** Mediante `tail -f` sobre los logs de alertas, se confirmó visualmente que los registros de actividad de la víctima (10.0.0.5) ya estaban siendo procesados por el motor de análisis del SIEM.

---

### 🔬 Análisis Final de Blue Team: ¿Por qué no hubo alertas rojas?

A pesar de confirmar el flujo de datos, los ataques de Nmap no dispararon alertas de criticidad alta en el Dashboard. Este es un hallazgo clave del proyecto:

1. **Saturación del Nodo:** Un escaneo `-A -T4` es tan agresivo que satura el demonio de logs antiguo (`sysklogd`), provocando que el servicio se detenga antes de poder reportar el ataque completo.

![Evidencia de saturacion 10.0.0.5](image19.png)

> **Hallazgo Clave:** El log detallado muestra la descripción `syslogd exiting (logging stopped)`. Esto confirma que el servicio de auditoría de la víctima se detuvo antes de poder reportar el ataque completo, generando un **"punto ciego"** por denegación de servicio en el propio registro de eventos.

2. **Ajuste de Reglas (Tuning):** Se concluye que para sistemas legacy, el Blue Team debe crear reglas personalizadas que detecten picos de conexiones en puertos cerrados, ya que las reglas genéricas pueden no alcanzar el umbral de severidad necesario por defecto ante una caída del demonio.

> **Éxito Técnico:** Se ha logrado establecer un canal de telemetría funcional en un entorno de red real, superando las barreras de compatibilidad de sistemas obsoletos.

---

## ⚠️ Análisis de Incidentes y Hallazgos

### 📊 Dashboard de Seguridad: Visualización de Eventos

![Dashboard de Wazuh con alertas](image5.png)

> **Evidencia técnica:** En el panel superior se observa que, aunque no hay agentes instalados (0 agents registered), el SIEM ya ha procesado **12 alertas de severidad media** y **106 de severidad baja**. Esto confirma que la ingesta de datos vía Syslog desde el sistema Legacy es exitosa y permite iniciar el análisis de comportamiento.

---

## 🚀 Conclusión

Este laboratorio demuestra mi capacidad para:
* **Configurar arquitecturas de red seguras** y monitorización centralizada.
* **Solventar limitaciones de compatibilidad** en sistemas antiguos mediante protocolos estándar (Syslog).
* **Analizar logs de sistema** para identificar anomalías de seguridad e identificar comportamientos críticos como la saturación de servicios de auditoría.
