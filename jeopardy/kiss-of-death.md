# Kiss of Death — mission téléphone / forensics (Zebulon, Papyrus, Dreamer…)

**Catégorie** : misc / forensics / OSINT — **Équipe** : DIVISION_0
**Statut** : plusieurs questions résolues, format des flags = placeholders littéraux du tag

Leçon structurante de cette mission : **le tag `flag:HLB2026{...}` du panneau mission est
le placeholder du format EXACT** (ex. `flag:HLB2026{ApplicationName:Code}` → le flag final
est littéralement `ApplicationName:Code` avec underscores pour les espaces).

---

## Zebulon — Q5 (300 pts)

**Flag** : `HLB2026{Boussole:114° ESE}`

### Résolution

1. Dump ADB d'un téléphone (207.180.205.54:5555, root) : FS minimal — `gesture.key`,
   `locksettings.db`, `users/0.xml` (`Dritan`), app `com.coffreplus.vault` (vide),
   `/data/local/tmp/notes.txt` : *« Rappel: verrouiller le tel. Schema change tous les
   mois. »*
2. Le profil leurre liste des applications dans un ordre codé (`TMGCHCBP`) ; l'app
   **Boussole** affiche un azimut.
3. Le serveur du challenge expose `/api/unlock`, `/api/real`, `/api/file` — aucun
   endpoint de validation d'un PIN Coffre+ (26 endpoints sondés → `not found`).
4. Le flag attendu est la chaîne **telle qu'affichée par l'UI** de la Boussole :
   nombre + unité + cardinal → `HLB2026{Boussole:114° ESE}` (accepté après ~360 essais ;
   les variantes `:114`, `:114°`, `:0114`, `:3:48`… étaient toutes refusées).

**Leçon** : le « Code » d'un flag `{ApplicationName:Code}` peut être la chaîne TELLE
QU'AFFICHÉE par l'interface (valeur + unité + cardinal), pas la valeur nue.

---

## Papyrus — Q6 (300 pts)

**Flag** : `HLB2026{CERT-TVS-2026-IR-001}`

### Résolution

Corpus de 6 incidents attribués à l'APT **Lynx Varelien (VARELID)** / **VARELCORP
Solutions** (contact `mateus.varga@varelcorp.var`) :

- **Jon Varek** (ing. AquaTerv) = source interne de la fuite (métadonnées DOCX, DarkVault) ;
- `j.aterbe@bct.gov.trv` victime de phishing `.docm` via le domaine homographe
  `xn--varelcrp-s4a.var` ;
- Marc de Lacroix (BCT) victime ; Anastas Volkov (Bank One Limited) destinataire.

Le flag = l'identifiant du rapport d'incident de référence : `HLB2026{CERT-TVS-2026-IR-001}`.

**Bonus P5** : `HLB2026{3p4n3t_fl0w_c0mpr0m1s3d}` — clé EPANET utilisée pour
l'Operation Tervalis (SCADA).

---

## Dreamer — Q7 (50 pts)

**Flag** : `HLB2026{25/08/2026 09:46 31415926}`

### Résolution

1. Points de traçance **imprimante** (tracking dots DEDA — yellow dots) sur un document
   imprimé : décodage via `libdeda` (paquet PyPI `deda`, github dfd-tud/deda).
2. Extraction : date/heure d'impression + numéro de série/identifiant de la machine.
3. Format = `date heure identifiant` tels qu'imprimés.

**Pièges** :
- Le flag a été **rejeté puis accepté ~2 h plus tard** : l'auteur corrigeait le flag
  plateforme en même temps qu'il publiait les hints (« Yellow dots »). Ne jamais conclure
  trop vite sur un rejet.
- Un script de soumission avait `chal=274` en dur → un batch entier a été testé sur la
  mauvaise question (`already_solved` trompeur). Toujours paramétrer l'ID.

---

## Poseidon — Q8 (500 pts)

Validée (solve utilisateur) — chaîne non re-dérivable depuis nos notes.

## Autres questions de la mission

- Q3 (tag `flag:HLB2026{password_NbrObjetsScelles}` / `flag:HLB2026{password}`) :
  deux formats acceptés par l'auteur.
- Le CSS de `served.html` contenait un écran **VAULT complet jamais utilisé** par le JS
  (`.vaultwrap/.pindisp/.keys/.secret/.flag`) : l'écran PIN Coffre+ qui affiche le flag
  final était prévu mais non câblé — indice d'un challenge possiblement incomplet.
