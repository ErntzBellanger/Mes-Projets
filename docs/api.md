# API MVP (REST JSON)

Base URL: `/api/v1`

## Standards
- Auth: JWT session (httpOnly cookie) ou bearer token.
- Format erreurs:
```json
{ "error": { "code": "VALIDATION_ERROR", "message": "...", "details": [] } }
```
- Validation: Zod/Joi côté API.
- RBAC par middleware.
- Rate limit: login, forgot-password, reports.

## Auth

### `POST /auth/register`
Body:
```json
{ "email": "user@mail.com", "password": "StrongPwd!23", "fullName": "Jean Pierre" }
```
Validations:
- email valide unique
- mot de passe min 8 + complexité

### `POST /auth/login`
Body:
```json
{ "email": "user@mail.com", "password": "StrongPwd!23" }
```
- Rate limiting strict + lockout progressif.

### `POST /auth/forgot-password`
Body:
```json
{ "email": "user@mail.com" }
```
- Réponse neutre (ne pas révéler existence compte).

### `POST /auth/reset-password`
Body: token + newPassword

## Campagnes publiques

### `GET /campaigns`
Query:
- `search`, `category`, `city`, `verified`, `urgent`, `sort=trending|recent|goal`
Retour:
- liste paginée campaigns publiées.

### `GET /campaigns/:id`
Retour détail campagne + média + updates + stats + donateurs publics.

## Espace porteur de campagne

### `POST /campaigns` (draft)
Body (ex):
```json
{
  "title": "Aide médicale pour...",
  "categoryId": "uuid",
  "goalAmount": 5000,
  "story": "...",
  "city": "Port-au-Prince"
}
```
Validations:
- goalAmount min 50 / max 1 000 000 (configurable)
- title longueur 10-140

### `PUT /campaigns/:id`
- Autorisé si owner + statut `draft` ou `submitted` (selon règle métier).

### `POST /campaigns/:id/submit`
- Vérifie pièces minimales requises avant passage `submitted` puis `under_review`.

### `POST /campaigns/:id/updates`
Body:
```json
{ "title": "Update semaine 1", "content": "..." }
```
- Seulement owner, campagne publiée.

## Donations

### `POST /donations`
Body:
```json
{
  "campaignId": "uuid",
  "amount": 25,
  "currency": "USD",
  "isAnonymous": true,
  "message": "Courage"
}
```
Règles:
- montant > 0
- campagne status `published`
- crée `payment_transaction` en `initiated`
- redirige/initialise paiement provider

Webhook provider:
- `POST /payments/webhook/:provider`
- signature vérifiée + idempotence (`provider_txn_id` unique)

## Withdrawals

### `POST /withdrawals`
Body:
```json
{ "campaignId": "uuid", "amount": 1200 }
```
Règles:
- owner uniquement
- amount <= fonds disponibles
- crée statut `requested`
- passe moteur de risque (hold auto possible -> `frozen`/`in_review`)

## Reports

### `POST /reports`
Body:
```json
{ "campaignId": "uuid", "category": "scam", "description": "..." }
```
Règles:
- utilisateur connecté
- rate limit anti-abus

## Admin/Moderation

### `GET /admin/campaigns`
Filtres: `status`, `verificationLevel`, `flags`, `dateRange`

### `POST /admin/campaigns/:id/approve`
Body: `{ "note": "..." }`
Effets:
- statut `published`
- log audit obligatoire

### `POST /admin/campaigns/:id/suspend`
Body: `{ "reason": "..." }`
Effets:
- statut `suspended`
- retraits gelés
- audit log

### `POST /admin/withdrawals/:id/approve`
- `requested|in_review -> approved`
- audit log + déclenchement payout provider

### `POST /admin/withdrawals/:id/freeze`
- statut `frozen`, motif obligatoire.

### `GET /admin/reports`
- liste signalements + workflow traitement.

## Uploads médias/docs

### `POST /uploads/presign`
Body:
```json
{ "scope": "campaign_media", "fileName": "image.jpg", "mimeType": "image/jpeg", "size": 1500000 }
```
Validations:
- MIME autorisés: `image/jpeg`, `image/png`, `application/pdf`
- Taille max: images 5MB, docs 10MB
Retour:
- URL signée PUT + `storageKey`

## CSRF/Cookies
- Si auth cookie: CSRF token obligatoire sur requêtes state-changing.
- Sinon bearer token + SameSite strict pour cookies annexes.
