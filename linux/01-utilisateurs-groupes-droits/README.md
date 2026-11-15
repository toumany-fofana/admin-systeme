# Dossiers partagés par service (Debian 13)

Deux services, compta et informatique. Chacun a son dossier dans
/srv/partage, accessible uniquement à ses membres. Un seul compte
a les droits sudo.

Testé sur srv-deb-01, Debian 13, VM VMware.

## Comptes et groupes

```bash
sudo adduser --disabled-password alice
sudo adduser --disabled-password bob
sudo adduser --disabled-password david
sudo adduser chloe

sudo groupadd compta
sudo groupadd informatique

sudo usermod -aG compta alice
sudo usermod -aG compta bob
sudo usermod -aG informatique david
sudo usermod -aG informatique,sudo chloe
```

Seule chloe a un mot de passe, elle en a besoin pour sudo.

## Dossiers

```bash
sudo mkdir -p /srv/partage/compta /srv/partage/informatique
sudo chown root:compta /srv/partage/compta
sudo chown root:informatique /srv/partage/informatique
sudo chmod 2770 /srv/partage/compta /srv/partage/informatique
```

```text
drwxrws--- 2 root compta       4096 compta
drwxrws--- 2 root informatique 4096 informatique
```

Le 2 (setgid) fait hériter le groupe du dossier aux fichiers créés
dedans. Sans lui, un fichier d'alice part dans le groupe alice et bob
ne peut pas l'ouvrir en écriture.

## Tests

Faits avec `sudo -iu <user>` :

- alice crée un fichier dans compta : ok, groupe compta
- bob écrit dedans : ok
- chloe fait un cat dessus : permission non accordée
- sudo whoami avec alice : refusé, avec chloe : root

## À retenir

- `usermod -G` sans `-a` écrase tous les groupes secondaires. Testé sur
  david : il a perdu users et informatique d'un coup.
- `passwd -d` vide le mot de passe, il ne bloque pas le compte. Pour
  bloquer c'est `passwd -l`. Vérification avec `sudo passwd -S <user>`.
- chloe est bloquée par le dossier, pas par le fichier : sans le x sur
  le dossier, les droits du fichier ne sont même pas lus.