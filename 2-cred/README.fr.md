# Atelier - Les inventaires, identifications et commandes Ad-hoc

**Lisez ceci dans d'autres langues**:
<br>![uk](../images/uk.png) [English](README.md),![france](../images/fr.png) [Française](README.fr.md)

## Table des matières

* [Objectif](#objectif)
* [Guide](#guide)
* [Creer un inventaire](#creer-un-inventaire)
* [Machine Credentials](#machine-credentials)
* [Configurer des informations d'identifications](#configurer-des-informations-d-identification)
* [Exécution des commandes Ad-hoc](#exécution-des-commandes-ad-hoc)
* [Défi: Les commandes Ad-hoc](#défi-les-commandes-ad-hoc)

# Objectif

Explorez et comprenez l'environnement du laboratoire. Cet exercice couvrira
- Localisation et compréhension:
  - Tour Ansible [**Inventaire**](https://docs.ansible.com/ansible-tower/latest/html/userguide/inventories.html)
  - Tour Ansible [**Informations d'identification**](https://docs.ansible.com/ansible-tower/latest/html/userguide/credentials.html)
- Exécution de commandes ad hoc via l'interface utilisateur Web d'Ansible Tower

# Guide

## Creer un inventaire

La première chose dont nous avons besoin est un inventaire de vos hôtes. C'est l'équivalent d'un fichier d'inventaire dans Ansible Engine. Il y a plusieurs possibilités pour définir un inventaire dans Tower (comme les inventaires dynamiques), mais commençons par les bases.

Vous devriez déjà avoir l'interface utilisateur Web ouverte, sinon ouvrir :

    https://student<N>.<LABID>.events.opentlc.com

(remplacez `<N>` avec votre numéro d'étudiant et `<LABID>` avec le nom de votre atelier actuel) et connectez-vous en tant qu'"admin" avec le mot de passe fournit sur la page d'accueil.

Creer l'inventaire:
* Dans Tower, cliquer sur  **RESSOURCES** → **Inventaires**
* Cliquer sur le bouton '+' et **Inventaire**
* Remplir le champs 'NOM' : Workshop Inventory
* Remplir le champs 'ORGANISATION' : Default
* ENREGISTRER


Revenir sur la liste des inventaires. Ouvrir 'Workshop Inventory' et cliquer  sur **HOTES**, la liste est vide car nous n'avons pas encore ajouté d'Hotes. 

Rajoutons donc des Hotes. Tout d'abord, nous devons recupérer la liste des machines qui sont accessibles depuis votre lab.

En utilisant l'interface code-server, ouvrez un terminal et ouvrir le contenu du fichier  ~/lab_inventory/hosts :

```bash
$ cat ~/lab_inventory/hosts
[all:vars]
ansible_user=student<X>
ansible_ssh_pass=PASSWORD
ansible_port=22

[web]
node1 ansible_host=22.33.44.55
node2 ansible_host=33.44.55.66
node3 ansible_host=44.55.66.77

[control]
ansible ansible_host=11.22.33.44
```

> **Warning**
>
> Les IPs de votre lab seront différentes


Revenir sur l'interface Tower, cliquer sur le bouton '+'  :
* NOM: node1
* Dans le champs **variables**, ajoutr **ansible_host: x.x.x.x** en utilisant l'IP provenant dans votre fichier  ~/lab_inventory/hosts file
* "Enregister" 
* Fermer le menu d'édition de l'hote en cliquant sur l'icone 'x' en haut à droite


![Create host](images/create_host.png) 

Répéter ces étapes pour ajouter les hotes node2 et node3.


A la fin, cliquer sur  **RESSOURCES → Inventaires → Workshop Inventory** et **HÔTES**  , vérifier que les 3 hotes sont présents :

![hosts_added](images/3nodes_added.png) 




###  Machine informations d identification

Pour que Tower puisse executer des actions sur des hotes distants, il faut configurer des informations d'identification (mot de passe, clé SSH, ...)
Une des fonctionalités les plus importantes de Tower est de pouvoir stocker et utiliser ces informations, sans les rendre visibles. 

> **Tip**
>
> Les informations d'identification sont definies de manière indépendantes et non pas directement attachées à un Hote ou un Inventaire. 

Pour tester l'accès aux hotes :

* Utiliser code-server et ouvrir un terminal (il fonctionne par défaut sur le noeud Tower)
* Se connecter en SSH avec l'utilisateur  ec2-user  sur node1 (ou node 2 ou node3) et executer sudo -i.

```bash
[student1@ansible ~]$ ssh ec2-user@node1
[ec2-user@node1 ~]$
sudo -i
[root@node1 ~]# exit
[ec2-user@node1 ~]$ exit
```

En resumé ?
* L'utilisateur  `student<N>` peut se connecter en SSH sans mot de passe sur les machines en utilisant l'utilisateur ec2-user.
* L'utilisateur ec2-user peut effectuer des commandes en tant que root via sudo, sans renseigner de mot de passe additionel.


## Configurer des informations d identification

Maintenant, nous allons configurer des informations d'identification pour que Tower puisse accéder à nos hotes :

* Depuis l'interface Tower, dans le menu **RESSOURCES**, choisir **informations d'identification**
* Cliquer  sur le bouton '+' 

* Remplir les champs :
 * NOM: Workshop Credentials
 * ORGANISATION: Cliquer sur la loupe, et choisir **Default**
 * Type d'informations d’identification : Cliquer sur la loupe, et rechercher 'machine'. Cocher et valider en appuyant sur 'selectionner'
 * Nom d'utilisateur: ec2-user
 * Méthode d'escalade privilégiée: sudo

Comme nous utilisons une identification par clé SSH, il faut fournir la clé SSH privée key  (On pourrait egalement utiliser une authentification par mot de passe).

Se connecter au terminal code-server et ouvrir le fichier .ssh/aws-private.pem , qui contient la clé privée SSH  : 

```bash
[student1@ansible ~]$ cat .ssh/aws-private.pem
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA2nnL3m5sKvoSy37OZ8DQCTjTIPVmCJt/M02KgDt53+baYAFu1TIkC3Yk+HK1
[...]
-----END RSA PRIVATE KEY-----`
```


* Copier cette clé (bien inclure les lignes “BEGIN” et “END” ) dans le champs "Clé privée SSH" de l'interface Tower.
* Sauvegarder

Revenir sur **RESSOURCES** -> **Informations d'identification -> Workshop Credentials** et constater que la clé SSH n'est plus visible et est chiffrée.

Bravo ! Vous avez configuré des informations d'identification qui seront utilisées dans les exercices suivants.


## Exécution des commandes Ad hoc

Il est possible d'exécuter des commandes ad hoc à partir d'Ansible Tower.

   - Dans l'interface utilisateur Web, accédez à **RESSOURCES → Inventaires → Workshop Inventory**

   - Cliquez sur le bouton **HÔTES** pour passer à la vue des hôtes et sélectionnez les trois hôtes en cochant les cases à gauche des entrées d'hôte.

   - Cliquez sur **EXÉCUTER COMMANDE**. Dans l'écran suivant, vous devez spécifier la commande ad hoc:
  <table>
    <tr>
      <th>Parametre</th>
      <th>Valeur</th>
    </tr>
    <tr>
      <td>MODULE</td>
      <td>ping</td>
    </tr>
    <tr>
      <td>MACHINE CREDENTIAL</td>
      <td>Workshop Credentials</td>
    </tr>
  </table>

  - Cliquer sur  **LANCER**, et observer les traces.

<hr>

Le module **ping** simple n'a pas besoin d'options. Pour les autres modules, vous devez fournir la commande à exécuter comme argument. Essayez le module **command** pour trouver l'ID utilisateur de l'utilisateur exécutant à l'aide d'une commande ad hoc.
  <table>
    <tr>
      <th>Parametre</th>
      <th>Valeur</th>
    </tr>
    <tr>
      <td>MODULE</td>
      <td>command</td>
    </tr>
    <tr>
      <td>ARGUMENTS</td>
      <td>id</td>
    </tr>
  </table>

> **Astuce**
>
> Après avoir choisi le module à exécuter, Tower fournira un lien vers la page de documentation du module en cliquant sur le point d'interrogation à côté de "Arguments". C'est pratique, essayez-le.

<hr>

Que diriez-vous d'essayer d'obtenir des informations secrètes du système? Essayez d'afficher le fichier */etc/shadow*.

<table>
  <tr>
    <th>Parametre</th>
    <th>Valeur</th>
  </tr>
  <tr>
    <td>MODULE</td>
    <td>command</td>
  </tr>
  <tr>
    <td>ARGUMENTS</td>
    <td>cat /etc/shadow</td>
  </tr>
</table>


> **Avertissement**
>
> **Attendez-vous à une erreur \!**

Oups, le dernier ne s'est pas bien passé, tout rouge.

Réexécutez la dernière commande Ad-hoc mais cette fois cochez la case **Activer l’élévation des privilèges**.

Comme vous le voyez, cette fois, cela a fonctionné. Pour les tâches qui doivent s'exécuter en tant que root, vous devez augmenter les privilèges. C'est la même chose que le **become: yes** utilisé dans vos Playbooks Ansible.

## Défi: Les commandes Ad hoc

D'accord, un petit défi: exécutez un ad hoc pour vous assurer que le package "tmux" est installé sur tous les hôtes. En cas de doute, consultez la documentation soit via l'interface utilisateur Web comme indiqué ci-dessus, soit en exécutant `[ansible @ tower ~] $ ansible-doc yum` sur votre hôte de contrôle Tower.

> **Avertissement**
>
> **Solution ci-dessous \!**

<table>
  <tr>
    <th>Parametre</th>
    <th>Valeur</th>
  </tr>
  <tr>
    <td>MODULE</td>
    <td>yum</td>
  </tr>
  <tr>
    <td>ARGUMENTS</td>
    <td>name=tmux</td>
  </tr>
  <tr>
    <td>Activer l’élévation des privilèges</td>
    <td>✓</td>
  </tr>
</table>

> **Astuce**
>
> La sortie jaune de la commande indique qu'Ansible a réellement fait quelque chose (ici, il fallait installer le paquet). Si vous exécutez la commande ad hoc une deuxième fois, la sortie sera verte et vous informera que le package a déjà été installé. Donc, le jaune dans Ansible ne signifie pas "soyez prudent"… ;-).
----
**Navigation**
<br>
[Exercice précédent](../1-intro/README.fr.md) - [Exercice suivant](../3-projects/README.fr.md)

[Cliquez ici pour revenir à l'atelier Ansible pour Red Hat Enterprise Linux](../README.fr.md)
