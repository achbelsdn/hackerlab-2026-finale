# Tervalis (Active Directory) — File Vault (254) et Silent Broker (250)

**Catégorie** : Active Directory, forensique — **Équipe** : DIVISION_0
**Instances** : éphémères, accessibles par tunnel WireGuard sur des sous-réseaux
`172.28.x.0/24` (contrôleur de domaine `172.28.3.3`, serveur de fichiers FS01
`172.28.3.4`, serveur applicatif APP01, selon le déploiement).

---

## File Vault (Q254) — résolue

**Flag** : `HLB2026{GMSA_MANAGED_PASSWORD_BACKUP_OPS_COMPROMISED}`

### Chaîne de résolution

1. **Énumération Active Directory** (bibliothèque ldap3, liaison SIMPLE avec UPN) :
   cartographie complète des ACL du déploiement.
2. Découverte d'un compte **gMSA** (`svc_gmsa_backup`) : la lecture de son mot de passe
   managé (attribut `msDS-ManagedPassword`) est accordée au principal compromis — c'est
   précisément le rôle d'un compte de service managé par groupe.
3. Ce compte dispose d'un droit de lecture sur le **partage caché `Backup-Archive`** de
   FS01, invisible avec les autres identités disponibles (`svc_mssql` ne voit qu'`IPC$`,
   `svc_backup` lit `IT-Share`).
4. Lecture de `flag-i210.txt` (54 octets) sur FS01 :

   ```
   HLB2026{GMSA_MANAGED_PASSWORD_BACKUP_OPS_COMPROMISED}
   ```

### Leurre documenté

Un « validateur » interne expose `/status`, `/verify/rbcd`,
`/verify/shadow-credentials` et `/verify/dcsync`, et délivre le marqueur
`HLB2026{RBCD_RESOURCE_DELEGATION_APP01_EXPLOITED}`. Ce marqueur n'est **pas** le flag
du challenge : c'est une confirmation de compromission, pas une réponse attendue.
Deux observations l'expliquent :

- l'attribut `msDS-KeyCredentialLink` est **absent** de l'annuaire : la technique des
  shadow credentials est matériellement impossible, l'endpoint correspondant est un
  leurre ;
- FS01 ne traite pas Kerberos : la délégation contrainte vantée par le scénario est
  inutilisable en pratique.

---

## Silent Broker (Q250) — marqueur validé

**Flag** : `HLB2026{INTERNAL_SERVICE_RCE_CASE_RELAY_COMPROMISED}`

### Chaîne de résolution

1. **Exécution de code à distance via OpenWire** sur Apache ActiveMQ (famille de
   vulnérabilités connues du protocole), aboutissant à un shell sous l'identifiant 0.
2. Lecture de `/opt/activemq/data/flag.txt`.
3. Ce marqueur correspond à la question Silent Broker. Le même fichier, lu sur
   d'autres instances du même scénario (Sealed Order, File Vault), ne constitue pas une
   réponse : un marqueur de compromission n'a de valeur que pour la question qui le
   demande.

---

## Sealed Order et Deep Archive (Q255) — non résolues

- Accès obtenus et revérifiés : `svc_gmsa_backup` lit `Backup-Archive`
  (`tims_comm_intercepts.txt`, `flag-i210.txt`), `svc_mssql` lit `Classified` sur le
  contrôleur de domaine, `svc_backup` lit `IT-Share` — mais aucun des deux flags
  recherchés ne figurait dans ces documents.
- Le portail interne (file d'attente `/queue/kappa`, protocole STOMP) confirme
  l'enregistrement d'une demande `release-order` (accusé `TR-0A8BCA`, statut
  `Transmis`), sans jamais retourner le document attendu.
- **Diagnostic** : les deux flags exigent une identité à privilèges supérieurs (compte
  Administrator local de FS01 ou compte dédié), qui n'a pas été obtenue avant
  l'expiration des instances — le quota de trois instances simultanées ayant limité les
  redéploiements.
