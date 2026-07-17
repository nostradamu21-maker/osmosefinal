# Note technique — Synchronisation du calendrier avec SuperHote (iCal)

À l'attention du développeur / intégrateur du site Signatures.

## Contexte

La page **« Réservation - Signatures »** affiche un calendrier de disponibilités
maison (deux mois côte à côte, code couleur, sélection de séjour) pour 4 logements.
Le code sait déjà lire les plannings SuperHote au format **iCal** et marquer les
nuits réservées. Il ne reste qu'**une seule chose à régler : le CORS.**

## Le problème : CORS

Les liens d'export iCal de SuperHote (`https://app.superhote.com/export-ics/…`)
**n'autorisent pas la lecture directe depuis un navigateur** : un `fetch()` depuis
la page renvoie `Failed to fetch` (pas d'en-tête `Access-Control-Allow-Origin`).
C'est une protection normale, non contournable côté front.

Tant que ce n'est pas réglé, le calendrier retombe automatiquement sur des
disponibilités d'**exemple** (aucune erreur visible, aucune page cassée).

## La solution : un proxy même-origine

Servez chaque iCal **depuis votre propre domaine** (le serveur fait la requête à
SuperHote, lui n'est pas soumis au CORS), puis pointez le calendrier vers ces URLs.

### Correspondance des 4 logements

| Logement              | Export SuperHote                                   |
|-----------------------|----------------------------------------------------|
| Osmose                | `https://app.superhote.com/export-ics/U3LzTuaYGs`  |
| Le Clos de Saint Jo   | `https://app.superhote.com/export-ics/NpJGoWU1f6`  |
| Into the Wine         | `https://app.superhote.com/export-ics/vCmXfzwTOH`  |
| Mid Century           | `https://app.superhote.com/export-ics/MQ24MuoReR`  |

Objectif : exposer par exemple `https://votre-site.com/ics/osmose.ics`, etc.

### Exemple A — Nginx (reverse proxy)

```nginx
location /ics/osmose.ics            { proxy_pass https://app.superhote.com/export-ics/U3LzTuaYGs; }
location /ics/le-clos-de-saint-jo.ics { proxy_pass https://app.superhote.com/export-ics/NpJGoWU1f6; }
location /ics/into-the-wine.ics     { proxy_pass https://app.superhote.com/export-ics/vCmXfzwTOH; }
location /ics/mid-century.ics       { proxy_pass https://app.superhote.com/export-ics/MQ24MuoReR; }
```

### Exemple B — Cloudflare Worker

```js
const MAP = {
  'osmose': 'U3LzTuaYGs',
  'le-clos-de-saint-jo': 'NpJGoWU1f6',
  'into-the-wine': 'vCmXfzwTOH',
  'mid-century': 'MQ24MuoReR',
};
export default {
  async fetch(req) {
    const id = new URL(req.url).pathname.split('/').pop().replace('.ics', '');
    if (!MAP[id]) return new Response('Not found', { status: 404 });
    const r = await fetch('https://app.superhote.com/export-ics/' + MAP[id]);
    return new Response(await r.text(), {
      headers: { 'content-type': 'text/calendar', 'access-control-allow-origin': '*' },
    });
  },
};
```

### Exemple C — PHP (un seul fichier `ics.php`)

```php
<?php
$map = [
  'osmose' => 'U3LzTuaYGs',
  'le-clos-de-saint-jo' => 'NpJGoWU1f6',
  'into-the-wine' => 'vCmXfzwTOH',
  'mid-century' => 'MQ24MuoReR',
];
$id = $_GET['id'] ?? '';
if (!isset($map[$id])) { http_response_code(404); exit; }
header('Content-Type: text/calendar');
echo file_get_contents('https://app.superhote.com/export-ics/' . $map[$id]);
```

> Astuce : mettez une mise en cache de 5–15 min côté proxy pour ne pas appeler
> SuperHote à chaque visite.

## Le branchement final (1 ligne par logement)

Dans le fichier **`Réservation - Signatures.dc.html`**, repérez la liste
`LOGEMENTS` et remplacez chaque `ical` par l'URL de votre proxy :

```js
LOGEMENTS = [
  { name: 'Osmose',              id: 'osmose',              ical: '/ics/osmose.ics',              seed: 0 },
  { name: 'Le Clos de Saint Jo', id: 'le-clos-de-saint-jo', ical: '/ics/le-clos-de-saint-jo.ics', seed: 1 },
  { name: 'Into the Wine',       id: 'into-the-wine',       ical: '/ics/into-the-wine.ics',       seed: 2 },
  { name: 'Mid Century',         id: 'mid-century',         ical: '/ics/mid-century.ics',         seed: 3 },
];
```

C'est tout. Au chargement, le calendrier lit ces fichiers, parse les événements
`BEGIN:VEVENT … DTSTART/DTEND … END:VEVENT`, et marque comme **« Déjà réservé »**
toutes les nuits de `DTSTART` à `DTEND` (exclu). Le `seed` et les dispos d'exemple
deviennent alors inutiles.

## Le bouton « Réserver »

Au clic, le client est envoyé sur SuperHote avec dates + voyageurs + logement :

```
https://app.superhote.com/#/get-available-rentals/[ID]?rental=osmose&checkin=2026-07-14&checkout=2026-07-17&adults=2&children=0
```

Les noms de paramètres (`checkin`, `checkout`, `adults`, `children`, `rental`) sont
modifiables au même endroit dans la fonction `bookingUrl()`. **À confirmer avec
SuperHote** que ces noms pré-remplissent bien leur moteur ; sinon, ajustez-les.
