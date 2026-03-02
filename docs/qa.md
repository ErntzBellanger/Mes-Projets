# Plan de tests & checklist QA

## 1) Stratégie
- **Unit tests**: services métier (campaign workflow, fraude rules, withdrawal checks).
- **Integration tests**: API + DB (auth, donations, withdrawals, moderation).
- **E2E tests**: parcours critiques user/admin (Playwright/Cypress).
- **Security smoke tests**: auth brute-force, access control, upload restrictions.

## 2) Cas critiques (MVP)

### Auth
- Register/login/logout OK.
- Reset password (token expiré/valide).
- Lockout après X tentatives login.

### Campagnes
- Création draft -> édition -> soumission.
- Transition refusée si docs requis manquants.
- Modérateur approuve -> campagne visible en public.
- Suspension admin retire CTA don.

### Donations
- Don réussi met à jour `raised_amount`.
- Don échoué n’affecte pas montant collecté.
- Don anonyme masque identité publiquement.

### Withdrawals
- Montant > disponible refusé.
- Retrait normal: requested -> approved -> paid.
- Cas risque: freeze automatique.

### Reports/Fraude
- Utilisateur signale campagne.
- Modérateur traite signalement (resolved/dismissed).
- Plusieurs signalements déclenchent flag risque.

### Audit logs
- Chaque action admin sensible crée une ligne audit.

## 3) Checklist QA release
- Responsive mobile/tablette/desktop.
- Pages légales présentes (CGU, confidentialité, anti-fraude).
- i18n FR complète (pas de clés brutes visibles).
- Performances: LCP acceptable sur home/explorer.
- Accessibilité: contrastes, labels, navigation clavier.
- Sécurité: headers HTTP (CSP, HSTS, X-Frame-Options), rate limits actifs.
- Monitoring: logs centralisés + alertes erreurs 5xx.

## 4) Commandes type (CI)
- `pnpm lint`
- `pnpm typecheck`
- `pnpm test`
- `pnpm test:e2e`

## 5) Critères d’acceptation MVP
- Un porteur peut publier une campagne validée.
- Un donateur peut découvrir et faire un don one-time.
- Un modérateur peut approuver/suspendre et traiter signalements.
- Un retrait peut être demandé, revu, approuvé/payé ou gelé.
- Toutes actions sensibles sont auditables.
