# Catálogo de Automatizaciones - Bizagi Pre-alistamiento

## 📋 Descripción

Sistema de automatización modular para pre-alistamiento de Bizagi usando Ansible/AWX. Permite validar conectividad de red (telnet/ping) de forma parametrizable y reutilizable desde hosts pivote hacia múltiples destinos.

## 🏗️ Arquitectura

```
catalogo_automatizaciones/
├── ansible.cfg                    # Configuración de Ansible
├── inventory/                     # Inventarios por ambiente
│   ├── linux/hosts               # Hosts Linux pivote
│   ├── windows/hosts             # Hosts Windows pivote
│   └── servers/hosts             # Consolidado de servidores
├── roles/                        # Roles reutilizables
│   ├── test_connection/          # Módulo de conectividad
│   └── file_comparison/          # Módulo de archivos
├── logs/                         # Logs de ejecución
├── test_connection.yml           # Playbook principal conectividad
└── README.md                     # Este archivo
```

## 🚀 Características

### ✅ **Conectividad Multi-protocolo**

- **Telnet**: Validación de puertos TCP específicos
- **Ping**: Validación ICMP básica
- **Ambos**: Ejecución combinada telnet + ping

### ✅ **Multi-plataforma**

- **Linux**: RHEL/CentOS/Debian usando `/dev/tcp` y `ping`
- **Windows**: PowerShell con `Test-NetConnection` y `Test-Connection`

### ✅ **Parametrización Dinámica**

- Hosts pivote seleccionables por grupo
- Targets múltiples por entrada de texto
- IDs de tarea personalizables
- Tipos de test configurables

### ✅ **Integración AWX/AAP**

- Estadísticas estructuradas (`set_stats`)
- Job Templates reutilizables
- Variables de survey configurables
- Logs centralizados

## 🔧 Instalación y Configuración

### Pre-requisitos

```bash
# Instalar Ansible
pip install ansible
pip install pywinrm  # Para Windows targets

# Verificar versión
ansible --version
```

### Configuración de Inventario

1. **Configurar hosts pivote en `inventory/`**:

   ```ini
   [windows_pivote]
   win_prod ansible_host=192.168.20.176

   [linux_pivote]
   linux_prod ansible_host=192.168.20.50
   ```

2. **Configurar credenciales**:

   ```ini
   [windows_pivote:vars]
   ansible_user=ansible
   ansible_password=ansible123
   ansible_connection=winrm

   [linux_pivote:vars]
   ansible_user=ansible
   ansible_password=ansible123
   ansible_connection=ssh
   ```

## 🎯 Uso

### Ejecución Línea de Comandos

```bash
# Validación telnet básica
ansible-playbook test_connection.yml \
  -e execution_hosts=linux_pivote \
  -e task_targets_input="90.5.0.16" \
  -e task_ports_input="9130" \
  -e test_connection_check_type="telnet" \
  -e task_id_input="bizagi_task_01"

# Validación ping múltiple
ansible-playbook test_connection.yml \
  -e execution_hosts=windows_pivote \
  -e task_targets_input="90.5.3.1.134
90.5.3.1.135
10.8.4.23" \
  -e test_connection_check_type="ping" \
  -e task_id_input="okta_connectivity"

# Validación combinada
ansible-playbook test_connection.yml \
  -e execution_hosts=linux_pivote \
  -e task_targets_input="90.5.4.230
10.8.32.10" \
  -e task_ports_input="80" \
  -e test_connection_check_type="ambos" \
  -e task_id_input="datapower_full"
```

### Configuración AWX Job Template

#### Variables de Survey

| Variable                     | Tipo            | Descripción                 | Ejemplo                          |
| ---------------------------- | --------------- | --------------------------- | -------------------------------- |
| `execution_hosts`            | Multiple Choice | Host pivote                 | `linux_pivote`, `windows_pivote` |
| `task_targets_input`         | Textarea        | IPs destino (una por línea) | `90.5.0.16`<br>`10.8.32.10`      |
| `task_ports_input`           | Text            | Puerto destino              | `9130`                           |
| `test_connection_check_type` | Multiple Choice | Tipo de test                | `telnet`, `ping`, `ambos`        |
| `task_id_input`              | Text            | ID de tarea                 | `bizagi_task_01`                 |

#### Configuración del Job Template

```yaml
Name: Test Connectivity
Inventory: Production Servers
Project: Catalogo Automation
Playbook: test_connection.yml
Survey Enabled: ✅
Prompt on Launch: Variables
```

## 📈 Salidas y Métricas

### Salida en Pantalla

```
====================================
TAREA bizagi_task_01: DataPower connectivity
====================================
Desde: linux_prod
Tipo: telnet
Destinos: 90.5.0.16
Puertos: 9130
====================================

TELNET 90.5.0.16:9130 = SUCCESS

RESUMEN: 1/1 exitosos (100.00%)
```

### Stats para AWX

```json
{
  "task_bizagi_task_01": {
    "task_info": {
      "task_id": "bizagi_task_01",
      "description": "DataPower connectivity",
      "target": "90.5.0.16:9130",
      "execution_time": "2025-07-18T10:30:00Z"
    },
    "hosts": {
      "linux_prod": {
        "status": "success",
        "timestamp": "2025-07-18T10:30:00Z",
        "details": ["TELNET 90.5.0.16:9130 = SUCCESS"]
      }
    }
  }
}
```

## 🔍 Troubleshooting

### Errores Comunes

#### "ERROR: Debes definir el grupo de hosts"

```bash
# Solución: Especificar execution_hosts
-e execution_hosts=linux_pivote
```

#### "ERROR: Las IPs descritas tienen un formato válido"

```bash
# Solución: Usar formato IPv4 válido
-e task_targets_input="192.168.1.1"  # ✅ Correcto
-e task_targets_input="invalid-host"  # ❌ Incorrecto
```

#### "SYSTEM_ERROR" en resultados

- Verificar conectividad WinRM/SSH al host pivote
- Validar credenciales en inventario

### Debug Mode

```bash
# Ejecutar con debug detallado
ansible-playbook test_connection.yml -vvv \
  -e execution_hosts=linux_pivote \
  -e task_targets_input="90.5.0.16" \
  -e test_connection_check_type="telnet"
```

## 🤝 Contribución

### Estructura de Desarrollo

1. **Nuevos tipos de test**: Expandir templates en `roles/test_connection/templates/`
2. **Nuevas plataformas**: Crear tasks en `roles/test_connection/tasks/`
3. **Validaciones**: Actualizar validaciones en playbook principal

### Coding Standards

- Usar `snake_case` para variables
- Prefijo `test_connection_` para variables del rol
- Documentar cambios en README del rol
- Mantener compatibilidad con AWX

## 📚 Enlaces Relacionados

- [Documentación del Rol test_connection](roles/test_connection/README.md)

## 📜 Licencia

Este proyecto es para uso interno de automatización.

---

**Autor**: cgarzont (NTTDATA)  
**Versión**: 1.0.0  
**Última actualización**: Julio 2025  
**Mantenedor**: Equipo Automatizacion NTTDATA
