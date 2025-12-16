# Core Banking Poller Process API

API Process qui synchronise périodiquement les données clients et comptes depuis Core Banking vers Global Data.

## Description

Cette API utilise un scheduler pour déclencher périodiquement la synchronisation des clients et comptes depuis Core Banking vers Global Data. Elle supporte également des synchronisations à la demande via des endpoints HTTP.

## Endpoints

### POST /api/poller/core-banking/sync/customers
Déclenche une synchronisation manuelle des clients depuis Core Banking vers Global Data.

**Response:** 202 Accepted avec syncId

### POST /api/poller/core-banking/sync/accounts
Déclenche une synchronisation manuelle des comptes depuis Core Banking vers Global Data.

**Response:** 202 Accepted avec syncId

## Configuration

### Scheduler

Configurer dans `src/main/resources/config.properties`:

```properties
# Scheduler Configuration
scheduler.frequency=3600000
scheduler.startDelay=60000

# HTTP Configuration
http.port=8082
http.host=0.0.0.0
```

- `scheduler.frequency`: Fréquence en millisecondes (ex: 3600000 = 1 heure)
- `scheduler.startDelay`: Délai avant le premier déclenchement en millisecondes

### Connexions HTTP Requises

- **Core Banking Customers System API** (port 8081)
- **Core Banking Accounts System API** (port 8081)
- **Global Party System API** (port 8081)
- **Global Financial Account System API** (port 8081)

Configurer dans `global.xml`:
- `Core_Banking_Customers_System_API_Config`
- `Core_Banking_Accounts_System_API_Config`
- `Global_Party_System_API_Config`
- `Global_Financial_Account_System_API_Config`

### Port

- **Port HTTP**: 8082

## Architecture Technique

### Flows Schedulés

- `scheduled-sync-customers-flow`: Synchronisation périodique des clients (déclenché par scheduler)
- `scheduled-sync-accounts-flow`: Synchronisation périodique des comptes (déclenché par scheduler)

### Flows Business-Logic (On-Demand)

- `sync-customers-business-logic`: Synchronisation manuelle des clients (appelé via POST /sync/customers)
- `sync-accounts-business-logic`: Synchronisation manuelle des comptes (appelé via POST /sync/accounts)

### Subflows

- `sync-customers-subflow`: Logique de synchronisation des clients
- `sync-accounts-subflow`: Logique de synchronisation des comptes

### Processus de Synchronisation

1. Récupère les données depuis Core Banking System API
2. Transforme au format Global Data
3. Pour chaque entité, appelle Global Data System API pour upsert
4. Gère les erreurs individuellement (continue même en cas d'erreur sur une entité)

## Exemples de Requêtes

### POST /api/poller/core-banking/sync/customers

```bash
curl -X POST http://localhost:8082/api/poller/core-banking/sync/customers
```

**Response:**
```json
{
  "message": "Customer sync completed",
  "syncId": "550e8400-e29b-41d4-a716-446655440000",
  "timestamp": "2024-01-15T10:30:00Z"
}
```

### POST /api/poller/core-banking/sync/accounts

```bash
curl -X POST http://localhost:8082/api/poller/core-banking/sync/accounts
```

