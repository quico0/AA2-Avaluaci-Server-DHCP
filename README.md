# Pràctica: Configuració d’un servidor DHCP amb Kea sobre Ubuntu Server

## Alumne
**Nom:** Quico  
**Curs:** SMX  
**Mòdul:** Xarxes

---

# Objectiu

L’objectiu d’aquesta pràctica és configurar un servidor DHCP amb Kea sobre Ubuntu Server per tal que assigni automàticament adreces IP a un client Zorin Linux. També es comprovarà el seu funcionament mitjançant diverses eines de diagnosi i captures de xarxa amb Wireshark.

---

# 1. Configuració inicial de les màquines

Per realitzar la pràctica s’utilitzen dues màquines virtuals:

## Servidor Ubuntu Server

El servidor disposa de dues interfícies de xarxa:

- Adaptador 1: NAT (accés a Internet)
- Adaptador 2: Xarxa interna

La interfície de xarxa interna s’ha configurat amb:

- IP: `192.169.X.1`
- Màscara: `255.255.255.0`
- Sense porta d’enllaç
- Sense DNS

### Captura 1
*Configuració de les interfícies de xarxa de l’Ubuntu Server.*

![Captura 1](captures/captura1 Client Zorin Linux

El client es configura inicialment amb una interfície en mode Xarxa Interna.

No es configura cap IP manualment perquè serà assignada pel servidor DHCP.

### Captura 2
*Configuració de xarxa de la màquina Zorin Linux.*

![Captes/captura2.png

---

# 2. Instal·lació del servidor DHCP Kea

Primer actualitzem el sistema:

```bash
sudo apt update
sudo apt upgrade -y
```

Instal·lem Kea:

```bash
sudo apt install kea -y
```

Un cop finalitzada la instal·lació, comprovem que el paquet s'ha instal·lat correctament.

```bash
systemctl status kea-dhcp4-server
```

### Captura 3
*Instal·lació correcta del paquet Kea DHCP.*

captures/captura3.png

---

# 3. Configuració de Kea

Editem l'arxiu principal de configuració:

```bash
sudo nano /etc/kea/kea-dhcp4.conf
```

Seguint les indicacions de la pràctica, es desactiva DHCPv6 i DDNS i es configura el servei DHCPv4.

Configuració utilitzada:

```json
{
  "Dhcp4": {

    "interfaces-config": {
      "interfaces": [ "enp0s8" ]
    },

    "lease-database": {
      "type": "memfile"
    },

    "renew-timer": 900,
    "rebind-timer": 1800,
    "valid-lifetime": 3600,

    "subnet4": [
      {
        "subnet": "192.169.X.0/24",

        "pools": [
          {
            "pool": "192.169.X.10 - 192.169.X.50"
          }
        ],

        "option-data": [
          {
            "name": "routers",
            "data": "192.169.X.254"
          },
          {
            "name": "domain-name-servers",
            "data": "8.8.8.8"
          }
        ]
      }
    ]
  }
}
```

On:

- El rang DHCP va de `192.169.X.10` a `192.169.X.50`
- La porta d’enllaç és `192.169.X.254`
- El servidor DNS és `8.8.8.8`

### Captura 4
*Configuració de l'arxiu `/etc/kea/kea-dhcp4.conf`.*

![ptures/captura4.png

---

# 4. Reinici i verificació del servei

Després de modificar la configuració reiniciem el servei:

```bash
sudo systemctl restart kea-dhcp4-server
```

Comprovem el seu funcionament:

```bash
sudo systemctl status kea-dhcp4-server
```

També podem consultar els registres:

```bash
journalctl -u kea-dhcp4-server
```

### Captura 5
*Servei Kea executant-se correctament.*

captures/captura5.png

---

# 5. Instal·lació de Wireshark al client

A la màquina Zorin instal·lem Wireshark:

```bash
sudo apt update
sudo apt install wireshark
```

Per executar-lo:

```bash
sudo wireshark
```

### Captura 6
*Instal·lació i execució de Wireshark.*

![Captes/captura6.png

---

# 6. Obtenció automàtica de la configuració IP

Amb el servidor DHCP actiu, el client Zorin demana una adreça IP automàticament.

Podem comprovar-ho amb:

```bash
ip a
```

L'adreça assignada ha d'estar dins del rang:

```text
192.169.X.10 - 192.169.X.50
```

### Captura 7
*Adreça IP obtinguda automàticament del servidor DHCP.*

![Captes/captura7.png

---

# 7. Comprovació de la ruta per defecte

Executem:

```bash
ip route
```

La sortida ha de mostrar:

```text
default via 192.169.X.254
```

Això indica que la porta d’enllaç s’ha rebut correctament.

### Captura 8
*Comprovació de la porta d’enllaç rebuda via DHCP.*

![Captura 8](captures/captura8.png)

ficació del servidor DNS

Comprovem que el client ha rebut correctament el DNS configurat.

```bash
cat /etc/resolv.conf
```

Ha d'aparèixer:

```text
nameserver 8.8.8.8
```

### Captura 9
*Comprovació del servidor DNS assignat pel DHCP.*

![Captes/captura9.png

---

# 9. Captura del procés DHCP amb Wireshark

A Wireshark iniciem una captura i filtrem els paquets DHCP.

Filtre utilitzat:

```text
dhcp
```

Es poden observar les quatre fases principals del protocol DHCP:

1. DHCP Discover
2. DHCP Offer
3. DHCP Request
4. DHCP ACK

Aquest intercanvi confirma que el servidor DHCP està funcionant correctament.

### Captura 10
*Procés complet DHCP Discover → Offer → Request → ACK capturat amb Wireshark.*

![Captes/captura10.png

---

# Conclusions

En aquesta pràctica s’ha instal·lat i configurat correctament un servidor DHCP mitjançant Kea sobre Ubuntu Server.

El servidor ha assignat automàticament una adreça IP al client Zorin Linux dins del rang configurat, així com la porta d’enllaç i el servidor DNS especificats.

Mitjançant les comprovacions realitzades amb les ordres de xarxa i la captura dels paquets DHCP amb Wireshark s’ha verificat que el servei funciona correctament.

Aquesta pràctica ha servit per comprendre el funcionament del protocol DHCP i la configuració bàsica d’un servidor Kea en un entorn Linux.

---

# Repositori GitHub

Enllaç al repositori:

```text
https://github.com/EL-TEU-USUARI/nom-del-repositori
```

Exemple:

```text
https://github.com/quico-smx/practica-kea-dhcp
```
``