# Rancher Assign Project Owner Role

## Descrizione

Questo ruolo Ansible permette di assegnare utenti come **owners** di progetti specifici in Rancher. Si integra perfettamente con i ruoli `rancher_create_projects` e `rancher_create_users`.

## Funzionalità

- ✅ Autenticazione automatica a Rancher API
- ✅ Risoluzione automatica degli ID di progetti e utenti
- ✅ Assegnazione del ruolo `project-owner` tramite `projectRoleTemplateBinding`
- ✅ Gestione intelligente degli errori (skip se progetto/utente non esiste)
- ✅ Supporto per assegnazioni multiple (un utente può essere owner di più progetti)
- ✅ Idempotente: non fallisce se l'assegnazione esiste già

## Variabili richieste

### Configurazione Rancher

| Variabile | Descrizione | Obbligatorio | Default |
|-----------|-------------|--------------|---------|
| `rancher_domain` | Dominio di Rancher (es. `rancher.example.com`) | ✅ Sì | - |
| `rancher_password` | Password dell'admin di Rancher | ✅ Sì | - |
| `rancher_api_user` | Username admin di Rancher | ❌ No | `admin` |
| `ansibleSshHost` | Host per delegate_to | ❌ No | `localhost` |

### Assegnazioni

| Variabile | Descrizione | Tipo | Obbligatorio |
|-----------|-------------|------|--------------|
| `project_owner_assignments` | Lista di assegnazioni utente→progetto | Lista | ✅ Sì |

## Formato della variabile `project_owner_assignments`

```yaml
project_owner_assignments:
  - username: "nome_utente"      # Username dell'utente (deve esistere in Rancher)
    project_name: "nome_progetto" # Nome del progetto (deve esistere in Rancher)
  - username: "altro_utente"
    project_name: "altro_progetto"
```

## Esempio d'uso

### Esempio semplice

```yaml
---
- name: Assign Project Owners
  hosts: localhost
  gather_facts: no
  vars:
    rancher_domain: "rancher.example.com"
    rancher_password: "{{ lookup('env', 'RANCHER_PASSWORD') }}"

    project_owner_assignments:
      - username: "alice"
        project_name: "project-alpha"
      - username: "bob"
        project_name: "project-beta"

  roles:
    - rancher_assign_project_owner
```

### Esempio completo con creazione utenti e progetti

```yaml
---
- name: Complete Setup
  hosts: localhost
  gather_facts: no
  vars:
    rancher_domain: "rancher.example.com"
    rancher_password: "SuperSecretPassword"

    # Progetti da creare
    rancher_projects:
      - name: "project-alpha"
        description: "Development Project"
      - name: "project-beta"
        description: "Testing Project"

    # Utenti da creare
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

    - name: Assign owners
      include_role:
        name: rancher_assign_project_owner
```

## Come funziona

### 1. Autenticazione

Il ruolo si autentica su Rancher usando le credenziali admin:

```yaml
POST https://{{ rancher_domain }}/v3-public/localProviders/local?action=login
```

### 2. Risoluzione ID

Recupera gli ID di tutti i progetti e utenti da Rancher:

- **Progetti**: `GET /v3/projects`
- **Utenti**: `GET /v3/users`

### 3. Assegnazione Owner

Per ogni assegnazione in `project_owner_assignments`, crea un `projectRoleTemplateBinding`:

```yaml
POST https://{{ rancher_domain }}/v3/projectroletemplatebindings
Body:
  type: "projectRoleTemplateBinding"
  roleTemplateId: "project-owner"
  projectId: "<project_id>"
  userId: "<user_id>"
```

## Codici di stato HTTP

| Codice | Significato |
|--------|-------------|
| `201` | ✅ Assegnazione creata con successo |
| `409` | ℹ️  Assegnazione già esistente (OK, idempotente) |
| `422` | ⚠️  Errore di validazione (progetto o utente non trovato) |

## Gestione errori

- Se un progetto non esiste, l'assegnazione viene **skippata** con un warning
- Se un utente non esiste, l'assegnazione viene **skippata** con un warning
- Le assegnazioni già esistenti (409) vengono gestite senza errori

## Output del ruolo

Il ruolo fornisce debug output per:
1. ✅ ID progetti estratti
2. ✅ ID utenti estratti
3. ✅ Risultato di ogni assegnazione
4. ⚠️  Assegnazioni skippate (con motivo)

## Ruoli Rancher supportati

Attualmente il ruolo assegna sempre `project-owner`. Se vuoi usare altri ruoli, modifica il valore di `roleTemplateId`:

- `project-owner`: Owner del progetto (accesso completo)
- `project-member`: Membro del progetto (accesso limitato)
- `read-only`: Solo lettura

## Integrazione con altri ruoli

Questo ruolo è progettato per essere usato insieme a:

1. **rancher_create_projects**: Crea i progetti
2. **rancher_create_users**: Crea gli utenti
3. **rancher_assign_project_owner**: Assegna gli owners (questo ruolo)

Vedi `example_assign_project_owners.yml` per un esempio completo.

## Requisiti

- Ansible >= 2.9
- Accesso HTTPS a Rancher API
- Credenziali admin di Rancher
- Moduli: `uri`, `set_fact`, `debug`

## Note di sicurezza

- ⚠️  Non inserire mai password in chiaro nei playbook
- ✅ Usa `ansible-vault` per le variabili sensibili
- ✅ Usa variabili d'ambiente: `lookup('env', 'RANCHER_PASSWORD')`
- ✅ Il ruolo imposta `validate_certs: no` per testing, modificalo in produzione

## Troubleshooting

### Problema: "Assegnazione skippata"

**Causa**: Il progetto o l'utente specificato non esiste in Rancher.

**Soluzione**:
- Verifica che i progetti siano stati creati prima
- Verifica che gli utenti siano stati creati prima
- Controlla i nomi nella lista `project_owner_assignments`

### Problema: "401 Unauthorized"

**Causa**: Credenziali Rancher non valide.

**Soluzione**: Verifica `rancher_password` e `rancher_api_user`.

### Problema: "Connection refused"

**Causa**: Rancher non raggiungibile.

**Soluzione**: Verifica `rancher_domain` e connettività di rete.

## Autore

Basato sui ruoli `rancher_create_projects` e `rancher_create_users`.

## Licenza

Da definire in base al progetto.
