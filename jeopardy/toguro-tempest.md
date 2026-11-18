# Toguro Tempest — Gitea (web, 200 pts)

**Challenge ID** : 266 — **Équipe** : DIVISION_0 — **Statut** : résolue
**Flag** : `HLB2026{3afa2b9352be617bc637d7753b8bdb83}`

**Cible** : instance `172.28.9.2` — « Tempest Press » (Flask) + **Gitea 1.21.11** sous
`/gitea/` + un service « deployer » sur `127.0.0.1:9000`.

---

## Chaîne complète

### 1. SSRF depuis Tempest Press

`POST /verification` valide une URL source mais **en GET uniquement, http/https
uniquement** → on atteint les services internes :

- Gitea `127.0.0.1:3000`
- deployer `127.0.0.1:9000`
- vitrine Flask `127.0.0.1:5000`

### 2. Dump Gitea exposé sans authentification

`GET /backup/` liste un dump : `gitea-dump-<id>.zip` contient :

- `app.ini`, `gitea-db.sq3` ;
- `data/deploy-token` (jeton de déploiement) ;
- la **clé privée `id_ed25519` de `k.valenko`**.

### 3. Clone des dépôts privés

```
git clone http://k.valenko:<deploy-token>@172.28.9.2/gitea/k.valenko/press-release.git
```

Le même token ouvre aussi `infra-tools`, qui contient le code du deployer :

```python
LOCKDOWN_EXPECTED = sha256(f"{FLAG}:lockdown")[:32]

def do_publish():
    # écrit le FLAG en clair dans /var/log/deployer/blocked.log
    # quand un lockdown valide est vérifié
```

### 4. Déclencher le chemin « lockdown »

1. Créer un fichier `LOCKDOWN` signé :

   ```bash
   ssh-keygen -Y sign -n lockdown -f k.valenko_id_ed25519 LOCKDOWN
   ```

2. Déposer `LOCKDOWN` + `LOCKDOWN.sig` dans `/www/publish/` (push sur `main` →
   webhook Gitea `127.0.0.1:9000/webhook` → `publish.sh` exécuté en `nobody`).
3. Lire `/var/log/deployer/blocked.log` :

   ```
   Lockdown verified. Operator credential: HLB2026{3afa2b9352be617bc637d7753b8bdb83}
   ```

### 5. Double vérification indépendante

Le flag se confirme par deux dérivations croisées :

- `sha256(FLAG + ":build")[:7]` = `93a25ad` = build affiché par la vitrine ;
- `sha256(FLAG + ":webhook")[:32]` = `webhook_secret` de `config.yml`.

---

## Leçons

- Un dump Gitea accessible sans auth = clés privées + tokens + base utilisateurs : c'est
  la compromise totale du SDLC.
- `publish.sh` déclenché par webhook = **vecteur d'exécution arbitraire** (exfiltration
  via `$PUBLISH_DIR/documents/`).
- Les secrets dérivés du flag (`:build`, `:webhook`, `:lockdown`) permettent de
  **vérifier un flag candidat sans le soumettre**.
