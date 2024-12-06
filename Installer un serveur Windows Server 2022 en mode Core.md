# `3) Windows 2022 + rôles`
   ### `3.1) Installation du serveur` 
### Le détail de l'instalation d'un serveur Windows Server en mode CORE n'est pas détaillé dans ce document INSTALL.md, mais ici ⬇️
* #### [Instalation d'un serveur Windows Server en mode CORE](https://www.it-connect.fr/chapitres/installer-windows-server-en-mode-core/) 

   ### `3.2) Installation du rôle AD-DS`


  
# Installation et Configuration des Services Active Directory Domain Services (AD-DS) sur Windows Server 2022 Core

## Étape 1 : Préparer le serveur Windows Server Core

### 1.1 Configurer un nom d’hôte
1. Connectez-vous à votre serveur.
2. Vérifiez le nom actuel du serveur :
   ```powershell
   hostname
   ```
3. Changez le nom du serveur :
   ```powershell
   Rename-Computer -NewName "NomDuServeur" -Restart
   ```
   Remplacez `NomDuServeur` par le nom que vous souhaitez attribuer au serveur (ex : `SRVWIN-CORE`).

4. Le serveur redémarrera automatiquement.

---

### 1.2 Configurer une adresse IP statique
1. Listez les interfaces réseau disponibles :
   ```powershell
   Get-NetIPAddress
   ```
2. Configurez une adresse IP statique, une passerelle et des serveurs DNS :
   ```powershell
   New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.40.10 -PrefixLength 24 -DefaultGateway 192.168.40.254
   Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.40.5
   ```
   - Remplacez `192.168.40.10` par l'adresse IP que vous souhaitez pour le serveur.
   - Remplacez `192.168.40.5` par l'adresse IP d'un serveur DNS (généralement celle du contrôleur de domaine principal).

3. Testez la connectivité réseau et DNS :
   ```powershell
   Test-Connection domaine.local
   ```

---

## Étape 2 : Joindre le serveur au domaine
1. Joignez le serveur au domaine existant (exemple : `domaine.local`) :
   ```powershell
   Add-Computer -DomainName "domaine.local" -Credential (Get-Credential) -Restart
   ```
   - Remplacez `domaine.local` par le nom de votre domaine.
   - Entrez les identifiants d’un compte administrateur du domaine.

2. Vérifiez que le serveur a rejoint le domaine :
   ```powershell
   Get-ComputerInfo | Select-Object CsDomain
   ```

---

## Étape 3 : Installer le rôle AD-DS

### 3.1 Installer le rôle AD-DS
1. Installez le rôle **Active Directory Domain Services** :
   ```powershell
   Install-WindowsFeature AD-Domain-Services
   ```

2. Vérifiez que l'installation est terminée avec succès :
   ```powershell
   Get-WindowsFeature AD-Domain-Services
   ```

---

## Étape 4 : Promouvoir le serveur en contrôleur de domaine

### 4.1 Lancer la promotion
1. Exécutez la commande suivante pour promouvoir le serveur en tant que contrôleur de domaine :
   ```powershell
   Install-ADDSDomainController -DomainName "domaine.local" -Credential (Get-Credential) -InstallDns:$true -SafeModeAdministratorPassword (ConvertTo-SecureString "MotDePasseDS" -AsPlainText -Force)
   ```
   - Remplacez `domaine.local` par le nom de votre domaine.
   - Remplacez `MotDePasseDS` par un mot de passe sécurisé pour le mode de restauration AD.

2. Attendez que la configuration soit terminée. Le serveur redémarrera automatiquement.

---

### 4.2 Vérifiez la promotion
1. Confirmez que le serveur est devenu un contrôleur de domaine :
   ```powershell
   Get-ADDomainController -Filter *
   ```
   Le serveur doit apparaître dans la liste des contrôleurs de domaine.

2. Vérifiez que le serveur est dans l’OU **Domain Controllers** :
   - Ouvrez **Active Directory Users and Computers** sur un autre contrôleur de domaine.
   - Assurez-vous que l'objet du serveur se trouve dans l'OU **Domain Controllers**.

---

## Étape 5 : Vérifications et tests

### 5.1 Tester la réplication Active Directory
1. Testez la réplication AD sur le serveur Core :
   ```powershell
   repadmin /replsummary
   ```
2. Affichez les détails de la réplication :
   ```powershell
   repadmin /showrepl
   ```

---

### 5.2 Valider les services DNS
1. Vérifiez que le DNS est opérationnel :
   ```powershell
   dcdiag /test:dns
   ```
2. Testez la résolution de noms dans le domaine :
   ```powershell
   Resolve-DnsName domaine.local
   ```

---

## Étape 6 : Finalisation

1. Vérifiez que le contrôleur de domaine est configuré en tant que **Global Catalog** :
   ```powershell
   Get-ADDomainController -Filter * | Select-Object Name, IsGlobalCatalog
   ```

2. Configurez les serveurs DNS pour qu'ils se référencent mutuellement en cas de panne.

---

## Résultat attendu
À la fin de ce tutoriel, votre serveur Windows Server Core sera configuré comme un contrôleur de domaine fonctionnel et intégré au domaine existant.

- Le serveur pourra gérer les authentifications et participer à la réplication AD.
- Les services DNS seront correctement configurés pour assurer la résolution des noms dans le domaine.
