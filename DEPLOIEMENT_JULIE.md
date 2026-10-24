# LoopFollow – déploiement et suivi TestFlight pour Julie

Dépôt : https://github.com/Frankydude-org/LoopFollow
Source officielle (stable) : https://github.com/loopandlearn/LoopFollow, branche `main`.

## Architecture et cadence

- Le workflow officiel `4. Build LoopFollow` (`.github/workflows/build_LoopFollow.yml`) est conservé sans modification.
- Cron officiel : `17 10 * * 0` (dimanche à 10 h 17 UTC). Chaque dimanche, tentative de synchronisation de `main` sur la version stable amont ; build si des commits sont nouveaux.
- Le deuxième dimanche de chaque mois, build même sans changement, pour maintenir les builds TestFlight avant leur expiration à 90 jours.
- Laisser `SCHEDULED_SYNC` et `SCHEDULED_BUILD` non définies (ou `true`).
- Ne pas suivre la branche `dev` pour l'iPhone de Julie. Ne pas personnaliser le workflow officiel pour faciliter la synchronisation du fork.

## Secrets / variables nécessaires

Réutiliser au niveau de l'organisation les secrets déjà employés par Trio / Loop, en vérifiant leur disponibilité POUR CE NOUVEAU DÉPÔT :

`TEAMID`, `FASTLANE_ISSUER_ID`, `FASTLANE_KEY_ID`, `FASTLANE_KEY`, `GH_PAT`, `MATCH_PASSWORD`.

**Important :** les secrets d'organisation peuvent être restreints à certains dépôts. Un nouveau fork ne reçoit pas forcément accès aux secrets existants. Les valeurs des secrets ne sont pas lisibles via l'API GitHub (conception normale). Ne jamais coller leurs valeurs dans un ticket, un chat ou un fichier du dépôt.

- `GH_PAT` : vérifier l'accès privé à `Frankydude-org/Match-Secrets` ainsi que les autorisations `repo` et `workflow` adaptées au PAT.
- `ENABLE_NUKE_CERTS` : facultatif pour un build avec certificat valide ; si activé, le renouvellement peut révoquer des certificats de distribution partagés par d'autres applications. Vérifier l'impact sur Trio / Loop avant de l'activer ou de le modifier. **Aucune modification automatique de cette variable n'est effectuée ici.**
- `TESTFLIGHT_EXTERNAL_GROUP` : variable de dépôt, à créer **après** la configuration Apple, avec exactement le nom du groupe de testeurs externes (ex. `Julie`). Si elle est absente, la distribution externe est ignorée sans bloquer le workflow officiel.

## Actions manuelles, dans l'ordre

1. Dans GitHub, ouvrir [Actions](https://github.com/Frankydude-org/LoopFollow/actions). Si GitHub demande d'activer les workflows du fork, autoriser explicitement l'exécution.
2. Ouvrir [Actions → 1. Validate Secrets](https://github.com/Frankydude-org/LoopFollow/actions/workflows/validate_secrets.yml), lancer `Run workflow` sur `main`. Cela valide les secrets / l'accès à `Match-Secrets`. Résoudre toute erreur AVANT d'aller plus loin.
3. **À la prochaine étape avec Apple seulement** : lancer [2. Add Identifiers](https://github.com/Frankydude-org/LoopFollow/actions/workflows/add_identifiers.yml) pour créer `com.TEAMID.LoopFollow` et son extension. Le préfixe TEAMID est substitué par la vraie valeur du secret.
4. Dans [App Store Connect](https://appstoreconnect.apple.com/apps), créer l'application iOS correspondant à ce Bundle ID, sans publication App Store.
5. Dans TestFlight, créer un groupe de testeurs **externes**, par exemple `Julie`, et inviter Julie avec l'adresse Apple qu'elle utilise sur l'iPhone. La première distribution externe exige généralement une revue bêta Apple.
6. Dans [Settings → Secrets and variables → Actions → Variables](https://github.com/Frankydude-org/LoopFollow/settings/variables/actions), ajouter la variable de dépôt `TESTFLIGHT_EXTERNAL_GROUP` avec le nom exact du groupe externe.
7. Lancer [4. Build LoopFollow](https://github.com/Frankydude-org/LoopFollow/actions/workflows/build_LoopFollow.yml) manuellement sur `main`. Après un build réussi et son transfert vers App Store Connect, le workflow local `5. Distribute LoopFollow to external TestFlight testers` est déclenché, attend le traitement Apple et tente l'affectation au groupe externe. Si Apple n'a pas encore approuvé la version, il faut finaliser la revue avant la distribution ; la distribution peut être relancée manuellement.
8. Sur l'iPhone de Julie, installer TestFlight, accepter l'invitation puis installer LoopFollow. Dans TestFlight → LoopFollow → `Automatic Updates`, activer les mises à jour automatiques.

## Workflow de distribution externe

Fichier local : `.github/workflows/distribute_external_testflight.yml`.

- Écoute uniquement la **réussite** du workflow officiel `4. Build LoopFollow`, plus le déclenchement manuel.
- Ne fonctionne que si `TESTFLIGHT_EXTERNAL_GROUP` est défini.
- Distribue la dernière compilation déjà téléversée via `fastlane pilot distribute`, sans reconstruire l'IPA et sans toucher aux certificats `Match`.
- Utilise temporairement les secrets App Store Connect existants; supprime le fichier d'authentification à la fin.
- Peut échouer si le build n'est pas encore admissible à la distribution externe, si l'approbation Apple manque, ou si la clé App Store Connect n'a pas les droits nécessaires. Une exécution réussie doit être confirmée dans App Store Connect.

## Sécurité et fiabilité

Les alertes LoopFollow et le suivi de glycémie nécessitent une connexion Internet et peuvent être retardés par les limites iOS. Ne pas utiliser cet affichage comme seule méthode de surveillance des hypoglycémies. Ne pas enregistrer l'URL privée Nightscout, les jetons, les données glycémiques ou l'adresse courriel de Julie dans ce dépôt GitHub public.

## Limites de la configuration

- Le fork existe, les workflows d'origine sont conservés, et le mécanisme de distribution externe optionnel est installé.
- Les secrets ne peuvent pas être lus directement ; leur validité doit être démontrée par l'exécution du workflow officiel `1. Validate Secrets`.
- La création de l'application Apple, du groupe externe et l'activation de la variable TestFlight sont **volontairement différées**.
- Les builds GitHub et les mises à jour iOS ne sont pas instantanés ni garantis ; la disponibilité dépend de GitHub Actions, d'Apple et des réglages de TestFlight sur l'iPhone.
