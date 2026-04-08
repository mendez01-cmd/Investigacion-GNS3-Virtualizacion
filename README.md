## 1. Arquitectura de Virtualización en Windows 11

### Aislamiento de Núcleo y VBS
Windows 11 introduce la Seguridad basada en Virtualización (VBS). Esta función utiliza el hipervisor nativo Hyper-V para crear una región de memoria aislada.
* **Impacto técnico:** Al estar activo, Hyper-V toma el control exclusivo de las extensiones de hardware (VT-x/AMD-V). Esto impide que otros hipervisores de Tipo 2 (como VirtualBox) accedan a la aceleración de hardware, provocando que la GNS3 VM no pueda ejecutar KVM.

### Activación de VT-x/AMD-V
1. **BIOS/UEFI:** Es el primer paso obligatorio para habilitar el soporte de virtualización a nivel de silicio.
2. **Verificación en OS:** Se confirma mediante el Administrador de Tareas o ejecutando `systeminfo` en el CMD. Si aparece como "Habilitado", el sistema operativo ya puede delegar funciones de virtualización.

---

## 2. GNS3 VM: El Motor de Simulación

### KVM (Kernel-based Virtual Machine)
KVM es una tecnología de virtualización de código abierto integrada en Linux.
* **Por qué es obligatorio:** Permite que la GNS3 VM actúe como un hipervisor de Tipo 1 para los nodos internos. Sin "KVM: True", los dispositivos QEMU/KVM (como routers Cisco modernos) funcionan por emulación de software, lo cual es ineficiente y extremadamente lento.

### Configuración de Recursos
Para un rendimiento profesional en Windows 11:
* **CPU:** Se recomienda asignar **2 vCPUs** como mínimo.
* **RAM:** **4 GB** es el estándar base. Es crítico no asignar más del 50% de la RAM física para evitar el "swapping" en Windows 11, lo cual congelaría la interfaz de GNS3.

---

## 3. Integración con VirtualBox (Local)

### Configuración de Red
Se utiliza el adaptador **Host-Only (Sólo-Anfitrión)**. Este crea una red virtual interna entre el host (Windows) y la VM, permitiendo que la GUI de GNS3 envíe instrucciones a la API del servidor sin depender de una conexión física de red.

### Modo Promiscuo
* **Explicación Técnica:** Las redes de datos reales utilizan tramas con diversas direcciones MAC. Por defecto, VirtualBox descarta cualquier trama que no vaya dirigida a la MAC de la VM. Al activar el **Modo Promiscuo (Allow All)**, el puente virtual permite el paso de todo el tráfico de Capa 2, permitiendo que protocolos como STP y ARP funcionen correctamente en el laboratorio.

---

## 4. Integración con VMware ESXi (Remoto)

### Arquitectura Cliente-Servidor
A diferencia de la instalación local, aquí la carga de cómputo se desplaza a un servidor dedicado. La laptop solo corre el "Front-end", conectándose vía TCP/IP al servidor ESXi que aloja las instancias de los nodos.

### Seguridad en vSwitch
En el Port Group de ESXi, es imperativo cambiar las políticas de seguridad a **Accept**:
1. **Modo Promiscuo:** Para el análisis de tráfico.
2. **Cambios de dirección MAC:** Para que los routers virtuales puedan generar sus propias direcciones.
3. **Transmisiones falsificadas:** Para permitir que las tramas salgan con MACs distintas a la principal del adaptador.

---

## 5. Matriz de Solución de Errores (Troubleshooting)

| Error Detectado | Causa Técnica | Solución Implementada |
| :--- | :--- | :--- |
| **GNS3 VM: KVM False** | Conflicto con "Integridad de Memoria" de Windows 11. | Desactivar Aislamiento de Núcleo y reiniciar el sistema. |
| **uBridge permissions** | Falta de privilegios para crear interfaces de red virtuales. | Ejecutar GNS3 como Administrador o reinstalar Npcap. |
| **Connection Timeout (3080)** | Firewall de Windows bloqueando el tráfico hacia la VM. | Crear regla de entrada para el puerto TCP 3080 en el Firewall. |
