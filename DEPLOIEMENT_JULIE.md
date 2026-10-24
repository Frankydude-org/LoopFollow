# Déploiement LoopFollow sur l'iPhone de Julie — TestFlight interne

## Objectif

Julie doit pouvoir utiliser **LoopFollow** et recevoir les nouvelles versions automatiquement, sans devenir développeuse d'applications ni faire approuver chaque version par la revue bêta externe Apple.

**Choix validé le 24 octobre 2026 : distribution TestFlight INTERNE.**
Ne pas utiliser un groupe de testeurs externes pour Julie.

- GitHub : https://github.com/Frankydude-org/LoopFollow (`main`)
- Source officielle : https://github.com/loopandlearn/LoopFollow (`main`)
- App Store Connect : https://appstoreconnect.apple.com/apps
- Documentation Apple officielle : https://developer.apple.com/help/app-store-connect/test-a-beta-version/add-internal-testers

## Automatisation existante

Le workflow officiel `.github/workflows/build_LoopFollow.yml`, nommé **4. Build LoopFollow**, est conservé tel quel :

- Exécution programmée chaque dimanche à 10:17 UTC (`17 10 * * 0`) ;
- synchronisation du fork avec `loopandlearn/LoopFollow:main` ;
- nouvelle compilation lorsqu'une modification existe ;
- compilation supplémentaire le deuxième dimanche du mois, même sans modification, pour renouveler une version TestFlight avant l'expiration de 90 jours ;
- transmission à App Store Connect par Fastlane.

Laisser `SCHEDULED_BUILD` et `SCHEDULED_SYNC` absentes, ou sur `true`. Ne pas suivre `dev` pour le téléphone de Julie.

**Distribution interne automatique :** elle se configure entièrement dans App Store Connect, via **TestFlight → Internal Testing → groupe Julie → Enable automatic distribution**. Apple rend les builds traités disponibles automatiquement aux membres du groupe. Aucun workflow GitHub additionnel pour distribution externe n'est requis.

Le workflow externe précédemment préparé a été **retiré** pour éviter une distribution erronée ou une soumission involontaire à la Beta App Review.

## Vérification des secrets : complétée

Workflow `.github/workflows/verify_fork.yml` : validation renforcée réussie le 24 octobre 2026.

Exécution : https://github.com/Frankydude-org/LoopFollow/actions/runs/40117959936

Les six secrets d'organisation suivants ont été accessibles depuis LoopFollow :
`TEAMID`, `FASTLANE_ISSUER_ID`, `FASTLANE_KEY_ID`, `FASTLANE_KEY`, `GH_PAT`, `MATCH_PASSWORD`.

Fastlane s'est exécuté correctement, et le dépôt privé `Frankydude-org/Match-Secrets` a été déchiffré. Aucun besoin identifié de régénérer les secrets.

Ne pas modifier `ENABLE_NUKE_CERTS` ou forcer le renouvellement des certificats sans contrôler les impacts sur Trio et Loop, qui partagent la signature.

## Procédure initiale, dans l'ordre

### 1. Créer les identifiants d'application Apple

Depuis https://github.com/Frankydude-org/LoopFollow/actions/workflows/add_identifiers.yml :

- Choisir `Run workflow` → `main` ;
- vérifier la conclusion `Success` ;
- ce workflow crée les identifiants `com.<TEAMID>.LoopFollow` et `com.<TEAMID>.LoopFollow.LoopFollowLAExtension` avec les capacités requises ;
- remplacer `<TEAMID>` par la vraie valeur de l'Apple Developer Team ID (secret, consultable dans le portail Apple). Ne pas partager de clé privée.

### 2. Créer l'application dans App Store Connect

Dans https://appstoreconnect.apple.com/apps → `+` → `New App` :
- Plateforme iOS ;
- nom disponible (nom administratif; l'icône installée utilisera le nom LoopFollow défini dans le projet) ;
- langue et SKU librement choisis ;
- choisir le Bundle ID créé à l'étape 1 ;
- créer l'enregistrement, mais **ne pas soumettre l'application à l'App Store**.

### 3. Ajouter Julie comme utilisatrice App Store Connect

Dans https://appstoreconnect.apple.com/access/users → `+` :
- inviter Julie à son adresse courriel habituelle ;
- rôle **Marketing** (éligible aux tests internes, permissions moindres que Developer/Admin) ;
- limiter son accès **uniquement à LoopFollow** ;
- ne pas donner d'accès `Certificates, Identifiers & Profiles`, ni accès aux rapports ;
- Julie accepte son invitation (Apple l'indique comme expirant après 3 jours).

Note : **Admin n'est pas nécessaire** et ne permet pas de limiter son accès aux autres applications.

### 4. Créer son groupe de test interne

Dans `Apps → LoopFollow → TestFlight → Internal Testing → +` :
- Groupe : **Julie** ;
- cocher **Enable automatic distribution** ;
- cliquer sur Create ;
- `Invite Testers` → choisir Julie (utilisatrice App Store Connect) ;
- Apple peut limiter le nombre à 100 testeurs internes par application, sans révision bêta externe.

### 5. Premier build GitHub

Dans https://github.com/Frankydude-org/LoopFollow/actions/workflows/build_LoopFollow.yml :
- Lancer **4. Build LoopFollow** manuellement sur `main` ;
- vérifier l'exécution complète et le téléversement TestFlight ;
- attendre le traitement par Apple ; confirmer la disponibilité du build dans le groupe interne Julie.

### 6. Installation et mises à jour sur l'iPhone de Julie

- Installer TestFlight depuis l'App Store, accepter l'invitation reçue, installer LoopFollow ;
- dans TestFlight, ouvrir LoopFollow et activer **Automatic Updates** ;
- pour l'avenir, le workflow GitHub génère les builds et le groupe interne les reçoit automatiquement après traitement Apple. L'installation automatique dépend des réglages TestFlight/iOS et n'est pas instantanée.

### 7. Configuration Nightscout

Configurer l'URL T1Pal/Nightscout et un accès de lecture seule uniquement **sur l'iPhone de Julie**. Éviter de mettre les jetons, données médicales ou identifiants Nightscout dans un dépôt GitHub public.

## Limites et prudence

- Les builds TestFlight expirent après 90 jours; renouvellement mensuel programmé.
- La revue bêta Apple des testeurs externes n'est pas exigée pour le groupe interne, mais un traitement Apple de chaque build peut prendre un délai et des exigences techniques ou de conformité peuvent s'appliquer.
- Ne pas configurer Julie comme **Admin**.
- Les alertes LoopFollow (glycémie basse, etc.) peuvent être retardées par iOS, la transmission Trio/Nightscout ou l'absence de réseau; ne pas les utiliser comme seul dispositif de sécurité.
