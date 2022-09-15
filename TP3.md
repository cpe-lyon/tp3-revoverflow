# TP3 - Utilisateurs, groupes et permissions

## Exercice 1. Gestion des utilisateurs et des groupes

Utilisez la commande groupadd pour créer deux groupes dev et infra

```bash
groupadd dev
groupadd infra
```

Créez ensuite 4 utilisateurs alice, bob, charlie, dave avec la commande useradd, en demandant la création de leur dossier personnel et avec bash pour shell

```bash
useradd -m -s /bin/bash alice
useradd -m -s /bin/bash bob
useradd -m -s /bin/bash charlie
useradd -m -s /bin/bash dave
```

Ajoutez les utilisateurs dans les groupes créés :
- alice, bob, dave dans dev
- bob, charlie, dave dans infra

```bash
usermod -a -G dev alice
usermod -a -G dev bob
usermod -a -G dev dave
usermod -a -G infra bob
usermod -a -G infra charlie
usermod -a -G infra dave
```

Donnez deux moyens d’afficher les membres de infra

```bash
getent group infra
cat /etc/group | grep infra
```

Faites de dev le groupe propriétaire des répertoires /home/alice et /home/bob et de infra le groupe propriétaire de /home/charlie et /home/dave

```bash
chown :dev /home/alice
chown :dev /home/bob
chown :infra /home/charlie
chown :infra /home/dave
```

Remplacez le groupe primaire des utilisateurs :
- dev pour alice et bob
- infra pour charlie et dave

```bash
usermod -g dev alice
usermod -g dev bob
usermod -g infra charlie
usermod -g infra dave
```

Créez deux répertoires /home/dev et /home/infra pour le contenu commun aux membres de chaque groupe, et mettez en place les permissions leur permettant d’écrire dans ces dossiers.

```bash
mkdir /home/dev
mkdir /home/infra
chown :dev /home/dev
chown :infra /home/infra
chmod 770 /home/dev
chmod 770 /home/infra
```

Comment faire pour que, dans ces dossiers, seul le propriétaire d’un fichier ait le droit de renommer ou supprimer ce fichier ?

```bash
chmod 750 /home/dev
chmod 750 /home/infra
```

Pouvez-vous ouvrir une session en tant que alice ? Pourquoi ?

```bash
su alice
```

Oui car je suis connecté en tant que root, mais si j'essaye sans être root je ne pourrais pas car alice ne possède pas de mot de passe.

Activez le compte de l’utilisateur alice et vérifiez que vous pouvez désormais vous connecter avec son compte

```bash
passwd alice
su alice
```

Comment obtenir l’uid et le gid de alice ?

```bash
id alice
```

Quelle commande permet de retrouver l’utilisateur dont l’uid est 1003 ?

```bash
getent passwd 1003
```

Quel groupe a pour gid 1002 ?

```bash
getent group 1002
```

Retirez l’utilisateur charlie du groupe infra. Que se passe-t-il ? Expliquez.

```bash
gpasswd -d charlie infra
```

Il ne peut plus accéder au dossier /home/infra car il n'est plus dans le groupe infra.

Modifiez le compte de dave de sorte que :
- il expire au 1
er juin 2021
- il faut changer de mot de passe avant 90 jours
- il faut attendre 5 jours pour modifier un mot de passe
- l’utilisateur est averti 14 jours avant l’expiration de son mot de passe
- le compte sera bloqué 30 jours après expiration du mot de passe

```bash
chage -E 2021-06-01 -m 90 -M 5 -W 14 -I 30 dave
```

Quel est l’interpréteur de commandes (Shell) de l’utilisateur root ?

```bash
$ cat /etc/passwd | grep root
root:x:0:0:root:/root:/bin/bash
```

C'est /bin/bash

Si vous regardez la liste des comptes présents sur la machine, vous verrez qu’il en existe un nommé nobody. A quoi correspond-il ?

```bash
$ cat /etc/passwd | grep nobody
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
```

C'est un utilisateur qui n'a pas de mot de passe et qui ne peut pas se connecter.

Par défaut, combien de temps la commande sudo conserve-t-elle votre mot de passe en mémoire ? Quelle commande permet de forcer sudo à oublier votre mot de passe ?

```bash
$ sudo -K
```

Par défaut, sudo conserve le mot de passe en mémoire pendant 15 minutes.

## Exercice 2. Gestion des permissions

Dans votre $HOME, créez un dossier test, et dans ce dossier un fichier fichier contenant quelques lignes de texte. Quels sont les droits sur test et fichier ?

```bash
drwxrwxr-x 2 user user 4096 Sep 15 07:23 test
-rw-rw-r-- 1 user user    6 Sep 15 07:23 fichier
```

Retirez tous les droits sur ce fichier (même pour vous), puis essayez de le modifier et de l’afficher en
tant que root. Conclusion ?

```bash
chmod 000 fichier
```

En tant que root j'ai toujours accès au fichier.

Redonnez vous les droits en écriture et exécution sur fichier puis exécutez la commande echo "echo Hello" > fichier. On a vu lors des TP précédents que cette commande remplace le contenu d’un fichier s’il existe déjà. Que peut-on dire au sujet des droits ?

```bash
chmod 700 fichier
echo "echo Hello" > fichier
```

Les droits sont conservés.

Essayez d’exécuter le fichier. Est-ce que cela fonctionne ? Et avec sudo ? Expliquez.

```bash
./fichier
```

Non car il n'est pas exécutable.

```bash
sudo ./fichier
```

Oui car sudo permet d'exécuter un fichier même si il n'est pas exécutable.

Placez-vous dans le répertoire test, et retirez-vous le droit en lecture pour ce répertoire. Listez le contenu du répertoire, puis exécutez ou affichez le contenu du fichier fichier. Qu’en déduisez-vous ? Rétablissez le droit en lecture sur test.

```bash
$ chmod -r test
$ ls test
ls: cannot open directory 'test': Permission denied
$ cat test/fichier
salut
```

Je ne peux pas lister le contenu du répertoire mais je peux quand même afficher le contenu du fichier.

```bash
$ chmod +r test
```

Créez dans test un fichier nouveau ainsi qu’un répertoire sstest. Retirez au fichier nouveau et au répertoire test le droit en écriture. Tentez de modifier le fichier nouveau. Rétablissez ensuite le droit en écriture au répertoire test. Tentez de modifier le fichier nouveau, puis de le supprimer. Que pouvezvous déduire de toutes ces manipulations ?

```bash
$ touch test/nouveau
$ mkdir test/sstest
$ chmod -w test/nouveau
$ chmod -w test
$ echo "salut" > test/nouveau
bash: test/nouveau: Permission denied
$ chmod +w test
$ echo "salut" > test/nouveau
$ rm test/nouveau
```

Je ne peux pas modifier le fichier nouveau lorsque je n'ai pas les droits en écriture sur ce fichier mais je peux le supprimer si j'ai les droits en écriture sur le répertoire parent.

Positionnez vous dans votre répertoire personnel, puis retirez le droit en exécution du répertoire test. Tentez de créer, supprimer, ou modifier un fichier dans le répertoire test, de vous y déplacer, d’en lister le contenu, etc… Qu’en déduisez vous quant au sens du droit en exécution pour les répertoires ?

```bash
$ chmod -x test
$ touch test/nouveau
bash: test/nouveau: Permission denied
$ rm test/nouveau
rm: cannot remove 'test/nouveau': Permission denied
$ echo "salut" > test/nouveau
bash: test/nouveau: Permission denied
$ cd test
bash: cd: test: Permission denied
$ ls test
ls: cannot open directory 'test': Permission denied
```

Je ne peux pas créer, supprimer, modifier un fichier dans le répertoire test, me déplacer dans le répertoire test, lister le contenu du répertoire test.

Rétablissez le droit en exécution du répertoire test. Positionnez vous dans ce répertoire et retirez lui à nouveau le droit d’exécution. Essayez de créer, supprimer et modifier un fichier dans le répertoire test, de vous déplacer dans sstest, de lister son contenu. Qu’en concluez-vous quant à l’influence des droits que l’on possède sur le répertoire courant ? Peut-on retourner dans le répertoire parent avec "cd .." ? Pouvez-vous donner une explication ?

```bash
$ chmod +x test
$ cd test
$ chmod -x .
$ touch nouveau
$ rm nouveau
$ echo "salut" > nouveau
$ cd sstest
$ ls sstest
```

Je peux créer, supprimer, modifier un fichier dans le répertoire test, me déplacer dans le répertoire sstest, lister le contenu du répertoire sstest.

Les droits que l'on possède sur le répertoire courant n'influencent pas les droits que l'on possède sur les répertoires et fichiers enfants.

Non je ne peux pas retourner dans le répertoire parent avec "cd .." car je n'ai pas les droits en exécution sur le répertoire parent.

Rétablissez le droit en exécution du répertoire test. Attribuez au fichier fichier les droits suffisants pour qu’une autre personne de votre groupe puisse y accéder en lecture, mais pas en écriture.

```bash
$ chmod +x test
$ chmod g+r test/fichier
```

Définissez un umask très restrictif qui interdit à quiconque à part vous l’accès en lecture ou en écriture, ainsi que la traversée de vos répertoires. Testez sur un nouveau fichier et un nouveau répertoire.

```bash
$ umask 077
$ touch nouveau
$ mkdir nouveaudir
$ ls -al
-rw------- 1 user user    0 Sep 15 07:43 nouveau
drwx------ 2 user user 4096 Sep 15 07:43 nouveaudir
```

Définissez un umask très permissif qui autorise tout le monde à lire vos fichiers et traverser vos répertoires, mais n’autorise que vous à écrire. Testez sur un nouveau fichier et un nouveau répertoire.

```bash
$ umask 022
$ touch nouveau
$ mkdir nouveaudir
$ ls -al
-rw-r--r-- 1 user user    0 Sep 15 07:45 nouveau
drwxr-xr-x 2 user user 4096 Sep 15 07:45 nouveaudir
```

Définissez un umask équilibré qui vous autorise un accès complet et autorise un accès en lecture aux membres de votre groupe. Testez sur un nouveau fichier et un nouveau répertoire.

```bash
$ umask 027
$ touch nouveau
$ mkdir nouveaudir
$ ls -al
-rw-r----- 1 user user    0 Sep 15 07:46 nouveau
drwxr-x--- 2 user user 4096 Sep 15 07:46 nouveaudir
```

Transcrivez les commandes suivantes de la notation classique à la notation octale ou vice-versa (vous
pourrez vous aider de la commande stat pour valider vos réponses) :
- chmod u=rx,g=wx,o=r fic
- chmod uo+w,g-rx fic en sachant que les droits initiaux de fic sont r--r-x---
- chmod 653 fic en sachant que les droits initiaux de fic sont 711
- chmod u+x,g=w,o-r fic en sachant que les droits initiaux de fic sont r--r-x---

```bash
1: r-x-wxr-- : 534
2: rw-----w-: 602
3: r-x-wxr-- : 534
4: r-x-w---- : 520
```

Affichez les droits sur le programme passwd. Que remarquez-vous ? En affichant les droits du fichier /etc/passwd, pouvez-vous justifier les permissions sur le programme passwd ?

```bash
$ ls -l /etc/passwd
-rw-r--r-- 1 root root 0 Sep 15 07:47 /etc/passwd
$ ls -l /usr/bin/passwd
-rwsr-xr-x 1 root root 0 Sep 15 07:47 /usr/bin/passwd
```

Le fichier /etc/passwd est accessible en lecture par tous les utilisateurs, mais seulement en écriture par l'utilisateur root.

Le programme passwd est un programme setuid. Il est donc exécuté avec les droits de l'utilisateur root.