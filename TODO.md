

Finir de documenter le fichier generate-matchs.mjs

Revoir le guidedeveloppeur.md
    a epurer surement
    Expliquer le script dans le planificateur de tache, mettre le code dedans


Attention j'ai du revoir la clé pour que ce soit sur les 2 repos, maj d ela doc avec ceci

Dans ton cas, je te conseille de remplacer la Deploy Key par une clé SSH dédiée à ton NAS, enregistrée sur ton compte GitHub.

Contrairement à une Deploy Key, cette clé pourra accéder à plusieurs dépôts, à condition que le compte GitHub qui lui est associé dispose des droits nécessaires sur chacun d'eux.

On va conserver ton organisation actuelle sur le Synology et éviter de toucher aux clés déjà utilisées tant que la nouvelle configuration n'est pas validée.

## 1. Générer une nouvelle clé SSH sur le NAS

Connecte-toi en SSH au NAS avec ton compte `github_azurea`, puis exécute :

```
ssh-keygen -t ed25519 -C "synology-azurea" \
-f /var/services/homes/github_azurea/.ssh/github_azurea_multi
```

Lorsqu'on te demande une passphrase, tu peux laisser vide pour permettre l'exécution automatique des tâches planifiées.

Cela crée deux fichiers :

- `github_azurea_multi` : clé privée, à conserver sur le NAS.
- `github_azurea_multi.pub` : clé publique, à enregistrer sur GitHub.

## 2. Ajouter la clé publique à ton compte GitHub

Affiche la clé publique :

```
cat /var/services/homes/github_azurea/.ssh/github_azurea_multi.pub
```

Copie son contenu.

Sur GitHub :

1. Ouvre les paramètres de ton compte.
2. Va dans Settings → SSH and GPG keys.
3. Clique sur New SSH key.
4. Titre : `Synology Azurea`.
5. Key type : `Authentication Key`.
6. Colle la clé publique et valide.

## 3. Configurer SSH sur le NAS

Toujours avec le compte `github_azurea`, crée ou complète le fichier :

```
vi /var/services/homes/github_azurea/.ssh/config
```

Ajoute :

```
Host github.com
    HostName github.com
    User git
    IdentityFile /var/services/homes/github_azurea/.ssh/github_azurea_multi
    IdentitiesOnly yes
```

Puis sécurise les fichiers :

```
chmod 700 /var/services/homes/github_azurea/.ssh
chmod 600 /var/services/homes/github_azurea/.ssh/github_azurea_multi
chmod 600 /var/services/homes/github_azurea/.ssh/config
chmod 644 /var/services/homes/github_azurea/.ssh/github_azurea_multi.pub
```

Important : la tâche DSM doit utiliser le compte qui a accès à cette clé et à ce fichier `config`. Je te conseille de configurer la tâche pour qu'elle s'exécute avec `github_azurea`.

## 4. Tester l'authentification

Exécute :

```
ssh -T git@github.com
```

Tu dois obtenir un message de ce type :

```
Hi TON_COMPTE_GITHUB! You've successfully authenticated,
but GitHub does not provide shell access.
```

Cela confirme que GitHub reconnaît bien ta nouvelle clé.

## 5. Vérifier l'accès aux différents dépôts

Teste ensuite les deux dépôts :

```
git ls-remote git@github.com:azureabasketgolfe/Test.git

git ls-remote git@github.com:azureabasketgolfe/Planning-AZUREA.git
```

Si les deux commandes affichent des références Git, l'accès est opérationnel.

Attention : le compte GitHub auquel tu as associé la clé doit avoir au minimum un accès en lecture à chacun des dépôts privés concernés.

## 6. Reprendre ton script DSM

Tu pourras alors utiliser normalement :

```
/usr/local/bin/git clone git@github.com:azureabasketgolfe/Planning-AZUREA.git \
    /volume1/AzureaBasket/depot
```

La même clé sera utilisée automatiquement pour les différents dépôts.

Un dernier point concernant la sécurité : cette clé donnera accès à tous les dépôts auxquels ton compte GitHub a accès, et pas uniquement aux dépôts Azuréa. Je te conseille donc d'utiliser un compte GitHub technique dédié si tu souhaites limiter précisément les permissions du NAS.



