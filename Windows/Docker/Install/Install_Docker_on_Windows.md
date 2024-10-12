# Guide d'Installation de Docker sur Windows

- [[#Prérequis|Prérequis]]
- [[#Étapes d'Installation de Docker|Étapes d'Installation de Docker]]
	- [[#Étapes d'Installation de Docker#1. Installer le Module Docker pour Windows|1. Installer le Module Docker pour Windows]]
	- [[#Étapes d'Installation de Docker#2. Installer la Dernière Version de Docker|2. Installer la Dernière Version de Docker]]
	- [[#Étapes d'Installation de Docker#3. Redémarrer l'Ordinateur|3. Redémarrer l'Ordinateur]]
	- [[#Étapes d'Installation de Docker#4. Vérification de l'Installation|4. Vérification de l'Installation]]
- [[#Résolution des Problèmes|Résolution des Problèmes]]
## Prérequis
Avant de commencer, assurez-vous de disposer des éléments suivants :
- **Powershell** en tant qu'administrateur.

## Étapes d'Installation de Docker

### 1. Installer le Module Docker pour Windows

Ouvrez **Powershell** en mode Administrateur et exécutez la commande suivante pour installer le module DockerMsftProvider depuis le dépôt PSGallery :

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery
```

Pendant le processus d'installation, vous recevrez des **pop-ups** vous demandant d'autoriser l'installation et l'importation du fournisseur **NuGet**. Cliquez sur **Oui** pour confirmer.

Ensuite, un autre message apparaîtra, vous avertissant que les modules sont installés à partir d'un référentiel non approuvé. Ce message indiquera que l'**InstallationPolicy** sera modifiée via la commande **Set-PSRepository**. Cliquez à nouveau sur **Oui pour tout**.

### 2. Installer la Dernière Version de Docker

Une fois le module DockerMsftProvider installé, exécutez la commande suivante pour installer Docker :

```powershell
Install-Package -Name docker -ProviderName DockerMsftProvider
```

Lorsque le message de confirmation s'affiche pour valider l'installation depuis **DockerDefault**, sélectionnez **Oui pour tout** afin de poursuivre l'installation.

### 3. Redémarrer l'Ordinateur

Après l'installation, il est nécessaire de redémarrer votre machine pour appliquer les changements. Tapez simplement la commande suivante dans **Powershell** :

```powershell
Restart-Computer
```

### 4. Vérification de l'Installation

Une fois votre ordinateur redémarré, ouvrez **Powershell** à nouveau et tapez la commande suivante pour vérifier que Docker fonctionne correctement :

```powershell
docker ps
```

Si Docker est installé et actif, cette commande devrait lister les conteneurs en cours d'exécution (même s'il n'y en a pas encore).


