# Proyecto Ansible - Compania XYZ (Redes V)

## Scripts (playbooks)

| Archivo | Que hace |
| --- | --- |
| `01_configuracion_base.yml` | Requisito 1: hostname, SSH v2, usuarios, enable secret, line vty, access-class (solo Guatemala), banners LOGIN y MOTD, NTP, SNMPv3 hacia Zabbix y logging |
| `02_red.yml` | Interconexion: VLANs, loopbacks, OSPF/EIGRP, eBGP con filtro BGP_OUT y redistribucion de BGP en el IGP |
| `03_zabbix.yml` | Instala Zabbix 7.0 LTS y da de alta todos los equipos por SNMPv3 |
| `04_respaldo.yml` | Requisito 2: guarda el running-config en `respaldos/<equipo>/<Mes>/<dia>/<equipo>.<MMDDAAAA>.<HH:MM>.cfg` |
| `05_cierre_sede.yml` | Requisito 3: cierre y reapertura de una sede |
| `site.yml` | Ejecuta 01, 02, 03 y 04 en un solo comando |

## Estructura

```
ansible.cfg
inventario/
  hosts.yml            equipos y grupos
  group_vars/          variables globales, de conexion y por sede
  host_vars/           interfaces, IP y BGP de cada equipo
plantillas/            red.j2, base.j2 y zabbix.conf.php.j2
tareas/                tareas reutilizadas por los playbooks
bootstrap/             configuracion minima (Day-0) de cada equipo
```

## Ejecucion

Requisitos en el servidor: Ansible con las colecciones `cisco.ios`,
`ansible.netcommon`, `community.zabbix`, `community.mysql` y `community.general`.

```bash
ansible-playbook 01_configuracion_base.yml
ansible-playbook 02_red.yml
ansible-playbook 03_zabbix.yml -K
ansible-playbook 04_respaldo.yml
ansible-playbook 05_cierre_sede.yml -e "sede_cierre=honduras accion_cierre=cerrar"
ansible-playbook 05_cierre_sede.yml -e "sede_cierre=honduras accion_cierre=abrir"
```

O todo junto (excepto el cierre de sedes):

```bash
ansible-playbook site.yml -K
```

## Day-0

Ansible se conecta a los equipos por SSH, asi que cada equipo necesita una
configuracion minima previa (carpeta `bootstrap/`): IP de gestion, hostname y
dominio (requeridos para generar las llaves RSA), usuario `ansible`, llaves RSA,
SSH version 2 y vty por SSH. Todo lo demas lo configura Ansible.

> Las claves de este repositorio son de un laboratorio aislado.
