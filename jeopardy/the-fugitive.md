# The Fugitive (misc / investigation, 200 points)

**Identifiant du challenge** : mission `the_fugitive` — **Équipe** : DIVISION_0 —
**Statut** : résolue (6/6 questions validées, flag final délivré par la plateforme).
**Flag** : `HLB2026{23c99ee2b06e3deb01ea46e24b10f880}`

---

## Énoncé

> Une opération d'exfiltration clandestine a été détectée dans les eaux nordiques de
> la République de Skara. Les services de renseignement ont intercepté plusieurs
> documents classifiés et saisi les systèmes de la NordMar — Nordic Maritime
> Authority.
>
> **Mission** : exploiter les données AIS, les registres maritimes, les registres
> commerciaux et les documents saisis pour reconstituer l'intégralité de
> l'opération.

- **Documents saisis** (`confidential.zip`) : DOC-01 « Note Coldwater »,
  DOC-05 « Avis de changement d'équipage », DOC-06 « Livre de codes BRN ».
- **Plateforme d'investigation** (instance VPN `172.28.22.2:80`) : registres
  maritimes, explorateur de trafic AIS, registres commerciaux, bulletin portuaire,
  et une section « Investigation » où soumettre les réponses aux 6 questions.
  Le flag final n'est révélé qu'après validation des six réponses.

## Méthode

Ce challenge est une vraie enquête OSINT/forensique « fermée » : chaque réponse se
dérive par recoupement entre les documents saisis et les données de la plateforme.

1. **Extraction des trois PDF saisis** et lecture des codes/identités qu'ils
   contiennent (le DOC-06 est un livre de codes : il fait le pont entre les
   abréviations des autres documents et les lieux réels).
2. **Collecte des données de la plateforme** : `data/registry.json` (registre
   maritime), `data/ais_positions.json` (positions AIS), `data/ports.geojson`,
   `docs/DOC-04-port-bulletin.html` (bulletin portuaire APVD).
3. **Corrélations horaires** : les horaires des documents sont en heure locale
   Nordheim (UTC+2), les positions AIS en UTC. Le navire suspecté est à quai
   Terminal B / berth B-03 avant son départ ; le départ AIS à 05:20 UTC correspond
   exactement au 07:20 local du DOC-05.
4. **Soumission des six réponses** via `POST /api/submit` — chacune retourne
   `correct: true` ; `/api/status` confirme `all_done=true`,
   `total_validated=6`, et l'API révèle le flag après la sixième validation.

## Les six réponses validées

| # | Question | Réponse validée | Preuve principale |
|---|---|---|---|
| Q1 | Ghost Passenger — qui est l'agent infiltré ? | `HLB2026{Mikkel_Strand}` | DOC-01 désigne l'asset `MS` ; DOC-06 associe `MS` à Mikkel Strand. DOC-05 donne Erik Moller comme identité de couverture embarquée. |
| Q2 | Dead Reckoning — itinéraire de l'opération | `HLB2026{Nordheim_Kalport_Porto_Valdis}` | DOC-01 : route ALPHA → BRAVO → CHARLIE ; DOC-06 mappe les zones sur Nordheim, Kalport, Porto Valdis. |
| Q3 | Shadow Hull — navire et horaire d'embarquement | `HLB2026{SLEIPNIR}` | DOC-05 : embarquement Terminal B le 2026-11-06 à 06:00 locale, départ 07:20 locale. AIS : le SLEIPNIR quitte B-03 à 05:20 UTC (= 07:20 UTC+2). |
| Q4 | Hull Number — IMO du navire | `HLB2026{9847322}` | Registre maritime `data/registry.json` : IMO du SLEIPNIR = 9847322. |
| Q5 | Shell Company — opérateur du navire | `HLB2026{NordMar_Shipping_AS}` | Registre maritime : l'opérateur du SLEIPNIR est NordMar Shipping AS. |
| Q6 | Last Port of Call — escale prévue | `HLB2026{N-07_Terminal_Nord_Porto_Valdis}` | DOC-04, bulletin APVD N° 2026-312 : SLEIPNIR planifié le 2026-11-09 à Porto Valdis, Terminal Nord, poste N-07, en provenance de Kalport. |

## Flag

```
HLB2026{23c99ee2b06e3deb01ea46e24b10f880}
```

## Leçons

- **Le livre de codes est la clé de voûte** : sans DOC-06, les zones ALPHA/BRAVO/
  CHARLIE de la note Coldwater restent abstraites. Toujours chercher le document
  « dictionnaire » avant d'interpréter les autres.
- **Les fuseaux horaires sont un piège classique** : la corrélation AIS/documents
  ne colle que si l'on convertit 07:20 UTC+2 en 05:20 UTC. C'est ce recoupement
  qui confirme le navire, pas l'intuition.
- **Un flag révélé par la plateforme se mérite** : les six réponses doivent être
  exactes ; une seule dérivation bancale bloque la révélation finale. Soumettre
  chaque réponse avec sa preuve écrite évite les allers-retours.
