# The Fall Of JANUS (forensics, 4 × 100 points)

**Identifiants des challenges** : Ransomless (Q1), The Trojan Horse (Q2), New Blood
(Q3), Royal Heist (Q4) — **Équipe** : DIVISION_0 — **Statut** : 3/4 résolues.
**Flag Q1** : `HLB2026{wh1t3_0n_wh1t3_1s_n0t_h1dd3n}`

---

## Énoncé

> Le portail du tribunal de Tervalis constitue une infrastructure critique
> centralisant les pièces à conviction, les scellés numériques et les décisions
> judiciaires relatives à plusieurs affaires sensibles. Le 8 novembre 2026, les
> équipes techniques constatent l'indisponibilité de cette infrastructure. Les
> premières investigations révèlent que plusieurs fichiers sur le serveur critique
> ont été chiffrés et renommés avec une extension inconnue.
>
> Ces éléments laissent supposer que l'attaque ne visait pas uniquement à perturber
> le fonctionnement du portail mais également à dissimuler une manipulation de
> dossiers judiciaires.
>
> Dans le cadre de la réponse à incident, une collecte forensique du serveur a été
> réalisée et l'ensemble des artefacts collectés vous est désormais mis à
> disposition. Votre mission consiste à exploiter ces éléments afin de reconstituer
> la chronologie de l'intrusion, de remonter jusqu'à son identité réelle et de
> récupérer les éléments de preuve chiffrés.

**Artéfacts** : archive Mega de 1 995 400 700 octets (~1,9 Go), à traiter sur une
machine disposant d'espace (nous avons utilisé une VM Google Colab).

---

## Q1 — Ransomless (100 points) : récupérer le fichier d'importance systémique

> La hiérarchie informe qu'un fichier d'importance systémique se trouvait sur le
> serveur compromis lors de l'incident. Les données du serveur ont été intégralement
> altérées par un ransomware. Récupérez d'une façon ou d'une autre le contenu de ce
> fichier.

### 1. Récupération et déchiffrement de l'archive Mega

La clé Mega n'est pas dans l'URL lisible : la première dérivation tentée produisait
des octets aléatoires. Le point de contrôle fiable est le **nom de fichier chiffré**
`2026-11-08_srvdump.zip` — une dérivation correcte doit reproduire ce nom. Une fois
la dérivation corrigée à partir de l'attribut de nom, le transfert en flux sur Colab
aboutit et l'archive s'ouvre.

### 2. Inventaire du dump

Le contenu est un **dump Windows NTFS** (24 036 entrées, ~2,9 Go décompressés) avec
des fichiers renommés en `.enc` et les journaux de copie du ransomware. Le triage
brut mélange des milliers de fichiers système ; le déclic vient d'un détail
d'infrastructure : la machine est un **serveur XAMPP** (pas IIS). Le périmètre utile
se réduit alors à `xampp/htdocs`, les données MySQL et les journaux associés —
là où un « fichier d'importance systémique » d'un portail judiciaire a des chances
de vivre.

### 3. Le ransomware est livré avec la victime

Artefact inattendu et **non chiffré** : `Users/backup/AppData/calc.exe`. L'analyse
statique (sans jamais l'exécuter) montre que ses symboles exposent
`derive_key`, `ctr_crypt` et `xtea_decrypt_block` :

- le chiffrement n'est **pas** AES-GCM mais **XTEA en mode CTR** ;
- une graine lisible est embarquée : `LYNX_FNS_ASHEN_TRIBUNAL_2026` ;
- la clé est obfusquée dans le binaire — on l'extrait avec les paramètres exacts du
  compteur CTR pour réimplémenter le déchiffrement en Python.

Premier essai : le « verdict » déchiffré est un PDF valide… sans flag. Il faut donc
dérouler le même déchiffrement sur les **28 fichiers de l'application XAMPP** et
chercher la pièce réellement porteuse de la preuve, sans présumer que le premier
document plausible était la cible.

### 4. Le flag : texte blanc sur blanc

La pièce visée est `requisitoire_ministere_public.pdf` (le réquisitoire du ministère
public). Une fois déchiffré, son texte apparent ne contient rien — mais le PDF
comporte du **texte blanc sur fond blanc**. La chaîne cachée est en **Base64
inversé** ; décodée, elle donne :

```
HLB2026{wh1t3_0n_wh1t3_1s_n0t_h1dd3n}
```

Le nom du challenge est littéralement la leçon : *white on white is not hidden*.

---

## Q2 — The Trojan Horse (100 points) : résolue par l'équipe

> Un fichier malveillant a été introduit sur le portail puis exploité. Identifiez :
> le nom du fichier malveillant ; l'adresse IP source de l'attaquant ; la date et
> l'heure de la dernière authentification réussie sur la plateforme.

Réponse validée par DIVISION_0 (l'analyse du dump — journaux web XAMPP, journaux
d'authentification Windows — permet de recouper les trois éléments demandés).

## Q3 — New Blood (100 points) : résolue par l'équipe

> Après avoir obtenu un premier accès au système, un compte utilisateur a été créé
> afin de disposer d'un accès indépendant au serveur. Retrouvez : le nom du compte
> créé ; l'heure de création ; l'heure de sa première ouverture de session.

Réponse validée par DIVISION_0 (hive SAM / journaux d'événements du dump).

## Q4 — Royal Heist (100 points) : non résolue

> L'attaquant a exporté la base de données des verdicts, puis l'a transférée vers
> une infrastructure distante. Déterminez : l'heure du lancement de l'exportation ;
> l'adresse IP et le port de destination utilisés pour le transfert.

**État de l'investigation** :

- La chronologie PowerShell établit une exportation vers `verdicts_dump.sql`
  (lancement de `mysqldump`), suivie d'un **POST vers `192.168.230.130:8700`**.
- Le point resté ouvert est la **distinction entre l'heure des commandes de
  transfert et l'heure réelle du lancement de l'export** : le candidat devait être
  recoupé entre le Prefetch de `mysqldump`, l'horodatage du fichier exporté et les
  traces réseau.
- Aucune soumission n'a été faite sans cet appui ; la question est restée ouverte.

---

## Leçons

- **Le nom de fichier est un oracle** : pour Mega, la dérivation de clé se vérifie
  sur le nom chiffré du fichier. Un contrôle indépendant évite d'analyser des
  octets aléatoires pendant une heure.
- **Le ransomware voyage avec ses victimes** : un binaire non chiffré (`calc.exe`)
  traînant dans le profil de sauvegarde contenait tout le schéma (XTEA-CTR, graine,
  clé obfusquée). L'analyse statique des symboles (`derive_key`, `ctr_crypt`,
  `xtea_decrypt_block`) oriente immédiatement la réimplémentation.
- **Ne pas s'arrêter au premier document plausible** : le premier PDF déchiffré
  était un leurre légitime. Dérouler le déchiffrement sur tout le périmètre métier
  (les 28 fichiers XAMPP) avant de conclure.
- **Le texte « invisible » est un classique** : blanc sur blanc, calques masqués,
  texte hors page — extraire le texte avec position/couleur (pdftotext, pdfplumber)
  plutôt que lire la page rendue.
- **Une heure d'export n'est pas une heure de commande** : en forensique Windows,
  Prefetch, horodatage $STANDARD_INFORMATION/$FILE_NAME et journaux applicatifs se
  recoupent ; la question Royal Heist demandait précisément cette rigueur.
