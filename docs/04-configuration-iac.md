1. Pourquoi utiliser Compose plutôt que plusieurs docker run ?

Au lieu de taper manuellement plusieurs commandes longues et complexes à chaque fois, Compose permet de décrire toute l'infrastructure (services, réseaux, volumes) dans un seul fichier lisible (compose.yaml). Cela rend le projet versionnable, facilement reproductible sur n'importe quel poste, et lançable en une seule commande.


2. Que fait réellement docker compose up -d (listez les objets créés : docker network ls, docker
volume ls, docker ps) ?

La commande docker compose up -d lit le fichier compose.yaml et orchestre le déploiement complet de l'infrastructure en arrière-plan (l'option -d signifiant detached). Elle automatise la création de tous les composants nécessaires au fonctionnement du projet.

Voici les objets spécifiquement créés sur le système :

    - Réseau (docker network ls) : Le réseau virtuel interne helpdesk-prosolo_backend (de type bridge) pour permettre la communication isolée entre les services.   
    - Volume (docker volume ls) : Le volume persistant helpdesk-prosolo_db-data pour stocker durablement les données de la base PostgreSQL, même en cas de redémarrage. 
    - Conteneurs (docker ps) : Les trois conteneurs exécutant les services de l'application :   
        - helpdesk-prosolo-api-1 : exposé sur le port 3000.   
        - helpdesk-prosolo-worker-1 : actif en arrière-plan.   
        - helpdesk-prosolo-db-1 : la base de données, validée par le statut healthy.


3. Quelle diff érence entre « démarré » et « en bonne santé » pour un service ?

C'est la panne que tu viens de réparer ! "Démarré" (Up) signifie simplement que le conteneur est lancé et que le processus principal tourne. "En bonne santé" (healthy) confirme que le service à l'intérieur du conteneur est pleinement initialisé et prêt à recevoir des requêtes.

4. Que se passe-t-il si un service ne passe jamais son healthcheck ? Testez en mettant un mauvais utilisateur
dans pg_isready -U ….

Son statut deviendra (unhealthy). Par conséquent, tous les autres conteneurs qui ont un depends_on avec la condition service_healthy resteront indéfiniment bloqués en attente et ne démarreront jamais.


5. Comment les services communiquent-ils entre eux ?

