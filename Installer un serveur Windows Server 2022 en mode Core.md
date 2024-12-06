# `3) Windows Core + rôles`
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
1. Joignez le serveur au domaine existant (exemple : `pharmgreen.com`) :

![2](https://github.com/user-attachments/assets/13a7ddc4-9812-4f8b-8e68-9626d07d6e92)
![3](https://github.com/user-attachments/assets/2b8a7ef5-6cdb-4422-a53e-3216171d6451)


2. Facultatif : Tapez 15 sur l'écran d'accueil et vérifiez si le serveur a bien rejoint le domaine :
   ```powershell
   Get-ComputerInfo | Select-Object CsDomain
   ```

---


#### Étape 3 : Se connecter au domaine

##### 3.1 Déconnectez-vous :

![1 - Copie - Copie](https://github.com/user-attachments/assets/bbfbfd90-3f9a-450c-b01d-228c44eed5d2)

##### 3.2 Connectez-vous sur le domaine en suivant les étapes suivantes :

- Appuyez sur ECHAP  
![4](https://github.com/user-attachments/assets/82a8c52f-6339-4a55-9d24-3da62bb85f79)  
- Appuyez sur ECHAP  
![5](https://github.com/user-attachments/assets/69f056dc-d824-43fb-98d3-93a1a8528cc0)  
- Choissiez Other user  
![6](https://github.com/user-attachments/assets/2c7971e3-37a2-4af6-9f78-66226dc657ba)  
- Choisissez Local or Domain account password  
![7](https://github.com/user-attachments/assets/b63f677d-e202-4bc0-bd2c-a703707590ba)  
- Entrez votre nom d'utilisateur dans le domaine et votre mot de passe  
![8](https://github.com/user-attachments/assets/beb86c59-da0b-4cd8-b284-43f1e3ce7a95)  


#### Étape 4 : Installer le rôle AD-DS

1. Installez le rôle **Active Directory Domain Services** :
   ```powershell
   Install-WindowsFeature AD-Domain-Services
   ```

- Attendez pendant l'installation  
![10](https://github.com/user-attachments/assets/ec5a2346-8ec5-44c3-a357-ee9bb3b9da2d)  
- Une fois le processus terminé vous devriez obtenir cet écran  
![11](https://github.com/user-attachments/assets/0b093d2b-a037-4bb5-8cc4-46a17655073b)


2. Vérifiez que l'installation est terminée avec succès :
   ```powershell
   Get-WindowsFeature AD-Domain-Services
   ```
![Capture d’écran 2024-12-06 134509](https://github.com/user-attachments/assets/b313e281-2cd5-4bc4-b067-d95d12f857d3)

---

#### Étape 5 : Promouvoir le serveur en contrôleur de domaine

##### 5.1 Lancer la promotion
1. Exécutez la commande suivante pour promouvoir le serveur en tant que contrôleur de domaine :
   ```powershell
   Install-ADDSDomainController -DomainName "pharmgreen.com" -Credential (Get-Credential) -InstallDns:$true -SafeModeAdministratorPassword (ConvertTo-SecureString "MotDePasseDS" -AsPlainText -Force)
   ```
   - Remplacez `pharmgreen.com` par le nom de votre domaine.
   - Remplacez `MotDePasseDS` par un mot de passe sécurisé pour le mode de restauration AD.  
![12](https://github.com/user-attachments/assets/d5cb9bbc-9771-43d1-bc36-90c78326b7de)  


2. Attendez que la configuration soit terminée. Le serveur redémarrera automatiquement.  
![13](https://github.com/user-attachments/assets/6adb7bba-26c7-4c9e-bef8-072c48af4f50)  
![14](https://github.com/user-attachments/assets/075cc29b-1d3c-4cad-a601-067313247417)



---

##### 5.2 Vérifiez la promotion
1. Confirmez que le serveur est devenu un contrôleur de domaine :
   ```powershell
   Get-ADDomainController -Filter *
   ```
   Le serveur doit apparaître dans la liste des contrôleurs de domaine.
![15](https://github.com/user-attachments/assets/75b19533-57cf-4a8c-8574-6641bf86e0b6)  

2. Vérifiez que le serveur est dans l’OU **Domain Controllers** :
   - Ouvrez **Active Directory Users and Computers** sur un autre contrôleur de domaine.
   - Assurez-vous que l'objet du serveur se trouve dans l'OU **Domain Controllers**.
![16](https://github.com/user-attachments/assets/2f7baf1a-ca9d-4e5f-93a6-0bf19fa77bd2)  

---

##### Résultat attendu
À la fin de ce tutoriel, votre serveur Windows Server Core sera configuré comme un contrôleur de domaine fonctionnel et intégré au domaine existant.

- Le serveur pourra gérer les authentifications et participer à la réplication AD.
- Les services DNS seront correctement configurés pour assurer la résolution des noms dans le domaine.
