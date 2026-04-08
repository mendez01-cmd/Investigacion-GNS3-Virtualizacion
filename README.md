# Investigación: Implementación de Laboratorios Avanzados con GNS3 y Hipervisores

![Estado](https://img.shields.io/badge/Estado-Completado-green) ![Plataforma](https://img.shields.io/badge/Plataforma-GNS3%20%2B%20Windows11-blue)

Este repositorio contiene la investigación técnica sobre la integración de GNS3 con hipervisores de Tipo 1 (ESXi) y Tipo 2 (VirtualBox) sobre un entorno de Windows 11.

---

## 1. Arquitectura de Virtualización en Windows 11

### Aislamiento de Núcleo y VBS
Windows 11 utiliza **VBS (Virtualization-Based Security)** para crear una región de memoria aislada del sistema operativo. 
* **Impacto:** Esta función utiliza Hyper-V de forma silenciosa. Si está activa, Hyper-V "secuestra" las extensiones de hardware (VT-x/AMD-V), impidiendo que la GNS3 VM pueda activar la aceleración KVM, lo que resulta en un rendimiento extremadamente pobre.

### Activación de VT-x/AMD-V
Para habilitar el soporte de hardware:
1. Se debe acceder a la **BIOS/UEFI** del equipo y activar "Intel Virtualization Technology" o "SVM Mode".
2. En Windows 11, se verifica en el **Administrador de Tareas > Rendimiento > CPU**, donde debe figurar "Virtualización: Habilitado".

![Verificación de Virtualización](img/verificacion-cpu.png)

---

## 2. GNS3 VM: El Motor de Simulación

### KVM (Kernel-based Virtual Machine)
KVM es el módulo de virtualización para el kernel de Linux que permite a la GNS3 VM actuar como un hipervisor nativo.
* **Por qué es obligatorio:** Sin KVM (estado "True"), los nodos de red como routers Cisco IOSv o ASAv emulan la CPU por software. Con KVM "True", los procesos corren directamente sobre el hardware del host, permitiendo laboratorios fluidos y profesionales.

### Configuración de Recursos
Para no desestabilizar Windows 11, se recomienda:
* **CPU:** Asignar 2 vCPUs (o el 50% de los hilos lógicos).
* **RAM:** Al menos 4GB, asegurándose de dejar 4GB libres para el sistema host para evitar el uso de memoria virtual (swap) en disco.

---

## 3. Integración con VirtualBox (Local)

### Configuración de Red
Se utiliza el adaptador **Host-Only** (Solo-Anfitrión) para establecer un canal de comunicación privado entre la GUI de GNS3 y la VM. Esto evita conflictos con otras redes físicas y asegura que la IP de la VM sea constante.

### Modo Promiscuo
* **Análisis Técnico:** Es necesario cambiar el modo promiscuo a **"Permitir todo"**. Las redes de GNS3 operan en Capa 2; sin este modo, VirtualBox descartaría cualquier trama Ethernet que no tenga como destino la dirección MAC propia de la VM, rompiendo la comunicación entre los routers virtuales.

---

## 4. Integración con VMware ESXi (Remoto)

### Arquitectura Cliente-Servidor
En esta modalidad, la Laptop actúa como cliente (GUI) y el servidor ESXi físico actúa como el motor de ejecución. Se conectan mediante la IP del servidor ESXi a través del puerto **3080**.

![Diagrama de Arquitectura](img/diagrama-red.png)

### Seguridad en vSwitch
En el **Port Group** de ESXi donde reside la GNS3 VM, se deben cambiar las políticas de seguridad a **Accept**:
* **Promiscuous mode:** Permite que los nodos vean tráfico que no les pertenece.
* **MAC address changes:** Permite que los routers virtuales cambien su MAC para protocolos de redundancia.
* **Forged transmits:** Permite el envío de tráfico con MACs distintas a la del adaptador principal.

---

## 5. Matriz de Solución de Errores (Troubleshooting)

| Error Detectado | Causa Técnica | Solución Implementada |
| :--- | :--- | :--- |
| **KVM support: False** | Conflicto con Hyper-V / Core Isolation en Windows 11. | Desactivar "Integridad de Memoria" en Seguridad de Windows y reiniciar. |
| **uBridge permissions** | El servicio uBridge requiere privilegios elevados para interceptar tráfico. | Ejecutar GNS3 como Administrador o reinstalar Npcap. |
| **Connection Timeout (3080)** | El Firewall de Windows o del Servidor bloquea el puerto de la API. | Crear una regla de entrada para permitir tráfico TCP en el puerto 3080. |

---

## Recursos Adicionales
* [Documentación Oficial de GNS3](https://docs.gns3.com/)
* [Guía de Seguridad en vSwitch de VMware](https://docs.vmware.com/)
