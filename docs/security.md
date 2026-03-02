# Sécurité MVP & anti-fraude

## 1) Mesures sécurité de base
- Hash mots de passe: Argon2id/bcrypt cost élevé.
- Session sécurisée: cookies httpOnly + secure + sameSite=lax/strict.
- TLS obligatoire en production.
- Secrets via variables d’environnement (rotation planifiée).
- Validation stricte des entrées (schema validation).
- Protection XSS: escaping + CSP de base.
- Protection SQLi: ORM paramétré uniquement.
- Protection brute-force: rate limiting + lockout login.
- Journalisation erreurs sans fuite de données sensibles.

## 2) Contrôles d’accès
- RBAC strict (user, campaign_owner, moderator, admin).
- Vérification ownership sur toutes routes user.
- Séparation endpoints admin sous `/admin/*` + audit systématique.

## 3) Upload & stockage
- Pré-signature serveur uniquement (pas de credentials S3 côté client).
- Contrôle MIME + extension + taille.
- Antivirus async recommandé (MVP+: ClamAV job).
- URLs signées courtes pour consultation docs KYC.
- Buckets séparés: `public-media` vs `private-documents`.

## 4) Anti-fraude MVP

### Niveaux de vérification
- **Niveau 0**: email + téléphone vérifiés.
- **Niveau 1**: pièce d’identité + selfie.
- **Niveau 2**: justificatifs additionnels (ex: médical / gros montants) + revue manuelle.

### Règles de risque retraits (hold auto)
Déclencher `withdrawal.status=frozen|in_review` si:
- Campagne récente (< 7 jours) + montant retrait élevé.
- Trop de signalements ouverts.
- Incohérences documents (rejet antérieur, mismatch identité).
- Pics de dons anormaux (velocity rules simples).

### Signalements communautaires
- Utilisateurs connectés peuvent signaler.
- Catégories normalisées + description obligatoire.
- Score de risque campagne augmenté selon volume/gravité.

## 5) Audit logs obligatoires
Actions à tracer:
- approbation/suspension campagne
- approbation/rejet/gel retrait
- changement de rôles/permissions
- modifications paramètres commissions/catégories

Structure minimale log:
- acteur, action, entité, ancien/nouvel état, IP, user-agent, timestamp.

## 6) Conformité minimale (RGPD-like)
- Consentement cookies minimal (essentiels only par défaut).
- Politique confidentialité accessible.
- Droit de suppression/rectification (process manuel MVP).
- Durée rétention documents KYC définie (ex: 12-24 mois).
