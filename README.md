# SiemWazuhWithContainerLab
---

# Installazione di Wazuh tramite Docker

Guida ufficiale: [Wazuh - Deploy with Docker](https://documentation.wazuh.com/current/deployment-options/docker/wazuh-container.html)

## Clonare il repository Wazuh Docker

```bash
git clone https://github.com/wazuh/wazuh-docker.git -b v4.11.2
```

### Accedere alla cartella corretta

Entrare nella directory **single-node** per eseguire i comandi successivi:

```bash
cd wazuh-docker/single-node
```

### Generare i certificati per l'indicizzatore

```bash
docker-compose -f generate-indexer-certs.yml run --rm generator
```

### Avviare i container

- Avvio in primo piano (vedi i log in tempo reale):

```bash
docker-compose up
```

- Avvio in background:

```bash
docker-compose up -d
```

### Accesso all'interfaccia web

Apri il browser e visita:

```
https://localhost
```

- **Credenziali di accesso:**
  - **Username:** Admin
  - **Password:** SecretPassword

---

# Installazione dell'agente Wazuh su Kali Linux

Poiché il comando `apt-key` è deprecato nelle versioni più recenti di Debian e Kali, utilizza il metodo aggiornato seguente.

## Aggiungere il repository Wazuh

### Scaricare e copiare la chiave GPG

```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo tee /usr/share/keyrings/wazuh-archive-keyring.gpg
```

### Aggiungere il repository

```bash
echo "deb [signed-by=/usr/share/keyrings/wazuh-archive-keyring.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
```

### Aggiornare i pacchetti

```bash
sudo apt update
```

## Installare l'agente Wazuh

Sostituisci `WAZUH_MANAGER_IP` con l'indirizzo IP del tuo server Wazuh:

```bash
sudo WAZUH_MANAGER="WAZUH_MANAGER_IP" apt install wazuh-agent
```

## Configurare l'agente

Modifica il file di configurazione:

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Assicurati che la sezione `<client>` sia configurata correttamente:

```xml
<client>
  <server>
    <address>WAZUH_MANAGER_IP</address>
    <port>1514</port>
    <protocol>tcp</protocol>
  </server>
</client>
```

## Avviare l'agente

```bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

## Verificare lo stato dell'agente

```bash
sudo systemctl status wazuh-agent
```

---

# Verifica della connessione

## Trovare l'IP del container Wazuh

Elenca i container e i loro IP utilizzando il comando `docker inspect`:

```bash
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' nome_container_wazuh
```

Esempio:

```bash
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' wazuh
```

## Verificare la comunicazione dell'agente da Kali

Visualizza i log dell'agente:

```bash
tail -f /var/ossec/logs/ossec.log
```

## Generare un errore di autenticazione

Apri una nuova sessione SSH sulla macchina Kali (ad esempio, duplicando la sessione da Putty), accedi come root e genera un errore di autenticazione:

```bash
sudo su
```

Inserisci credenziali errate e osserva i log in tempo reale per un avviso simile a:

```
** Alert 1743597161.761861: - pam,syslog,authentication_failed,pci_dss_10.2.4, ...
2025 Apr 02 12:32:41 (kali) any->journald
Rule: 5557 (level 5) -> 'unix_chkpwd: Password check failed.'
Apr 02 12:32:40 kali unix_chkpwd[43360]: password check failed for user (kali)
```

---

# Accesso al container Wazuh

Entra nel container per verificare i log:

```bash
docker exec -it single-node_wazuh.manager_1 /bin/bash
```

Visualizza i log degli avvisi:

```bash
bash-5.2# tail -f /var/ossec/logs/alerts/alerts.log
```

Ora dovresti poter visualizzare i log anche tramite il browser.
