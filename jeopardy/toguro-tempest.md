# Toguro Tempest — Gitea (web, 200 points)

**Identifiant du challenge** : 266 — **Équipe** : DIVISION_0 — **Statut** : résolue.
**Flag** : `HLB2026{3afa2b9352be617bc637d7753b8bdb83}`

**Cible** : instance `172.28.9.2` hébergeant « Tempest Press » (Flask), une instance
**Gitea 1.21.11** sous `/gitea/`, et un service de déploiement (« deployer ») à l'écoute
sur `127.0.0.1:9000`.

---

## Chaîne complète

### 1. SSRF depuis Tempest Press

La route `POST /verification` valide une URL de source, mais effectue la vérification en
**GET uniquement**, avec les protocoles `http` et `https` seuls autorisés. Cette
contrainte n'empêche pas de joindre les services internes :

- Gitea sur `127.0.0.1:3000` ;
- le deployer sur `127.0.0.1:9000` ;
- la vitrine Flask sur `127.0.0.1:5000`.

### 2. Un dump Gitea accessible sans authentification

`GET /backup/` liste une archive `gitea-dump-<id>.zip`, qui contient :

- `app.ini` et la base `gitea-db.sq3` ;
- `data/deploy-token` (jeton de déploiement) ;
- la **clé privée `id_ed25519` de `k.valenko`**.

### 3. Clonage des dépôts privés

```
git clone http://k.valenko:<jeton-de-déploiement>@172.28.9.2/gitea/k.valenko/press-release.git
```

Le même jeton ouvre le dépôt `infra-tools`, qui livre le code du deployer :

```python
LOCKDOWN_EXPECTED = sha256(f"{FLAG}:lockdown")[:32]

def do_publish():
    # écrit le FLAG en clair dans /var/log/deployer/blocked.log
    # lorsqu'un lockdown valide est vérifié
```

### 4. Emprunter le chemin « lockdown »

1. Signer un fichier `LOCKDOWN` avec la clé récupérée :

   ```bash
   ssh-keygen -Y sign -n lockdown -f k.valenko_id_ed25519 LOCKDOWN
   ```

2. Déposer `LOCKDOWN` et `LOCKDOWN.sig` dans le répertoire publié : un push sur `main`
   déclenche le webhook Gitea vers `127.0.0.1:9000/webhook`, et `publish.sh` s'exécute
   sous l'utilisateur `nobody`.
3. Lire `/var/log/deployer/blocked.log` :

   ```
   Lockdown verified. Operator credential: HLB2026{3afa2b9352be617bc637d7753b8bdb83}
   ```

### 5. Contre-vérification indépendante

Le flag se confirme par deux dérivations croisées, sans toucher à la cible :

- `sha256(FLAG + ":build")[:7]` = `93a25ad`, le numéro de build affiché par la vitrine ;
- `sha256(FLAG + ":webhook")[:32]` = la valeur `webhook_secret` de `config.yml`.

---

## Leçons

- Un dump Gitea accessible sans authentification équivaut à une compromission complète
  de la chaîne de développement : clés privées, jetons, base d'utilisateurs.
- Un script de publication déclenché par webhook est un **vecteur d'exécution
  arbitraire** (ici, une exfiltration via `$PUBLISH_DIR/documents/` était possible).
- Lorsqu'un système dérive ses secrets du flag (`:build`, `:webhook`, `:lockdown`), ces
  dérivations permettent de **valider un flag candidat sans le soumettre** : une
  discipline précieuse pendant une épreuve chronométrée.
