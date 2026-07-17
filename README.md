# Signatures — Maison d'adresses à Dijon

Site vitrine statique (HTML / CSS / JS, sans framework ni build) reproduisant le design
et les fonctionnalités des maquettes Claude Design « sitepaul ».

## Pages

| Fichier | Page |
|---|---|
| `index.html` | Accueil (héros, filtre « Entrer par l'envie », la collection, promesse, journal) |
| `nos-adresses.html` | La collection complète |
| `reservation.html` | Réservation directe — moteur SuperHote (iframe) |
| `osmose.html` | Osmose — Suite & Spa (thème Signature Dark) |
| `le-clos-de-saint-jo.html` | Le Clos de Saint Jo (Signature Light, accent pierre/bleu) |
| `into-the-wine.html` | Into the Wine (Signature Light, accent bordeaux) |
| `mid-century.html` | Mid Century (Signature Light, accent terracotta) |

Les images sont dans `assets/img/` (extraites des maquettes, format WebP).

## Fonctionnalités

- **Filtre par envie** sur l'accueil : un clic sur une carte (« À deux », « Bien-être »…)
  filtre les adresses correspondantes ; re-clic pour tout réafficher.
- **En-tête de l'accueil** : transparent sur le héros, devient opaque au défilement.
- **Modale de réservation** sur chaque page logement : « Vérifier les disponibilités »
  ouvre le moteur SuperHote du logement dans une fenêtre modale (fermeture par ✕,
  clic hors de la fenêtre ou touche Échap). L'iframe n'est chargée qu'à l'ouverture.
- **Page réservation** : moteur SuperHote multi-logements intégré en iframe.
- **Responsive** : menu burger sous 900 px, grilles et typographies adaptées
  (tablette et mobile), modale plein écran sur mobile.

## À compléter avant la mise en ligne

1. **Photo du héros de l'accueil** : l'image d'origine (visuel généré « Dijon ») n'était
   pas incluse dans l'export. Remplacez `assets/img/hero-accueil.webp` par votre visuel
   (actuellement une photo du Mid Century en attendant).
2. **Clé SuperHote du Clos de Saint Jo** : dans `le-clos-de-saint-jo.html`, l'URL de la
   modale contient encore `PROPERTY_KEY_CLOS_SAINT_JO_A_COMPLETER` (comme dans la
   maquette). Remplacez-la par la vraie clé `propertyKey…` du logement.
3. **Liens morts** (`href="#"`) : Journal, Destination, mentions légales, CGV,
   réseaux sociaux — à brancher quand les pages existeront.
4. **Calendrier iCal (optionnel)** : si vous souhaitez un jour afficher les
   disponibilités sans iframe, voir `docs/note-technique-calendrier-superhote.md`
   (proxy iCal même-origine à mettre en place côté serveur).

## Déploiement

Aucune compilation : servez le dossier tel quel (GitHub Pages, Netlify, Nginx…).
