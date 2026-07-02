# 🛡️ Policy-Based VPN (Crypto Maps) con IPsec e IKEv1 en Cisco

## 👤 Perfil del Autor
- **Ingeniero/Desarrollador:** Edgardy Olivero Flores
- **Especialidad:** Seguridad Informática y Redes

## 🗺️ Arquitectura de Red
![Diagrama de Topología](Topologia.png)

## ⚙️ Resumen Técnico
Este repositorio documenta la implementación de una **VPN Site-to-Site basada en políticas (Policy-Based VPN)** utilizando **Crypto Maps** tradicionales sobre la plataforma Cisco vIOS.

En este modelo arquitectónico, no se utilizan interfaces de túnel lógicas. En su lugar, el túnel se establece dinámicamente mediante la interceptación del tráfico en la interfaz WAN física. Se emplean Listas de Control de Acceso (ACLs) para clasificar y definir estrictamente el "tráfico interesante"; cuando un paquete coincide con las reglas de la ACL, el router detona la negociación IKE/IPsec, lo encripta y lo envía hacia el peer remoto. 

### 🔐 Diseño Criptográfico y Clasificación de Tráfico
La protección de los datos se estructura mediante la suite IKEv1 y políticas de clasificación:
* **Fase 1 (ISAKMP / IKEv1):** * Cifrado: AES-256
  * Integridad: SHA-256
  * Grupo de Intercambio: Diffie-Hellman 14
  * Autenticación: Pre-Shared Key (PSK) vinculada a la IP pública del destino.
* **Fase 2 (IPsec en Modo Túnel):** * Protocolo ESP con encriptación AES-256 y autenticación HMAC-SHA-256. Al operar en `mode tunnel`, IPsec genera una nueva cabecera IP exterior para proteger las identidades originales del tráfico LAN-to-LAN.
* **Tráfico Interesante (ACLs):** * Se configuraron Access-Lists extendidas que permiten y envían a cifrar exclusivamente el tráfico originado en la subred local hacia la subred remota autorizada, descartando el resto.

## 🛠️ Flujo de Despliegue
La configuración sigue la metodología clásica de Crypto Maps en Cisco CLI:
1. **Enrutamiento Perimetral:** Asignación de interfaces y ruteo estático hacia la red de tránsito (ISP).
2. **Definición ISAKMP (Fase 1):** Creación de la política de intercambio y definición de la llave pre-compartida (Keyring).
3. **Parámetros IPsec (Fase 2):** Creación del `transform-set` dictando los algoritmos de encriptación del *payload*.
4. **Clasificación de Tráfico:** Creación de la ACL extendida (ej. `access-list 100 permit ip...`).
5. **Ensamblaje del Crypto Map:** Integración de la dirección del peer, el *transform-set* y la ACL dentro de un mapa criptográfico unificado.
6. **Aplicación Física:** Asignación del Crypto Map a la interfaz WAN (frente a internet) para comenzar la inspección de paquetes.

## 🔍 Auditoría y Diagnóstico
Comandos esenciales en la terminal de Cisco para verificar la inyección de tráfico y el estado de los Crypto Maps:

```ios
! Verificar el estado de la negociación Fase 1 (Debe mostrar QM_IDLE)
show crypto isakmp sa

! Confirmar el tráfico cifrado/descifrado y coincidencia con la ACL (Fase 2)
show crypto ipsec sa

! Auditar la configuración actual del Crypto Map aplicado en la interfaz
show crypto map

! Verificar aciertos (hits) en la lista de control de acceso del tráfico interesante
show access-lists
