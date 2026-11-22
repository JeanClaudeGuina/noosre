# Retours sur docs/architecture.md

Corrige docs/architecture.md en intégrant les points ci-dessous, puis résume-moi les changements. N'écris toujours pas de code applicatif.

## A. Failles de sécurité (bloquant)

1. **Élévation de privilèges via `profiles.role`** : la politique « UPDATE uniquement son propre profil » permet à n'importe qui de passer son rôle à `admin`. Les politiques RLS ne filtrent pas par colonne. Protéger `role` (et tout champ sensible) par un trigger qui refuse la modification hors service role / admin, ou par des privilèges de colonne (`REVOKE UPDATE (role) ...`).
2. **Même problème sur `speaker_profiles`** : « propriétaire sauf champ `status` » n'est pas faisable en RLS seule. Un locuteur pourrait s'auto-valider ou remettre `cancellation_count` à 0. Champs à verrouiller par trigger : `status`, `rejection_reason`, `cancellation_count`, `approved_at`.
3. **Montants modifiables par l'apprenant** : `bookings` INSERT autorisé à l'apprenant, alors que la ligne contient `amount`, `commission_amount` et `speaker_amount`. Supprimer cet accès : une réservation n'est créée que côté serveur (service role), avec des montants calculés à partir du tarif en base, jamais transmis par le navigateur.
4. **Coordonnées de versement** : ne pas stocker de numéro de compte ou de mobile money en clair dans `payout_details`. Stocker de préférence l'identifiant de bénéficiaire chez le prestataire ; sinon utiliser Supabase Vault. Préciser le mécanisme retenu.
5. **Accès à la salle de visio** : la route qui génère le jeton doit vérifier que l'utilisateur est l'apprenant ou le locuteur de la réservation, que la réservation est confirmée, et que l'heure est dans une fenêtre autour de la séance (ex. 15 min avant jusqu'à la fin).

## B. Erreurs qui casseraient le build ou les réservations

6. **Conflit de routes Next.js** : les groupes `(apprenant)`, `(locuteur)` et `(admin)` définissent tous `/tableau-de-bord`, `/seances` et `/messagerie`. Les groupes entre parenthèses n'apparaissent pas dans l'URL, donc ces routes entrent en conflit et le build échoue. Utiliser des préfixes réels : `/apprenant/...`, `/locuteur/...`, `/admin/...`.
7. **Double réservation pendant le paiement** : deux apprenants peuvent ouvrir Stripe Checkout sur le même créneau. Prévoir une réservation temporaire : la réservation est créée au statut `pending_payment` avec une expiration (ex. 30 min, alignée sur l'expiration de la session Stripe), le créneau passe en `held`, puis le webhook confirme la réservation, ou un cron/l'expiration la libère. Ajouter ces statuts aux enums. Cela règle aussi l'incohérence actuelle : `payments.booking_id` est NOT NULL alors que la réservation n'est créée qu'au webhook.
8. **Chevauchement des créneaux** : ajouter une contrainte d'exclusion PostgreSQL (`btree_gist`, `EXCLUDE USING gist (speaker_id WITH =, tstzrange(start_at, end_at) WITH &&)`) sur les créneaux non annulés, pour qu'un locuteur ne puisse jamais avoir deux séances qui se chevauchent.
9. **Génération des créneaux à partir des règles** : préciser l'algorithme. Comment une plage (ex. 18 h–20 h) est-elle découpée selon les durées 30/45/60 ? Quand le locuteur propose plusieurs durées, comment l'apprenant choisit-il ? Quel horizon (ex. 8 semaines glissantes, régénérées chaque jour) ? Que deviennent les créneaux déjà générés quand une règle change ? La conversion vers l'UTC doit se faire occurrence par occurrence avec le fuseau IANA, pour gérer les changements d'heure.
10. **Lien versements ↔ séances** : `payouts` ne dit pas quelles séances ont été payées, seulement combien (`bookings_count`). Ajouter `bookings.payout_id` (ou une table `payout_items`) pour garantir qu'une séance n'est jamais versée deux fois et que chaque versement est traçable.
11. **Passage à « séance réalisée »** : préciser qui met une réservation au statut `completed` ou `no_show`, et quand (ex. cron après `end_at` + données de `video_sessions`). Les versements en dépendent.

## C. Points fonctionnels à corriger

12. **Visibilité des noms** : avec « chacun lit son propre profil », l'apprenant ne voit pas le nom et la photo du locuteur, et inversement dans « Mes séances » et la messagerie. Prévoir une vue publique limitée (prénom, initiale du nom, photo) pour les fiches validées et entre participants d'une même réservation ou conversation.
13. **Devise des versements** : les locuteurs reçoivent souvent en XOF, XAF, KES, etc. Ajouter `payout_currency`, `local_amount` et `exchange_rate` dans `payouts`. La part locuteur reste calculée en EUR.
14. **Rappels et tâches planifiées** : sur le plan gratuit de Vercel, les crons sont limités (une exécution par jour), donc le rappel 1 h avant n'est pas possible ainsi. Utiliser `pg_cron` dans Supabase, ou programmer les deux e-mails à la réservation avec l'envoi différé de Resend (et les annuler si la séance est annulée).
15. **Suppression de compte (RGPD)** : les données financières (paiements, versements, séances) doivent être conservées pour la comptabilité. Prévoir une anonymisation du profil plutôt qu'une suppression en cascade, et des clés étrangères qui ne suppriment pas l'historique financier.
16. **Création de conversation** : la créer côté serveur uniquement (pas d'INSERT libre), entre un apprenant et un locuteur validé.

## D. Choix techniques

17. **Visio : Daily au lieu de LiveKit.** Les cours sont donnés par des humains, sans agent IA. Daily est plus rapide à intégrer (interface prête à l'emploi), propose un mode audio seul et une qualité adaptative, et son offre gratuite couvre le lancement. Remplacer LiveKit par Daily partout (stack, dossiers `lib/daily.ts`, route `api/daily/room`, table `video_sessions.room_name` = salle Daily).
18. **Versions** : vérifier et utiliser la dernière version stable de Next.js (dans les versions récentes, `middleware.ts` est remplacé par `proxy.ts`). Avec Tailwind 4, la configuration se fait dans le CSS (pas de `tailwind.config.ts`). Pour la PWA, utiliser `@serwist/next`, pas `next-pwa` qui n'est plus maintenu.

## E. Réponses aux zones floues (6 à 12)

6. Admins créés manuellement en base : **oui**.
7. Un seul rôle par utilisateur en v1 : **oui**.
8. Pages des langues : **contenu géré par l'admin** dans la table `languages`, avec la liste des locuteurs ajoutée automatiquement.
9. Messagerie en temps réel avec Supabase Realtime : **oui**.
10. Notifications : **e-mail uniquement** en v1.
11. Annulation par l'admin : **remboursement intégral toujours possible**, quel que soit le délai.
12. Page hors ligne : **simple page statique**, sans cache de données.
