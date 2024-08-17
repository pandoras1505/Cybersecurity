# Outils de Social Engineering

## SET (Social Engineering Toolkit)

Le Social-Engineer Toolkit (SET) est un cadre open-source conçu pour effectuer des tests de pénétration et des attaques d'ingénierie sociale. SET est utilisé par les professionnels de la sécurité pour simuler des attaques d'ingénierie sociale et pour tester la résilience des organisations face à ces types d'attaques.

De base, Il est intégré à Kali Linux et Parot Security. Il est possible de le télécharger afiin de l’intégrer sur les distributions Linux et MacOs sur ce lien : https://github.com/trustedsec/social-engineer-toolkit 


## Browser Exploitation Framework (BeEF)

Le Browser Exploitation Framework (BeEF) est un outil de sécurité avancé qui permet aux professionnels de la sécurité de tester et d'exploiter les vulnérabilités des navigateurs web. BeEF se concentre sur l'exploitation des navigateurs web pour permettre aux attaquants d'exécuter des commandes à distance sur les machines des utilisateurs ciblés via leur navigateur.

### Fonctionnalités Principales de BeEF

- ```Exploitation des Navigateurs``` : BeEF peut exploiter des vulnérabilités spécifiques des navigateurs pour exécuter des commandes malveillantes sur les systèmes des utilisateurs.

- ```Modules d'Attaque``` : BeEF est livré avec une variété de modules d'attaque qui peuvent être utilisés pour effectuer différentes actions, telles que le vol de cookies, la prise de captures d'écran, et l'exécution de scripts.

- ```Hooking des Navigateurs``` : BeEF permet de "hooker" des navigateurs, c'est-à-dire de les compromettre, pour maintenir un accès persistant à travers le navigateur.

- ```Intégration avec d'autres Outils``` : BeEF peut être intégré avec d'autres outils de tests de pénétration comme Metasploit pour des attaques plus sophistiquées.

- ```Interface Web``` : BeEF fournit une interface web conviviale pour gérer les navigateurs hookés et lancer des modules d'attaque.

### Installation de BeEF

Pour installer BeEF, vous devez avoir un environnement Linux. Voici un exemple d'installation sur une distribution basée sur Debian, comme Ubuntu :

```bash
# Mettre à jour les paquets
sudo apt-get update

# Installer les dépendances nécessaires
sudo apt-get install -y git curl

# Cloner le dépôt de BeEF depuis GitHub
git clone https://github.com/beefproject/beef.git

# Accéder au répertoire cloné
cd beef

# Installer BeEF
./install

# Démarrer BeEF
./beef

```

### Utilisation de BeEF

Une fois installé et démarré, BeEF est accessible via une interface web. Par défaut, BeEF écoute sur le port 3000.

- Accéder à l'Interface Web : Ouvrez votre navigateur et accédez à http://localhost:3000/ui/panel.

- Se Connecter : Utilisez les identifiants par défaut (nom d'utilisateur : beef, mot de passe : beef) pour vous connecter. Vous pouvez changer ces identifiants dans le fichier de configuration.

- Hooker un Navigateur : Pour hooker un navigateur, vous devez inciter la cible à visiter une page web contenant le script de hook de BeEF. Ce script se trouve généralement à l'URL http://<votre-ip>:3000/hook.js. Vous pouvez l'injecter dans des pages web ou des e-mails de phishing.

- Lancer des Attaques : Une fois que le navigateur de la cible est hooké, il apparaîtra dans l'interface de BeEF. Vous pouvez alors utiliser les différents modules d'attaque pour tester et exploiter les vulnérabilités.

#### Exemples de Modules d'Attaque

- Vol de Cookies : Permet de capturer les cookies de session de l'utilisateur pour potentiellement usurper son identité.

- Récupération de l'Historique : Permet de récupérer l'historique de navigation de la cible.

- Injection de Scripts : Permet d'injecter des scripts JavaScript dans la session de navigation de l'utilisateur pour exécuter des actions spécifiques.

- Redirection de Pages : Permet de rediriger l'utilisateur vers une autre page web, potentiellement malveillante.

- Prise de Captures d'Écran : Permet de prendre des captures d'écran de la session de navigation de l'utilisateur.


## Call Spoofing Tools

Vous pouvez très facilement modifier les informations relatives à l'identification de l'appelant qui s'affichent sur un téléphone. 
affichées sur un téléphone. Il existe plusieurs outils d'usurpation d'identité qui peuvent être 
être utilisés dans les attaques d'ingénierie sociale.
Voici quelques exemples d'outils d'usurpation d'identité :

- ```SpoofApp``` : Il s'agit d'une application Apple iOS et Android qui permet d'usurper facilement un numéro de téléphone.
- ```SpoofCard``` : Il s'agit d'une application Apple iOS et Android qui permet d'usurper un numéro et de changer de voix, d'enregistrer des appels, de générer différents types d'appels, générer différents bruits de fond, et envoyer les appels directement à la messagerie vocale.
- ```Asterisk``` : Asterisk est un outil légitime de gestion de la voix sur IP (VoIP) qui peut également être utilisé pour usurper l'identité de l'appelant.