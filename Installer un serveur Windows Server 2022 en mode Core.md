# `3) Windows 2022 + rôles`
   ### `3.1) Installation du serveur` 
### Le détail de l'instalation d'un serveur Windows Server en mode CORE n'est pas détaillé dans ce document INSTALL.md, mais ici ⬇️
* #### [Instalation d'un serveur Windows Server en mode CORE](https://www.it-connect.fr/chapitres/installer-windows-server-en-mode-core/) 

   ### `3.2) Installation et Configuration des Services AD-DS sur Windows Server 2022 Core`

#### Étape 1 : Préparer le serveur Windows Server Core

##### 1.1 Configurer un nom d’hôte
1. Connectez-vous à votre serveur.

2. Taper 2 sur la page de gestion :  
![1](https://github.com/user-attachments/assets/5884ced1-46d1-4dab-afb0-0335372696cb)

3. Changez le nom du serveur :  
![2](https://github.com/user-attachments/assets/fef20715-8b1a-425a-b210-d242ff50cd92)


5. Le serveur redémarrera automatiquement.

---

##### 1.2 Configurer une adresse IP statique
1. Taper 8 sur la page de gestion :
 
![3](https://github.com/user-attachments/assets/22e46eab-3fe5-4ebd-b75f-924090f200d9)
   
2. Choisissez le network que vous souhaitez configurer :

![4](https://github.com/user-attachments/assets/65fccbcf-f5ab-41da-b4ed-699ddabbb67d)

3. Entrez les différentes données pour chaque ligne :
   
![5](https://github.com/user-attachments/assets/51f530d2-1bfd-4623-bf6b-2ed29a1b7ed9)

---

#### Étape 2 : Joindre le serveur au domaine
1. Joignez le serveur au domaine existant (exemple : `domaine.local`) :
   ```powershell
   Add-Computer -DomainName "domaine.local" -Credential (Get-Credential) -Restart
   ```
   - Remplacez `domaine.local` par le nom de votre domaine.
   - Entrez les identifiants d’un compte administrateur du domaine (exemple : `domaine\Administrator`)

2. Vérifiez que le serveur a rejoint le domaine :
   ```powershell
   Get-ComputerInfo | Select-Object CsDomain
   ```

---

#### Étape 3 : Installer le rôle AD-DS

##### 3.1 Installer le rôle AD-DS
1. Installez le rôle **Active Directory Domain Services** :
   ```powershell
   Install-WindowsFeature AD-Domain-Services
   ```

2. Vérifiez que l'installation est terminée avec succès :
   ```powershell
   Get-WindowsFeature AD-Domain-Services
   ```

---

#### Étape 4 : Promouvoir le serveur en contrôleur de domaine

##### 4.1 Lancer la promotion
1. Exécutez la commande suivante pour promouvoir le serveur en tant que contrôleur de domaine :
   ```powershell
   Install-ADDSDomainController -DomainName "domaine.local" -Credential (Get-Credential) -InstallDns:$true -SafeModeAdministratorPassword (ConvertTo-SecureString "MotDePasseDS" -AsPlainText -Force)
   ```
   - Remplacez `domaine.local` par le nom de votre domaine.
   - Remplacez `MotDePasseDS` par un mot de passe sécurisé pour le mode de restauration AD.

2. Attendez que la configuration soit terminée. Le serveur redémarrera automatiquement.

---

##### 4.2 Vérifiez la promotion
1. Confirmez que le serveur est devenu un contrôleur de domaine :
   ```powershell
   Get-ADDomainController -Filter *
   ```
   Le serveur doit apparaître dans la liste des contrôleurs de domaine.

2. Vérifiez que le serveur est dans l’OU **Domain Controllers** :
   - Ouvrez **Active Directory Users and Computers** sur un autre contrôleur de domaine.
   - Assurez-vous que l'objet du serveur se trouve dans l'OU **Domain Controllers**.

---



---

#### Résultat attendu
À la fin de ce tutoriel, votre serveur Windows Server Core sera configuré comme un contrôleur de domaine fonctionnel et intégré au domaine existant.

- Le serveur pourra gérer les authentifications et participer à la réplication AD.
- Les services DNS seront correctement configurés pour assurer la résolution des noms dans le domaine.
