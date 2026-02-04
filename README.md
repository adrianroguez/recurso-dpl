# 🛡️ GUÍA MAESTRA: EXAMEN DPL (DNS + HTTPS)

**Estado:** Exam Ready  
**Advertencia:** Examen "All In" (Sin recuperación).  
**Objetivo:** Configurar Bind9 (Forwarder) y Nginx (HTTPS) + Diagnóstico.

---

## 0. TABLA DE REFERENCIA RÁPIDA

### 📌 Puertos y Protocolos
| Servicio | Puerto | Protocolo | Notas |
| :--- | :--- | :--- | :--- |
| **DNS** | **53** | **UDP** | TCP se usa solo si la respuesta es > 512 bytes. |
| **HTTP** | 80 | TCP | Web insegura. |
| **HTTPS** | **443** | TCP | Web segura (SSL/TLS). Encriptación + Hash + Identidad. |

### 📂 Rutas de Archivos Críticos
| Archivo / Directorio | Función | Comando |
| :--- | :--- | :--- |
| **DNS Config** | Configuración de Forwarders | `sudo nano /etc/bind/named.conf.options` |
| **Web Config** | Configuración VirtualHost | `sudo nano /etc/nginx/sites-available/default` |
| **Hosts** | DNS Local (Manual) | `sudo nano /etc/hosts` |
| **Certs (.crt)** | Certificados Públicos | `/etc/ssl/certs/` |
| **Keys (.key)** | Claves Privadas | `/etc/ssl/private/` |

---

## 1. PREPARACIÓN (Antes de empezar)

> **⚠️ IMPORTANTE:** Haz esto nada más sentarte.

1.  **Snapshot:** Crea una instantánea de la máquina virtual llamada "Limpia".
2.  **Backups de seguridad:**
    ```bash
    sudo cp /etc/bind/named.conf.options /etc/bind/named.conf.options.bak
    sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/default.bak
    ```
3.  **Configurar IP (Solo si se pide manual):**
    ```bash
    # Ejemplo: Añadir IP 172.17.0.7 a la interfaz enp0s3
    sudo ip addr add 172.17.0.7/24 dev enp0s3
    ```

---

## 2. DNS: BIND9 (Teoría y Práctica)

### ⚙️ Configuración: El "Forwarder"
**Objetivo:** Reenviar todo al router del profesor. NO usar raíces.

**Archivo:** `/etc/bind/named.conf.options`

```c
options {
    directory "/var/cache/bind";

    // 1. Permitir recursividad (tu server trabaja para otros)
    recursion yes;
    allow-query { any; };

    // 2. A quién preguntar (IP DEL PROFESOR / ROUTER)
    // ¡CAMBIAR LA X POR LA IP REAL DEL EXAMEN!
    forwarders {
        192.168.X.X;
    };

    // 3. OBLIGATORIO: Si el profe falla, tú fallas (no buscar fuera)
    forward only;

    // 4. Seguridad (OFF para evitar líos en laboratorio)
    dnssec-validation no;

    listen-on-v6 { any; };
};