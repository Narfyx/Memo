# Guide d'installation et de sécurisation d'Armbian sur NanoPi NEO3 (1 Go de RAM)

## 1. Préparation de la carte SD avec Armbian

1. **Téléchargez l'image Armbian**  
   Rendez-vous sur le site officiel pour télécharger l'image Armbian compatible avec le NanoPi NEO3 :  
   [Lien de téléchargement Armbian](https://www.armbian.com/nanopineo3/).  
   Choisissez la version stable (ex. Debian Bookworm) pour une meilleure stabilité.

2. **Préparez la carte SD**  
   - Insérez une carte SD (16 Go minimum) dans votre ordinateur.
   - Utilisez **Rufus** ou **Etcher** pour écrire l'image Armbian sur la carte SD en veillant à sélectionner le bon périphérique.

3. **Insertion dans le NanoPi et démarrage**  
   Une fois l'image écrite, retirez la carte SD en toute sécurité et insérez-la dans le NanoPi NEO3.

---

## 2. Premier démarrage et connexion SSH

1. **Démarrage du NanoPi**  
   Connectez le NanoPi à l'alimentation et au réseau local via un câble Ethernet. Le NanoPi obtiendra automatiquement une adresse IP de votre serveur DHCP.

2. **Connexion via SSH**  
   Retrouvez l’adresse IP du NanoPi (depuis l’interface de votre routeur ou via un logiciel de scan réseau). Connectez-vous en SSH :

   ```bash
   ssh root@ip_address
   ```

   - **Nom d’utilisateur par défaut :** `root`
   - **Mot de passe par défaut :** `1234`

3. **Configuration initiale**  
   Lors de la première connexion, il vous sera demandé de :
   - **Définir un nouveau mot de passe pour root**
   - **Créer un utilisateur** avec droits sudo pour une utilisation quotidienne.

   Exemple de configuration :

   ```plaintext
   Create root password: *********
   Repeat root password: *********
   Please provide a username: user
   Create user (user) password: **************
   Repeat user (user) password: **************
   ```

4. **Première commande**

   ```plaintext
   su user
   ```

   Cette commande permet de passer à l’utilisateur nouvellement créé (`user` dans l'exemple).

---

## 3. Sécurisation du système

Après la configuration initiale, renforcez la sécurité de votre système avec les étapes suivantes.

### 3.1 Modification des ports par défaut

Changer le port SSH (par défaut 22) aide à réduire les tentatives de connexion non autorisées.

1. **Modifier la configuration SSH :**  
   Éditez le fichier de configuration SSH avec `nano` ou un autre éditeur :

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. **Changer le port SSH**  
   Recherchez la ligne contenant `Port` et modifiez le numéro de port (par exemple, `2222`) :

   ```plaintext
   Port 2222
   ```

3. **Désactiver la connexion root directe**  
   Toujours dans le fichier `/etc/ssh/sshd_config`, trouvez la ligne `PermitRootLogin` et réglez-la sur `no` pour désactiver l'accès SSH root.

   ```plaintext
   PermitRootLogin no
   ```

4. **Redémarrer le service SSH**  
   Après les modifications, redémarrez SSH :

   ```bash
   sudo systemctl restart ssh
   ```

5. **Connexion avec le nouveau port**  
   Utilisez désormais le nouveau port configuré :

   ```bash
   ssh -p 2222 user@ip_address
   ```

### 3.2 Configuration de pare-feu avec UFW

**UFW** (Uncomplicated Firewall) simplifie la gestion d’`iptables` pour limiter les accès.

1. **Installer UFW** (si non installé) :

   ```bash
   sudo apt update && sudo apt install ufw -y
   ```

2. **Autoriser SSH sur le nouveau port** :

   ```bash
   sudo ufw allow 2222/tcp
   ```

3. **Activer le pare-feu** :

   ```bash
   sudo ufw enable
   ```

4. **Vérifier l’état du pare-feu** :

   ```bash
   sudo ufw status
   ```

### 3.3 Mettre en place les mises à jour automatiques

1. **Installer `unattended-upgrades`** :  
   [Guide complet pour `unattended-upgrades`](https://guide.ubuntu-fr.org/server/automatic-updates.html)

   ```bash
   sudo apt install unattended-upgrades -y
   ```

2. **Configurer `unattended-upgrades`** :  
   Éditez le fichier `/etc/apt/apt.conf.d/50unattended-upgrades` et vérifiez que les lignes suivantes sont activées pour appliquer les mises à jour de sécurité :

   ```plaintext
   Unattended-Upgrade::Origins-Pattern {
       "origin=Debian,codename=${distro_codename}-updates";
       "origin=Debian,codename=${distro_codename},label=Debian";
       "origin=Debian,codename=${distro_codename}-security,label=Debian-Security";
   };
   ```

3. **Activer les mises à jour automatiques** en modifiant `/etc/apt/apt.conf.d/20auto-upgrades` :

   ```plaintext
   APT::Periodic::Update-Package-Lists "1";
   APT::Periodic::Download-Upgradeable-Packages "1";
   APT::Periodic::AutocleanInterval "7";
   APT::Periodic::Unattended-Upgrade "1";
   ```

4. **Configurer les notifications de mise à jour :**  
   Le paramètre `Unattended-Upgrade::Mail` dans `/etc/apt/apt.conf.d/50unattended-upgrades` permet d'envoyer un courriel à l'administrateur. Pour configurer des alertes de mise à jour supplémentaires, installez **apticron** :

   ```bash
   sudo apt install apticron
   ```

   Ensuite, modifiez `/etc/apticron/apticron.conf` pour définir l’adresse email de notification :

   ```plaintext
   EMAIL="root@exemple.com"
   ```

### 3.4 Utiliser Fail2Ban pour protéger les accès SSH

**Fail2Ban** surveille les tentatives de connexion et bloque les adresses IP suspectes.

1. **Installer Fail2Ban** :

   ```bash
   sudo apt install fail2ban -y
   ```

2. **Configurer Fail2Ban pour SSH** :  
   Créez un fichier `/etc/fail2ban/jail.d/custom.conf` contenant les paramètres par défaut suivants :

   ```plaintext
   [DEFAULT]
   destemail = adresse@example.com
   ignoreip = 127.0.0.1 123.123.123.0/24
   findtime = 10m
   bantime = 24h
   maxretry = 5
   ```

   - `ignoreip` : liste d'adresses IP à ignorer pour ne pas être bannies.
   - `findtime` : durée pendant laquelle les tentatives de connexion sont surveillées (10 minutes).
   - `bantime` : durée du bannissement (24 heures).
   - `maxretry` : nombre maximal de tentatives avant bannissement.

3. **Configurer la surveillance des connexions SSH** :  
   Ajoutez le bloc suivant dans `/etc/fail2ban/jail.d/custom.conf` :

   ```plaintext
   [sshd]
   enabled = true
   port    = 2222
   logpath = %(sshd_log)s
   backend = %(sshd_backend)s
   ```

   Ce bloc indique :

   - **port** : les ports à bloquer via `iptables`
   - **logpath** : l'emplacement des fichiers de log à surveiller
   - **backend** : le moteur de surveillance des logs.

4. **Démarrer et activer Fail2Ban** :

   ```bash
   sudo systemctl enable fail2ban
   sudo systemctl start fail2ban
   ```

5. **Vérifier le statut de Fail2Ban** :

   ```bash
   sudo fail2ban-client status
   ```

---

## 4. Vérification de la configuration et conseils de maintenance

- **Surveillance des performances système** : Utilisez `htop` pour vérifier l’état du processeur, de la RAM et des processus :

  ```bash
  htop
  ```

- **Sauvegardes régulières** : Sauvegardez la configuration Armbian et les fichiers essentiels pour faciliter la récupération.

- **Vérification des journaux de sécurité** : Utilisez `journalctl` pour surveiller les journaux système et détecter des anomalies :

  ```bash
  sudo journalctl -u ssh -f
  ```
