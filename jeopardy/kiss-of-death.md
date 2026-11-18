# Kiss of Death — mission téléphone et forensique (Zebulon, Papyrus, Dreamer…)

**Catégorie** : misc, forensique, OSINT — **Équipe** : DIVISION_0 — **Statut** :
plusieurs questions résolues.

La leçon structurante de cette mission tient en une phrase : **la balise
`flag:HLB2026{...}` affichée sur le panneau d'une mission est le gabarit exact du flag
attendu** (par exemple `flag:HLB2026{ApplicationName:Code}` annonce littéralement
`ApplicationName:Code`, avec des underscores à la place des espaces).

---

## Zebulon — Q5 (300 points)

**Flag** : `HLB2026{Boussole:114° ESE}`

### Résolution

1. Extraction ADB d'un téléphone (`207.180.205.54:5555`, accès root) : système de
   fichiers minimal — `gesture.key`, `locksettings.db`, `users/0.xml` (utilisateur
   « Dritan »), application `com.coffreplus.vault` vide, et
   `/data/local/tmp/notes.txt` : *« Rappel : verrouiller le tel. Schema change tous les
   mois. »*
2. Le profil leurre liste des applications dans un ordre codé (`TMGCHCBP`) ; parmi elles,
   l'application **Boussole** affiche un azimut.
3. Le serveur du challenge n'expose que `/api/unlock`, `/api/real` et `/api/file` —
   aucun point d'entrée ne valide un code de coffre (26 chemins supplémentaires sondés,
   tous en `not found`).
4. Le flag attendu est la chaîne **telle qu'affichée par l'interface** de la Boussole :
   valeur, unité et point cardinal, soit `HLB2026{Boussole:114° ESE}`. Toutes les
   variantes partielles (`:114`, `:114°`, `:0114`, `:3:48`…) étaient refusées.

**Leçon** : le « code » d'un flag de la forme `{ApplicationName:Code}` peut être la
chaîne telle qu'elle s'affiche à l'écran (valeur, unité, cardinal), et non la valeur
nue. Observer l'interface avant de normaliser les données.

---

## Papyrus — Q6 (300 points)

**Flag** : `HLB2026{CERT-TVS-2026-IR-001}`

### Résolution

Le corpus décrit six incidents attribués à l'APT **Lynx Varelien (VARELID)** et à la
société **VARELCORP Solutions** (contact `mateus.varga@varelcorp.var`) :

- **Jon Varek** (ingénieur chez AquaTerv) est la source interne de la fuite
  (métadonnées de documents DOCX, plateforme DarkVault) ;
- `j.aterbe@bct.gov.trv` est victime d'un hameçonnage par pièce jointe `.docm`
  provenant du domaine homoglyphe `xn--varelcrp-s4a.var` ;
- Marc de Lacroix (BCT) est également victime ; Anastas Volkov (Bank One Limited)
  figure parmi les destinataires.

Le flag est l'identifiant du rapport d'incident de référence :
`HLB2026{CERT-TVS-2026-IR-001}`.

**Bonus P5** : `HLB2026{3p4n3t_fl0w_c0mpr0m1s3d}` — clé EPANET utilisée pour
l'opération Tervalis (systèmes de contrôle industriels).

---

## Dreamer — Q7 (50 points)

**Flag** : `HLB2026{25/08/2026 09:46 31415926}`

### Résolution

1. Le document imprimé porte des **points de traçance d'imprimante** (marqueurs jaunes,
  « tracking dots ») : leur décodage passe par l'outil `libdeda` (paquet PyPI `deda`,
  dépôt dfd-tud/deda).
2. Le décodage restitue la date et l'heure d'impression ainsi que l'identifiant de la
   machine.
3. Le flag suit le format `date heure identifiant`, tel qu'imprimé.

**Deux pièges à retenir** :

- Le flag a été **corrigé par l'organisateur en cours d'épreuve**, en même temps que la
  publication des indices (« Yellow dots ») : un refus initial ne signe pas une réponse
  fausse lorsque la piste est solide.
- Un outil interne avait l'identifiant de challenge codé en dur sur la mauvaise question,
  ce qui a faussé un lot entier de vérifications. Toujours paramétrer l'identifiant de
  challenge dans ses outils, et relire l'énoncé ciblé avant toute conclusion.

---

## Poseidon — Q8 (500 points)

Question validée par un coéquipier ; la chaîne détaillée sera complétée à partir de ses
notes.

---

## Autres éléments de la mission

- La question 3 admettait **deux formats** de flag (mot de passe seul, ou mot de passe
  suivi du nombre d'objets scellés) — un cas rare où l'auteur accepte deux réponses.
- La feuille de style de `served.html` contenait un écran **VAULT complet jamais
  utilisé** par le code JavaScript (`.vaultwrap`, `.pindisp`, `.keys`, `.secret`,
  `.flag`) : l'écran de code du Coffre+ censé afficher le flag final était prévu mais
  non câblé — un indice précieux sur l'état d'achèvement du challenge.
