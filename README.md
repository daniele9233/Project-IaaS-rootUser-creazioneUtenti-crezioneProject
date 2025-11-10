# Project-IaaS-rootUser-creazioneUtenti-crezioneProject

Collezione di ruoli Ansible per la gestione completa di utenti e progetti in Rancher.

## 📋 Ruoli disponibili

### 1. `rancher_create_projects`
Crea progetti in Rancher con rilevamento automatico del cluster.

**Funzionalità**:
- Autenticazione automatica
- Rilevamento automatico del cluster ID
- Creazione progetti con gestione idempotente

**Variabili richieste**:
```yaml
rancher_domain: "rancher.example.com"
rancher_password: "password"
rancher_projects:
  - name: "project-name"
    description: "Project description"
```

---

### 2. `rancher_create_users`
Crea utenti in Rancher e assegna il ruolo "Standard User".

**Funzionalità**:
- Creazione utenti con password temporanea
- Forzatura cambio password al primo login
- Assegnazione automatica ruolo "Standard User"
- Estrazione e mappatura degli ID utente

**Variabili richieste**:
```yaml
rancher_domain: "rancher.example.com"
rancher_password: "password"
rancher_users:
  - username: "alice"
    password: "TempPass123!"
    display_name: "Alice Developer"
```

---

### 3. `rancher_assign_project_owner` ⭐ NUOVO
Assegna utenti come owners di progetti specifici.

**Funzionalità**:
- Assegnazione del ruolo `project-owner` a livello di progetto
- Risoluzione automatica di ID progetti e utenti
- Supporto per assegnazioni multiple
- Gestione intelligente degli errori

**Variabili richieste**:
```yaml
rancher_domain: "rancher.example.com"
rancher_password: "password"
project_owner_assignments:
  - username: "alice"
    project_name: "project-alpha"
  - username: "bob"
    project_name: "project-beta"
```

---

## 🚀 Quick Start

### Esempio completo

Questo esempio crea progetti, utenti e assegna gli owners in un'unica esecuzione:

```yaml
---
- name: Complete Rancher Setup
  hosts: localhost
  gather_facts: no
  vars:
    rancher_domain: "rancher.example.com"
    rancher_password: "{{ lookup('env', 'RANCHER_PASSWORD') }}"
    ansibleSshHost: "localhost"

    # Progetti
    rancher_projects:
      - name: "project-alpha"
        description: "Development Project"
      - name: "project-beta"
        description: "Testing Project"

    # Utenti
    rancher_users:
      - username: "alice"
        password: "TempPass123!"
        display_name: "Alice Developer"
      - username: "bob"
        password: "TempPass456!"
        display_name: "Bob Tester"

    # Assegnazioni owner
    project_owner_assignments:
      - username: "alice"
        project_name: "project-alpha"
      - username: "bob"
        project_name: "project-beta"

  tasks:
    - name: Create projects
      include_role:
        name: rancher_create_projects

    - name: Create users
      include_role:
        name: rancher_create_users

    - name: Assign project owners
      include_role:
        name: rancher_assign_project_owner
```

### Esecuzione

```bash
# Imposta la password di Rancher come variabile d'ambiente
export RANCHER_PASSWORD="your-secure-password"

# Esegui il playbook completo
ansible-playbook example_assign_project_owners.yml

# Oppure esegui solo specifici step con i tag
ansible-playbook example_assign_project_owners.yml --tags "projects,users"
ansible-playbook example_assign_project_owners.yml --tags "owners"
```

---

## 📁 Struttura del repository

```
.
├── roles/
│   ├── rancher_create_projects/
│   │   └── tasks/
│   │       └── main.yml
│   ├── rancher_create_users/
│   │   └── tasks/
│   │       └── main.yml
│   └── rancher_assign_project_owner/
│       ├── tasks/
│       │   └── main.yml
│       └── README.md
├── example_assign_project_owners.yml
└── README.md
```

---

## 🔧 Requisiti

- Ansible >= 2.9
- Accesso HTTPS a Rancher API
- Credenziali admin di Rancher
- Python `requests` (per il modulo `uri`)

---

## 📖 Documentazione dettagliata

Per informazioni dettagliate su ogni ruolo, consulta:
- [rancher_assign_project_owner/README.md](roles/rancher_assign_project_owner/README.md)

---

## 🔐 Note di sicurezza

- ⚠️  Non inserire mai password in chiaro nei playbook
- ✅ Usa `ansible-vault` per le variabili sensibili
- ✅ Usa variabili d'ambiente per le password
- ✅ Modifica `validate_certs: no` in produzione per validare i certificati SSL

### Esempio con ansible-vault

```bash
# Crea file criptato con le variabili sensibili
ansible-vault create vars/secrets.yml

# Contenuto del file:
rancher_password: "SuperSecretPassword"

# Usa nel playbook
ansible-playbook playbook.yml --extra-vars "@vars/secrets.yml" --ask-vault-pass
```

---

## 🤝 Workflow tipico

1. **Creazione infrastruttura**: Usa `rancher_create_projects` per creare i progetti
2. **Creazione utenti**: Usa `rancher_create_users` per creare gli utenti
3. **Assegnazione permessi**: Usa `rancher_assign_project_owner` per assegnare gli owners

Tutti e tre i ruoli sono idempotenti e possono essere eseguiti più volte senza problemi.

---

## 📝 Licenza

Da definire.
