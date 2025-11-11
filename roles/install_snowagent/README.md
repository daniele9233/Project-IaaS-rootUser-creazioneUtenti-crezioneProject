# SnowAgent Installation Role

Ruolo Ansible per l'installazione automatica di Snow Software Agent su server Linux (RHEL-like e Ubuntu/Debian).

## 📋 Descrizione

Questo ruolo installa Snow Software Agent per inventory e monitoring su:
- **RHEL-like**: RedHat, CentOS, Rocky Linux, AlmaLinux (pacchetti .rpm)
- **Debian-like**: Ubuntu, Debian (pacchetti .deb)

## 📦 Pacchetti supportati

Il ruolo supporta due tipi di agenti:

### 1. Oracle Agent
- `ALMAVIVA-740-Linux-Oracle-snowagent-7.4.0-x64.deb` (Ubuntu/Debian)
- `ALMAVIVA-740-Linux-Oracle-snowagent-7.4.0-x64.rpm` (RHEL-like)

### 2. ScaloHTTP Agent
- `ScaloHTTP-snowagent-7.0.1-x64.deb` (Ubuntu/Debian)
- `ScaloHTTP-snowagent-7.0.1-x64.rpm` (RHEL-like)

## 🚀 Prerequisiti

### 1. Posizionare i pacchetti

Copiare i 4 pacchetti nella directory `files/` del ruolo:

```bash
cd roles/install_snowagent/files/
# Copiare qui i 4 pacchetti .deb e .rpm
ls -lh
```

Dovrebbe mostrare:
```
ALMAVIVA-740-Linux-Oracle-snowagent-7.4.0-x64.deb
ALMAVIVA-740-Linux-Oracle-snowagent-7.4.0-x64.rpm
ScaloHTTP-snowagent-7.0.1-x64.deb
ScaloHTTP-snowagent-7.0.1-x64.rpm
```

### 2. Requisiti Ansible

- Ansible >= 2.9
- Moduli: `copy`, `apt`, `yum`, `systemd`, `file`

## 📖 Variabili

### Variabili configurabili (defaults)

| Variabile | Descrizione | Default | Opzioni |
|-----------|-------------|---------|---------|
| `snowagent_type` | Tipo di agente da installare | `oracle` | `oracle`, `scalo` |
| `snowagent_enable_service` | Abilita servizio all'avvio | `true` | `true`, `false` |
| `snowagent_start_service` | Avvia servizio dopo install | `true` | `true`, `false` |
| `snowagent_service_name` | Nome del servizio | `snowagent` | - |
| `snowagent_temp_dir` | Directory temporanea | `/tmp/snowagent` | - |

## 💻 Utilizzo

### Esempio 1: Installazione Oracle Agent

```yaml
---
- name: Install SnowAgent Oracle on all servers
  hosts: all_servers
  become: yes
  vars:
    snowagent_type: "oracle"
  roles:
    - install_snowagent
```

### Esempio 2: Installazione ScaloHTTP Agent

```yaml
---
- name: Install SnowAgent ScaloHTTP on monitoring servers
  hosts: monitoring_servers
  become: yes
  vars:
    snowagent_type: "scalo"
  roles:
    - install_snowagent
```

### Esempio 3: Installazione senza gestione servizio

```yaml
---
- name: Install SnowAgent without service management
  hosts: target_servers
  become: yes
  vars:
    snowagent_type: "oracle"
    snowagent_enable_service: false
    snowagent_start_service: false
  roles:
    - install_snowagent
```

### Esempio 4: Installazione su gruppi misti (RHEL + Ubuntu)

```yaml
---
- name: Install SnowAgent on mixed environment
  hosts: all
  become: yes
  vars:
    snowagent_type: "oracle"
  roles:
    - install_snowagent
```

Il ruolo **rileva automaticamente** il sistema operativo e installa il pacchetto corretto (.deb o .rpm).

## 🎯 Playbook completo di esempio

```yaml
---
- name: Complete SnowAgent Deployment
  hosts: all
  become: yes
  gather_facts: yes

  vars:
    # Scegli il tipo di agente
    snowagent_type: "oracle"  # o "scalo"

    # Configurazione servizio
    snowagent_enable_service: true
    snowagent_start_service: true

  pre_tasks:
    - name: Ensure system is up to date (Ubuntu/Debian)
      apt:
        update_cache: yes
      when: ansible_os_family == "Debian"

    - name: Ensure system is up to date (RHEL-like)
      yum:
        update_cache: yes
      when: ansible_os_family == "RedHat"

  roles:
    - install_snowagent

  post_tasks:
    - name: Verify SnowAgent installation
      command: which snowagent
      register: snowagent_check
      ignore_errors: yes

    - name: Display verification result
      debug:
        msg: "SnowAgent installed: {{ snowagent_check.rc == 0 }}"
```

## 📂 Struttura del ruolo

```
install_snowagent/
├── defaults/
│   └── main.yml           # Variabili di default
├── files/
│   ├── .gitkeep
│   ├── ALMAVIVA-740-Linux-Oracle-snowagent-7.4.0-x64.deb
│   ├── ALMAVIVA-740-Linux-Oracle-snowagent-7.4.0-x64.rpm
│   ├── ScaloHTTP-snowagent-7.0.1-x64.deb
│   └── ScaloHTTP-snowagent-7.0.1-x64.rpm
├── tasks/
│   └── main.yml           # Task principale
└── README.md              # Questa documentazione
```

## 🔄 Workflow di installazione

1. **Rilevamento OS** - Identifica se è Debian o RHEL-like
2. **Selezione pacchetto** - Sceglie .deb o .rpm in base all'OS
3. **Copia pacchetto** - Trasferisce il file dal controller al target
4. **Installazione** - Usa `apt` o `yum` per installare
5. **Gestione servizio** - Abilita e avvia il servizio (opzionale)
6. **Cleanup** - Rimuove file temporanei
7. **Verifica** - Mostra riepilogo installazione

## 🎨 Personalizzazione

### Cambiare versione pacchetti

Modifica `defaults/main.yml`:

```yaml
snowagent_packages:
  oracle:
    deb: "ALMAVIVA-800-Linux-Oracle-snowagent-8.0.0-x64.deb"
    rpm: "ALMAVIVA-800-Linux-Oracle-snowagent-8.0.0-x64.rpm"
```

### Installazione su host specifici

```yaml
---
- name: Install SnowAgent on specific hosts
  hosts: all
  become: yes
  vars:
    snowagent_type: "{{ 'oracle' if inventory_hostname in groups['oracle_servers'] else 'scalo' }}"
  roles:
    - install_snowagent
```

## 🔧 Troubleshooting

### Problema: Pacchetto non trovato

**Errore**: `Could not find or access 'ALMAVIVA-740-Linux-Oracle-snowagent-7.4.0-x64.deb'`

**Soluzione**:
```bash
# Verifica che i file siano nella directory corretta
ls -lh roles/install_snowagent/files/

# Verifica i permessi
chmod 644 roles/install_snowagent/files/*.deb
chmod 644 roles/install_snowagent/files/*.rpm
```

### Problema: Installazione fallita per dipendenze

**Errore**: `dependency X is not satisfied`

**Soluzione Ubuntu/Debian**:
```bash
# Sul server target
sudo apt-get install -f
```

**Soluzione RHEL/CentOS**:
```bash
# Sul server target
sudo yum install <dipendenza-mancante>
```

### Problema: Servizio non esiste

**Warning**: `Could not find the requested service snowagent`

**Soluzione**: Se il pacchetto non include un servizio systemd, disabilita la gestione del servizio:
```yaml
snowagent_enable_service: false
snowagent_start_service: false
```

## 📊 Output atteso

```
TASK [install_snowagent : Display installation info]
ok: [server1] => {
    "msg": [
        "Installing SnowAgent: oracle",
        "Target OS Family: RedHat",
        "Target Distribution: Rocky 8.5"
    ]
}

TASK [install_snowagent : Copy SnowAgent package to target server]
changed: [server1]

TASK [install_snowagent : Install SnowAgent on RHEL/CentOS/Rocky/AlmaLinux]
changed: [server1]

TASK [install_snowagent : Display installation summary]
ok: [server1] => {
    "msg": [
        "✅ SnowAgent installation completed successfully!",
        "Agent type: oracle",
        "Package: ALMAVIVA-740-Linux-Oracle-snowagent-7.4.0-x64.rpm",
        "OS: Rocky 8.5"
    ]
}
```

## 🔐 Note di sicurezza

- Il ruolo richiede privilegi `sudo` (`become: yes`)
- I pacchetti .rpm vengono installati con `disable_gpg_check: yes` - considera di firmare i pacchetti in produzione
- I file temporanei vengono rimossi automaticamente dopo l'installazione

## 📝 Tags supportati

```bash
# Installa solo SnowAgent
ansible-playbook site.yml --tags "snowagent"

# Esclude installazione SnowAgent
ansible-playbook site.yml --skip-tags "snowagent"
```

## 🤝 Integrazione nel progetto

Per integrare nel tuo `site.yml`:

```yaml
# Nel tuo site.yml
- name: Install SnowAgent on all managed servers
  hosts: all
  become: yes
  gather_facts: yes
  vars:
    snowagent_type: "oracle"
  roles:
    - { role: install_snowagent, tags: ['snowagent'] }
```

## 📞 Supporto

Per problemi o domande specifiche sul ruolo, verifica:
1. I pacchetti sono nella directory `files/`
2. Il sistema operativo è supportato (RHEL-like o Debian-like)
3. Hai privilegi sudo sul server target
4. I log di sistema: `journalctl -xe` o `/var/log/messages`

## 📄 Licenza

Da definire in base al progetto.
