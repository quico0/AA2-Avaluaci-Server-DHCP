# Pràctica: Configuració d’un servidor DHCP amb Kea sobre Ubuntu Server

## Alumne
**Nom:** Quico Carbonell verdura 
**Curs:** SMX 2A
**Mòdul:** Serveis En Xarxa

---

# 1. Configuració inicial de les màquines

Per realitzar la pràctica s’utilitzen dues màquines virtuals:

## Servidor Ubuntu Server

El servidor disposa de dues interfícies de xarxa:

- Adaptador 1: NAT (accés a Internet)
- Adaptador 2: Xarxa interna

Client Zorin
- Adaptador 1: Xarxa interna

*Configuració de les interfícies de xarxa de el servidor i la maquina client.*

![Captura 1](captures/captura1 Client Zorin Linux

![Captura 2](captures/captura1 Client Zorin Linux

![Captura 3](captures/captura1 Client Zorin Linux

El client es configura inicialment amb una interfície en mode NAT a la primera interficie i a la segona Xarxa Interna.

La maquina client de zorin posarem el primer adaptador en xarxa interna.

Ara Aqui Configurarem la ip de la segona interficie de el servidor amb aquesta comanda.

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

*Configuració de xarxa de el servidor.*

![Captura 4](captures/captura1 Client Zorin Linux

La interfície de xarxa interna s’ha configurat amb:

- IP: `192.169.04.1`
- Màscara: `255.255.255.0`
- Sense porta d’enllaç
- Sense DNS

---

# 2. Instal·lació del servidor DHCP Kea

Primer actualitzem el sistema:

```bash
sudo apt update && sudo apt upgrade -y
```

Instal·lem Kea:

```bash
sudo apt install kea -y
```

Ara simplement ens sortira aquesta pantalla i li donem a enter a la opcio per defecte do_nothing.

![Captura 5](captures/captura1 Client Zorin Linux

Un cop finalitzada la instal·lació, comprovem que el paquet s'ha instal·lat correctament.

```bash
systemctl status kea-dhcp4-server.service
```

![Captura 6](captures/captura1 Client Zorin Linux

---

# 3. Configuració de Kea

Editem l'arxiu principal de configuració:

```bash
sudo nano /etc/kea/kea-dhcp4.conf
```

Seguint les indicacions de la pràctica, es desactiva DHCPv6 i DDNS i es configura el servei DHCPv4.



- El rang DHCP va de `192.169.04.10` a `192.169.04.50`
- La porta d’enllaç és `192.169.04.254`
- El servidor DNS és `8.8.8.8`

*Configuració de l'arxiu `/etc/kea/kea-dhcp4.conf`.*

![Captura 7](captures/captura1 Client Zorin Linux

---

# 4. Reinici i verificació del servei

Després de modificar la configuració reiniciem el servei:

```bash
sudo systemctl restart kea-dhcp4-server.service
```

Comprovem el seu funcionament:

```bash
sudo systemctl status kea-dhcp4-server
```

També podem consultar els registres:

```bash
journalctl -u kea-dhcp4-server
```
Servei Kea executant-se correctament.*

![Captura 8](captures/captura1 Client Zorin Linux

---

# 5. Instal·lació de Wireshark al client

A la màquina Zorin instal·lem Wireshark:

```bash
sudo apt update
sudo apt install wireshark
```

Per fer l'instalacio s'ha canviat tamporalment de xarxa interna a NAT per tenir acces a internet despres de la instalacio el tornem a posar en xarxa interna.

Per executar-lo:

```bash
sudo wireshark
```

Per instal·lar et sortitra una pantalla i hem de seleccionar que si. 

Instal·lació i execució de Wireshark.

![Captura 9](captures/captura1 Client Zorin Linux

![Captura 10](captures/captura1 Client Zorin Linux

---

# 6. Obtenció automàtica de la configuració IP

Amb el servidor DHCP actiu, el client Zorin demana una adreça IP automàticament.

Podem comprovar-ho amb:

```bash
ip a
```

L'adreça assignada ha d'estar dins del rang:

```text
192.169.4.10 - 192.169.4.50
```

Adreça IP obtinguda automàticament del servidor DHCP.

![Captura 11](captures/captura1 Client Zorin Linux

---



# 7. Comprovació de la ruta per defecte

Executem:

```bash
ip route
```

Això indica que la porta d’enllaç s’ha rebut correctament.

Comprovació de la porta d’enllaç rebuda via DHCP.

![Captura 12](captures/captura1 Client Zorin Linux

Comprovem que el client ha rebut correctament el DNS configurat.

```bash
resolvectl status
`````

Ha d'aparèixer:

```text
Current DNS Server 8.8.8.8
```

Comprovació del servidor DNS assignat pel DHCP.

![Captura 13(captures/captura1 Client Zorin Linux

---

# 9. Captura del procés DHCP amb Wireshark

A Wireshark iniciem una captura i filtrem els paquets DHCP.

Es poden observar les quatre fases principals del protocol DHCP:

1. DHCP Discover
2. DHCP Offer
3. DHCP Request
4. DHCP ACK

Aquest intercanvi confirma que el servidor DHCP està funcionant correctament.

![Captura 14(captures/captura1 Client Zorin Linux
