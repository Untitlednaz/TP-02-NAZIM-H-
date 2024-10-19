Compte Rendu : TP 02 - Services, Processus et Signaux

1. Secure Shell (SSH)

1.1 Connexion SSH root et configuration

Pour permettre la connexion SSH en tant que root, j’ai modifié la ligne PermitRootLogin dans le fichier /etc/ssh/sshd_config :

•	Ligne modifiée :

    PermitRootLogin yes  

•	Redémarrage du service SSH :
    
    sudo systemctl restart ssh

•	Avantage de PermitRootLogin no : Meilleure sécurité en forçant l’utilisation d’un utilisateur non-root.
•	Inconvénient de PermitRootLogin yes : Expose root à des attaques par force brute, recommandé uniquement pour des environnements de développement.

1.2 Génération de clés SSH

J’ai généré une paire de clés SSH avec la commande suivante :

    ssh-keygen -t rsa -b 2048

La clé publique a été copiée sur le serveur avec :

    ssh-copy-id root@10.0.2.15

•	Passphrase : Dans un contexte réel, il est conseillé d’utiliser une passphrase pour éviter l’accès non autorisé en cas de vol de la clé privée.

1.3 Connexion avec une clé publique

•	Création du répertoire .ssh et ajout de la clé publique :

    mkdir -p /root/.ssh
    chmod 700 /root/.ssh
    nano /root/.ssh/authorized_keys
    chmod 600 /root/.ssh/authorized_keys

•	Connexion avec la clé privée :

    ssh -i ~/.ssh/id_rsa root@10.0.2.15


1.4 Sécurisation de l’accès SSH

Pour sécuriser l’accès SSH, j’ai désactivé les connexions par mot de passe en modifiant /etc/ssh/sshd_config :

    PermitRootLogin prohibit-password
    PasswordAuthentication no

Cela empêche les attaques de type brute-force, où un attaquant essaie des combinaisons de mots de passe pour accéder au serveur.

2. Étude des processus UNIX

2.1 Affichage des processus

•	Pour lister les processus avec des informations telles que l’utilisateur, le PID, l’utilisation CPU et mémoire :

    ps aux  

•	TIME : Indique le temps CPU total utilisé par le processus.
•	Premier processus : systemd, avec le PID 1.
•	Processus utilisant le plus de CPU : kworker, un processus de gestion des tâches du noyau.

2.2 Afficher les processus avec leurs processus parents

•	Commande pour afficher le PID et le PPID :

    ps -o pid,ppid,cmd

2.3 Visualisation avec pstree

Pour afficher l’arborescence des processus, j’ai utilisé :

    sudo apt install pstree
    pstree

2.4 Utilisation de top et htop

•	top permet de surveiller les processus en temps réel, triés par utilisation mémoire ou CPU. J’ai utilisé la touche ? pour voir l’aide et Shift + P pour trier par CPU.
•	htop offre une interface plus visuelle, facilitant la navigation et l’arrêt de processus via les flèches et F9.

3. Arrêt d’un processus

3.1 Exécution des scripts en arrière-plan

J’ai exécuté deux scripts d’affichage de l’heure :

1.	Script date.sh :

        #!/bin/sh
        while true; do sleep 1; echo -n 'date: '; date +%T; done

	2.	Script date-toto.sh :

   	        #!/bin/sh
            while true; do sleep 1; echo -n 'toto: '; date --date '5 hour ago' +%T; done

Les deux scripts ont été lancés en arrière-plan avec :

    ./date.sh &
    ./date-toto.sh &

3.2 Gestion des processus avec jobs, fg et kill

 Liste des tâches en arrière-plan :

     jobs

Pour ramener un processus à l’avant-plan et l’arrêter avec CTRL+C :

    fg %1
    
J’ai également arrêté les processus avec leur PID :

    ps aux | grep date.sh
    kill <PID>

4. Utilisation des pipes (tubes)

Les pipes permettent de rediriger la sortie d’une commande vers une autre. Quelques exemples :

•	Lister les fichiers et afficher avec cat :

    ls | cat

•	Enregistrer la sortie dans un fichier :

    ls -l | cat > liste

•	Afficher et enregistrer simultanément avec tee :

    ls -l | tee liste

•	Compter les lignes avec wc -l :

    
    ls -l | tee liste | wc -l

5. Journal système : rsyslog

5.1 Vérification du service rsyslog

•	Pour vérifier que rsyslog est actif :

    sudo systemctl status rsyslog

•	PID du démon : 993 dans ce cas.

5.2 Configuration et consultation des logs

•	Le fichier de configuration principal de rsyslog est situé dans /etc/rsyslog.conf. Les logs standards sont enregistrés dans /var/log/syslog.
•	Suivi des logs en temps réel :
    
    tail -f /var/log/messages

5.3 Service cron

•	Le service cron est utilisé pour automatiser des tâches répétitives. J’ai redémarré le service avec :

    sudo systemctl restart cron

Le fichier /etc/logrotate.conf est utilisé pour la rotation des logs.

6. Examen des messages du noyau avec dmesg

	•	Modèle de processeur détecté par Linux :

        dmesg | grep -i "cpu"

Résultat : CPU MTRRs all blank - virtualized system.

•	Cartes réseau détectées :
        
        dmesg | grep -i "eth"
