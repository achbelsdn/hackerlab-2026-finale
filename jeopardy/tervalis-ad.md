# Tervalis (AD) — File Vault (254) & Silent Broker (250)

**Catégorie** : Active Directory / forensics — **Équipe** : DIVISION_0
**Instances** : éphémères WireGuard, sousnets `172.28.x.0/24` (DC `172.28.3.3`,
FS01 `172.28.3.4`, APP01… selon déploiement)

---

## File Vault (Q254) — RÉSOLU

**Flag** : `HLB2026{GMSA_MANAGED_PASSWORD_BACKUP_OPS_COMPROMISED}`

### Chaîne

1. Énumération AD (ldap3, bind SIMPLE UPN) : cartographie des ACL du déploiement.
2. Découverte d'un compte **gMSA** (`svc_gmsa_backup`) : la lecture de son mot de passe
   managé (attribut `msDS-ManagedPassword`) est permise au principal compromis.
3. Ce compte lit le **partage caché `Backup-Archive`** sur FS01 — invisible avec les
   autres comptes (`svc_mssql` ne voit qu'`IPC$`, `svc_backup` lit `IT-Share`).
4. Lecture de `flag-i210.txt` (54 octets) sur FS01 :

   ```
   HLB2026{GMSA_MANAGED_PASSWORD_BACKUP_OPS_COMPROMISED}
   ```

### Leurre documenté

Le « validateur » interne (`/status`, `/verify/rbcd`, `/verify/shadow-credentials`,
`/verify/dcsync`) délivre `HLB2026{RBCD_RESOURCE_DELEGATION_APP01_EXPLOITED}` —
**rejeté par la plateforme** pour 250/252/254/212/279 : c'est un marqueur de compromission,
pas un flag de challenge. De plus :

- `msDS-KeyCredentialLink` **absent** → shadow credentials impossible (endpoint leurre) ;
- FS01 ne traite pas Kerberos → la délégation contrainte est inutilisable en pratique.

---

## Silent Broker (Q250) — flag marqueur validé

**Flag** : `HLB2026{INTERNAL_SERVICE_RCE_CASE_RELAY_COMPROMISED}`

### Chaîne

1. **RCE OpenWire** sur Apache ActiveMQ (CVE classique de la famille
   `ClassAssembling`/OpenWire) → shell en UID 0, callback reçu depuis la cible.
2. Lecture de `/opt/activemq/data/flag.txt`.
3. Soumission acceptée pour la question Silent Broker ; le même fichier lu sur d'autres
   instances (Sealed Order, File Vault…) était **refusé** : le marqueur n'est valide que
   pour la question qui le demande.

---

## Sealed Order (Q250-bis) & Deep Archive (Q255) — NON RÉSOLUS

- Accès obtenus et re-vérifiés : `svc_gmsa_backup` lit `Backup-Archive`
  (`tims_comm_intercepts.txt`, `flag-i210.txt`), `svc_mssql` lit `Classified` sur le DC,
  `svc_backup` lit `IT-Share` — mais aucun des deux flags n'était dans ces documents.
- Le portail interne (`/queue/kappa`, STOMP) confirme l'enregistrement d'une demande
  `release-order` (`TR-0A8BCA`, statut `Transmis`) sans jamais retourner de document.
- Blocage : les deux flags exigent une identité à privilèges supérieurs (Administrator
  local de FS01 / compte dédié) non obtenue avant expiration des instances.
