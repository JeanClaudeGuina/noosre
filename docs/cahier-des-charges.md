# Noosre – Cahier des charges (prompt de développement)

## Rôle et contexte

**Rôle :** Tu es un développeur full-stack senior, expert en applications web responsive, en paiements en ligne et en visioconférence temps réel.

**Contexte :** Je crée Noosre, une plateforme web qui met en relation des locuteurs natifs de langues africaines avec des apprenants de la diaspora, pour des cours particuliers en visioconférence. Les apprenants vivent surtout en Europe et en Amérique du Nord ; les locuteurs vivent souvent en Afrique, parfois avec une connexion internet limitée.

## Fonctionnalités par espace

### Partie publique

- Page d'accueil de Noosre : présentation, accès à la recherche, inscription
- Page de présentation pour chaque langue : où elle est parlée, nombre de locuteurs, quelques mots et expressions de base, particularités (tons, alphabet…), et liste des locuteurs qui l'enseignent avec un bouton pour réserver
- Inscription avec choix du rôle (apprenant ou locuteur), connexion, mot de passe oublié
- Pages légales (mentions légales, confidentialité, conditions), avec des textes que je fournirai

### Espace apprenant

- Mon profil : photo, pays de résidence et fuseau horaire, langue(s) à apprendre, niveau actuel (débutant, intermédiaire, avancé), objectifs (parler avec la famille, renouer avec ses racines, voyage…), lien avec la langue (héritage familial ou non), langue de communication (français, anglais)
- Catalogue de recherche des locuteurs, avec filtres par langue, niveau, tarif (fourchette de prix) et disponibilité (jour de la semaine, moment de la journée dans l'heure de l'apprenant)
- Fiche du locuteur : photo, présentation, langue, niveaux acceptés, tarif, prochains créneaux affichés dans l'heure de l'apprenant, bouton pour réserver et bouton pour lui écrire
- Liste de locuteurs favoris
- Paiement par carte bancaire via une page sécurisée Stripe Checkout, puis page de confirmation ; l'e-mail de confirmation part aux deux personnes avec le lien vers la salle, chacun recevant l'horaire dans son heure locale
- Mes séances : à venir et passées, avec le bouton pour rejoindre la visio
- Annulation en un clic depuis « Mes séances », avec une fenêtre de confirmation ; remboursement automatique si l'annulation a lieu plus de 24 h avant la séance, sinon pas de remboursement ; le créneau est libéré et le locuteur prévenu par e-mail
- Suivi de progression : nombre de séances suivies, heures cumulées
- Salle de visio

### Espace locuteur

- Ma fiche, que je remplis et mets à jour moi-même ; elle reste invisible tant que l'administrateur ne l'a pas validée
- Mes disponibilités : je publie et retire mes créneaux, saisis dans mon heure locale ; je peux créer des créneaux répétés chaque semaine (ex. tous les mardis à 18 h), retirer une seule occurrence (vacances, imprévu) ou arrêter la répétition
- Mes séances : à venir et passées, avec le bouton pour rejoindre la visio
- Annulation d'une séance : remboursement intégral et automatique de l'apprenant, prévenu par e-mail, et libération du créneau ; si l'annulation a lieu moins de 24 h avant la séance, les frais de paiement non récupérables sont déduits de mon prochain versement
- Mes revenus : séances réalisées, montants gagnés, versements reçus et à venir
- Mon moyen de versement : compte bancaire ou mobile money (Orange Money, Wave, M-Pesa, MTN Mobile Money…), selon mon pays
- Salle de visio (la même que côté apprenant)

### Commun aux apprenants et aux locuteurs

- Espace personnel : modifier mon profil, ma photo, mon fuseau horaire, ma langue d'interface, mon e-mail et mon mot de passe ; gérer mes préférences de notification ; supprimer mon compte (RGPD)
- Messagerie entre apprenant et locuteur, accessible depuis la fiche du locuteur et depuis chaque séance ; notification par e-mail à chaque nouveau message ; l'administrateur peut consulter une conversation en cas de litige
- Rappels avant chaque séance : par e-mail 24 h avant et 1 h avant, dans l'heure locale de chacun, avec le lien direct vers la salle de visio

### Espace administrateur

- Locuteurs en attente et validés : j'ouvre la fiche, je valide ou je refuse
- Séances payées : montant, commission Noosre et part due à chaque locuteur
- Versements automatiques aux locuteurs après chaque séance réalisée (ou par lot hebdomadaire), vers leur compte bancaire ou mobile money, via un prestataire adapté à l'Afrique (CinetPay ou PayDunya pour l'Afrique francophone, Flutterwave pour l'Afrique anglophone et de l'Est) ; suivi de l'état de chaque versement (en attente, effectué, échoué) et relance possible
- Annulation d'une séance, avec remboursement de l'apprenant et libération du créneau
- Suivi du nombre d'annulations par locuteur, avec possibilité de désactiver sa fiche en cas d'abus
- Réglage du taux de commission

## Exigences transverses

### Gestion des fuseaux horaires

- Détection automatique du fuseau horaire de chaque utilisateur, modifiable dans son espace personnel
- Le locuteur saisit ses créneaux dans son heure locale ; l'apprenant les voit convertis dans la sienne
- Affichage des deux heures lors de la réservation (ex. « 18 h chez vous / 19 h chez votre locuteur »)
- Prise en compte des changements d'heure été/hiver en Europe et en Amérique du Nord, alors que la plupart des pays africains n'en ont pas ; cas particuliers comme le Maroc pendant le Ramadan
- Les créneaux répétés chaque semaine restent à l'heure locale du locuteur, même quand l'heure de l'apprenant change
- Stockage de toutes les dates en UTC, avec les fuseaux horaires au format IANA (ex. Africa/Dakar, Europe/Paris)

### Contraintes techniques

- Approche mobile-first : chaque écran est conçu d'abord pour le téléphone, puis adapté pour la tablette et l'ordinateur (et non l'inverse) ; utilisable de 360 px à 1920 px, sans défilement horizontal
- Application web installable (PWA) : ajout à l'écran d'accueil du téléphone, icône et écran de démarrage aux couleurs de Noosre, ouverture en plein écran comme une vraie appli, et page hors ligne indiquant que la connexion est perdue
- Navigation : barre d'onglets en bas sur mobile, barre latérale sur ordinateur
- Visio intégrée au navigateur, sans installation, optimisée pour les faibles débits (qualité adaptative, mode audio seul possible)
- Pages légères, rapides à charger sur réseau 3G
- Interface en français et en anglais
- Code entièrement en TypeScript en mode strict, avec les types de la base générés automatiquement depuis Supabase ; pas de JavaScript simple
- Mise en forme responsive uniquement avec les classes et points de rupture de Tailwind (sm, md, lg, xl), sans CSS personnalisé sauf nécessité ; composants d'interface (boutons, formulaires, calendriers, fenêtres de confirmation) basés sur shadcn/ui, personnalisés aux couleurs de Noosre ; chaque écran est vérifié sur mobile, tablette et ordinateur
- Stack proposée : Next.js + Tailwind CSS + shadcn/ui, Supabase (base PostgreSQL et authentification), hébergement Vercel, visio WebRTC (LiveKit, Daily ou Jitsi), encaissement Stripe, versements via CinetPay, PayDunya ou Flutterwave, e-mails via Resend ou Brevo. Tu peux proposer mieux en justifiant.
- Paiements et remboursements gérés par webhooks fiables, avec un historique de chaque transaction
- Données personnelles conformes au RGPD

### Design

- Chaleureux et moderne, inspiré de motifs et couleurs africains, sans surcharge
- Mode clair et mode sombre
- Accessible : contrastes suffisants, zones tactiles d'au moins 44 px

### Hors périmètre pour la première version

- Application mobile native
- Cours collectifs
- Avis et notes
- Contenus pédagogiques en autonomie (exercices, quiz)

### Livrable attendu

- Code complet, structuré et commenté
- Schéma de la base de données
- Instructions d'installation et de déploiement

### Méthode

Procède étape par étape. Propose d'abord l'architecture, la liste des écrans et le parcours utilisateur, puis attends ma validation avant d'écrire le code. Développe ensuite un espace à la fois, en commençant par la partie publique. Pour les paiements et les versements, utilise d'abord les modes test des prestataires.
