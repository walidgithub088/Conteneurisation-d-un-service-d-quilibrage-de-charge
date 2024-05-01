Environnement logiciel
1 GNS3
GNS3 (Graphical Network Simulator) est un logiciel open source qui simule des réseaux complexes tout en se rapprochant le plus possible du fonctionnement des réseaux réels. Tout cela sans avoir de matériel réseau physique dédié tel que des routeurs et des commutateurs.
Ce logiciel fournit une interface utilisateur graphique intuitive pour concevoir et configurer des réseaux virtuels, il fonctionne sur du matériel PC traditionnel et peut être utilisé sur plusieurs systèmes d'exploitation, notamment Windows, Linux et MacOS.
Afin de fournir des simulations complètes et précises, GNS3 utilise en fait les émulateurs suivants pour exécuter les mêmes systèmes d'exploitation que dans les réseaux réels.
• Dynamips : l’émulateur de Cisco IOS
• Virtuelbox/Vmware : exécute des systèmes d’exploitation de bureau et de serveur.
• Docker daemon : crée, exécute, tire des centenaires depuis le registre de référentiel docker Hub
• Qemu : un émulateur de machine open source générique, il exécute Cisco ASA, PIX et IPS.
Nous l’équipons avec une GNS3 VM pour des situations lorsqu’on utilise Windows ou Mac OS. L'équipe de développement de GNS3 a créé un moyen léger et robuste de créer des topologies GNS3 qui évite plusieurs problèmes courants rencontrés lors de l'utilisation d'une installation locale de GNS3[34].
Pour réaliser l’architecture du premier scénario nous avons utilisé les programmes suivants tous sous forme de docker containers :
2 Nginx
Nginx est un serveur web open-source qui assure les services suivants :
Proxy inversé avec mise en cache, IPv6, Équilibrage de charge L4 et L7 avec les algorithmes Round Robin, weighetd, IP Hash et ‘Least Connections’, ‘WebSockets’, Gestion des fichiers statiques, des fichiers d’index et de l’indexation automatique, TLS/SSL.
Il a été créé par Igor Sysoev, avec sa première sortie publique en octobre 2004. Igor a d’abord conçu le logiciel comme une réponse au problème de performance liée à la gestion de 10.000 connexions simultanées(C10k).
Nginx surpasse souvent d’autres serveurs web populaires dans les tests de benchmarks, en particulier dans les situations avec un contenu statique et/ou des requêtes simultanées élevées.
Nginx est conçu pour offrir une fiable utilisation de la mémoire et une grande simultanéité. Plutôt que de créer de nouveaux processus pour chaque requête Web, Nginx utilise une approche asynchrone et événementielle où les requêtes sont traitées dans un seul thread [35].
3 Apache benchmark
Ab (apache benchmark) est un utilitaire qui nous permet de tester les performances de notre serveur http. Il a été conçu pour nous donner une idée du degré de performances de notre installation de serveurs. Il nous permet en particulier de déterminer le nombre de requêtes que notre installation est capable de servir par seconde.
La syntaxe nécessaire pour générer des requêtes HTTP est la suivante :
ab -n (le Nombre de requêtes à effectuer au cours du test de performances) -c (Nombre de requêtes à effectuer simultanément. Par défaut, une seule requête est effectuée à la fois) -g (Enregistre toutes les valeurs mesurées dans un fichier 'gnuplot' ou TSV (valeurs séparées par des tabulations)) http[s]://nom- serveur[:port]/chemin[36].
4 Wireshark
Un analyseur de paquets open source (GNU) populaire. Ses décodeurs de protocoles permettent d’interpréter le trafic du réseau.
Conçu en 1997-1998 par Gerald Combs sous le nom historique de “Ethereal”. Il est repris en 2006 sous le nom moderne de “Wireshark”. En 2008, Wireshark sort en version 1.0 et en 2015 en version avec une nouvelle interface graphique. Disponible sur linux et windows [37].
Remarques : tous ces conteneurs sont exécutés sur la machines virtuelle de gns3 qui possède un processeur core virtuel, une mémoire RAM de trois Gb et qui est exécuté sur l’hyperviseur VMware.
Pour réaliser l’architecture du deuxième scénario nous avons utilisé le programme suivant :
5 Bitnami VM
Les machines virtuelles Bitnami contiennent un système d'exploitation Linux minimal avec NGINX, WordPress et SSL installés et configurés. L'utilisation de l'image de la machine virtuelle Bitnami nécessite un logiciel hyperviseur tel que VMware Player ou VirtualBox.
Cette machine virtuelle possède un processeur virtuel, d’un Gb de ram et qui est exécuté sur l’hyperviseur Vmware.


motivation de choix de l’environnement logiciel
⚫ nginx vient sur ses deux formes machine virtuelle (Bitnami VM) et conteneur donc pouvoir comparer ces technologies en terme de performance et optimisation, et peut être un serveur web, un load balancer et un proxy/reverse proxy et peut appliquer des Health checks et du SSL offload (fonctionnalités implémentées par les solutions LB des fournisseurs cloud) alors simuler une application traitée par des serveur web en appliquant du load balencing pour assurer sa disponibilité ce qui forme nos deux scénarios.
⚫ Apache benchmark pour simuler les requêtes HTTP de multiples clients.
⚫ Wireshark pour s’assurer de la bonne distribution des requêtes aux serveurs web par les LB.

Topologie du premier scénario
La topologie proposée comporte deux générateurs de requêtes http l’un visant des images et l’autre de vidéos, un LB externe, deux LB internes, ainsi que deux backends de serveurs web.

![image](https://github.com/walidgithub088/Conteneurisation-d-un-service-d-quilibrage-de-charge/assets/151946258/6b1f80bf-228d-4766-8b55-edaa9ea16ce6)

Pour adresser ces conteneurs nous accédons aux fichiers de configuration (edit conf) sur l’interface de GNS3.
Nous citons les images dockers suivantes :
• ajnouri-ab les générateurs de requêtes HTTP avec les adresses 177.177.177.2/24 et 177.177.177.3/24.
• lbext-1 est le load balancer externe avec deux interfaces, l’interface eth0 avec l’adresse 177.177.177.1/24 destinée aux générateurs (clients) et l’interface eth1 avec l’adresse 10.10.10.254/24,
• lbint1-1 est le load balancer interne destiné à la première région /Images avec l’adresse 10.10.10.253/24 pour l’interface eth0, 10.10.9.1/24 pour eth1.
• lbint2-1 est le load balancer interne destiné à la deuxième région /Videos avec l’adresse 10.10.10.252/24 pour l’interface eth0, 10.10.8.1/24 pour eth1.
• les images ajnouri-nginx représentent nos serveurs web, trois pour chaque région, chaque serveur considère l’interface interne des équilibreurs de sa région comme une passerelle.


Configuration des conteneurs équilibreurs de charge
Pour construire une image docker nginx pour être un load balancer nous avons procédé ainsi :
⚫ ouvrir le shell de la gns3 VM
⚫ créer un fichier nommé nginx.conf où nous introduisons les configurations nécessaires
⚫ vu que c’est un load balancer de couche 7 on définit d’abord un contexte http
⚫ dedans nous créons deux autres contextes, le premier appelé upstream pour définir notre backend de serveurs suivi du nom de ce backend, chaque serveur est défini par l’attribut server suivi par son adresse IP et le port sur lequel il écoute le trafic,
⚫ pour appliquer des ‘health checks’ sur ces serveurs nous utilisons l’attribut Max_fails (le nombre maximum de connexions échouées avant que le serveur est marqué indisponible) et Fail_timeout (le temps pour lequel le nombre spécifié de tentatives de communication échouées avec le serveur doit se produire), ainsi que le nom de l’algorithme à appliquer (par défaut Round Robin).
⚫ Le deuxième contexte appelé server pour définir le serveur virtuel vu par les clients, avec l’attribut listen pour dire sur quel numéro de port l’équilibreur attend le trafic (80,8080 ou 443 pour le L7), et le contexte Location / (le path a consulté par l’équilibreur pour appliquer le routage avancé), ce contexte contiendra l’attribut Proxy_pass http:// nom-du-backend; pour envoyer les requêtes à l’upstream correspondantes ace path.
⚫ Après cela nous créons un dockerfile que nous utilisons pour construire cette image et intégrer notre configuration ce fichier aura la forme :
• FROM nomdu docker image
• COPY fichier-contenant-la-config chemain-de-ce-ficher-dans-l ’image-officielle-de-ce- conteneur (figure 16)
⚫ Et en fin nous avons qu’à lancer la commande Docker build -t nom-du-conteneur, et qui va chercher un dockerfile sur notre machine et faire un PULL de cette image du docker repository publique et intégrer nos configurations.
Pour nos équilibreurs nous avons défini pour l’équilibreur externe deux contextes location un pour les requêtes demandant des images (/images) et l’autre pour des vidéos (/vidéos) qui passe en proxy_pass au backend correspondant soit pour le lbint1 ou lbint2 (figure 15).
Pour la 2éme région nous avons défini du weighted Round Robin tel que le serveur d’adresse 10.10.8.4 (weight=3) recevra trois fois plus de requêtes que 10.10.8.2 et le serveur 10.10.8.3 (weight=2) le double du nombre de requêtes transmises au 10.10.8.2 (figure 14).
Tous nos équilibreurs écoutent le trafic sur le port 80 vu que c’est du HTTP donc on définit l’attribut Listen 80.
Voici les fichiers nginx.conf correspondants à nos équilibreurs de charge :
![image](https://github.com/walidgithub088/Conteneurisation-d-un-service-d-quilibrage-de-charge/assets/151946258/257c9ac5-0dcc-4079-b570-40c97197f22c)
![image](https://github.com/walidgithub088/Conteneurisation-d-un-service-d-quilibrage-de-charge/assets/151946258/f2053dfd-de83-4419-bcb1-68cd8b1702bf)
![image](https://github.com/walidgithub088/Conteneurisation-d-un-service-d-quilibrage-de-charge/assets/151946258/8a4f5491-7376-4645-8eec-75ea59a57854)
Ensuite, pour chaque conteneur nous créons le dockerfile et nous construisons notre image :
![image](https://github.com/walidgithub088/Conteneurisation-d-un-service-d-quilibrage-de-charge/assets/151946258/c357675e-e3fc-4780-9aec-2d4aa3b31843)
![image](https://github.com/walidgithub088/Conteneurisation-d-un-service-d-quilibrage-de-charge/assets/151946258/63b2e48b-2d85-423c-83be-a0fe5be5fbf5)


Déroulement du scénario
Nous comptons d’abord lancer un des deux générateurs en envoyant des requêtes de type 177.177.177.1/images/test.php ensuite 177.177.177.1/videos/test.php et pour surveiller la distribution de la charge selon le path nous lançons wireshark sur les liens entre l’interface interne du lbext et les deux équilibreurs internes.
Nous lançons 100 requêtes visant les images donc le lbext va choisir comme adresse ip destination 10.10.10.253 et l’autre équilibreur ne recevra aucune requête. Donc sur le générateur ajnouri-ab-1 nous appliquons la commande ab -n 100 177.177.177.1/images/test.php
Et nous lançons la capture de paquet avec un filtre de ip source 10.10.10.254 et protocole http et voici une capture du résultat :
![image](https://github.com/walidgithub088/Conteneurisation-d-un-service-d-quilibrage-de-charge/assets/151946258/fd4f0f28-7675-43e5-b47c-8bc4b5363817)
Nous remarquons qu’aucune requête n’est envoyer à l’équilibreur lbint2-1 (10.10.10.252) donc nous concluons que le routage avancé (de couche 7) est fonctionnel.
Pour tester la distribution dans les régions on applique la même méthode mais cette fois en lançant les deux générateurs en même temps et faire une capture de paquets entre l’interface interne des lbint et les serveurs web.
Les figures 19 et 20 montrent les résultats de la capture pour 100 requêtes envoyées des deux générateurs. Pour la capture de round robin on voit bien que l’équilibreur boucle sur le backend en envoyant une requête à un serveur et passe à l’autre.
![image](https://github.com/walidgithub088/Conteneurisation-d-un-service-d-quilibrage-de-charge/assets/151946258/4ae5a855-51d6-4da7-9f86-7e12c1313560)
Mais pour la région 2 ce n’est pas le cas, on envoie 3 fois plus de requêtes au serveur d’adresse 10.10.8.4 et deux fois plus à 10.10.8.3 qu’au serveur 10.10.8.2 (figure 20).
![image](https://github.com/walidgithub088/Conteneurisation-d-un-service-d-quilibrage-de-charge/assets/151946258/e9fda0bd-5937-4cc3-af0c-876677437aff)
En se basant sur ces captures, nous avons obtenu les diagrammes suivants :
Pour la première zone selon la figure 21, tous les serveurs ont reçu ‘presque‘ le même nombre de requêtes (puisque nous avons envoyé 100 requêtes distribués sur 3 serveurs) alors le premier serveur qui possède l’adresse ip 10.10.9.2 a reçu 34 requêtes, le deuxième et le troisième ont tous les deux reçu 33 requêtes.
![image](https://github.com/walidgithub088/Conteneurisation-d-un-service-d-quilibrage-de-charge/assets/151946258/8185ee35-09aa-4c7f-9a04-93a53c837774)
Par-contre selon la figure 22, dans la deuxième région, chaque serveur a reçu un nombre de requêtes différent du nombre de requêtes reçues des autres serveurs à cause des poids différents associés à chaque’un d’eux.
![image](https://github.com/walidgithub088/Conteneurisation-d-un-service-d-quilibrage-de-charge/assets/151946258/b35f4d86-76cf-40c9-a990-da5057389bf6)
Le serveur qui possède l’adresse 10.10.9.4 reçoit plus de requêtes par rapport aux deux autres serveurs car il possède un poids (weight = 3) et donc il reçoit les requêtes trois fois de plus par rapport au serveur configuré avec le poids (weight =1).
Le serveur configuré avec l’adresse ip 10.10.8.3 et avec le poids (weight=2) reçoit le double de ce que reçoit le serveur qui possède l’adresse10.10.8.2 et qui est configuré avec le poids (weight=1).


Topologie du deuxième scénario
La topologie réalisée comporte un générateur de requêtes, deux équilibreurs de charge l’un sous forme de VM et l’autre de conteneur et un backend de quatre serveurs web.
![image](https://github.com/walidgithub088/Conteneurisation-d-un-service-d-quilibrage-de-charge/assets/151946258/3a1bfae3-3a65-4d93-859c-94b8f0dbfc9e)
Configurations des deux équilibreurs de charge
La VM bitnami-nginx possède un OS Linux Debian et gns3 VM possède un OS Linux Ubuntu. Les deux VMs ( la vm qui exécute le conteneur LB et la vm bitnami-nginx sont dotés de la même puissance ( 1Gb de RAM, 2 vCPU).
Pour configurer les adresses IP des interfaces de la VM bitnami-nginx nous avons qu’à :
⚫ accéder au fichier /etc/network/intefaces en tant qu’administrateur
⚫ saisir les adresse de nos deux interfaces
⚫ redémarrer le service networking (sur Debian avec sudo /etc/init.d/networking restart).
Les deux types d’équilibreurs appliquent du Round Robin sur les quatre serveurs web, pour configurer le fichier nginx.conf sur la VM bitnami-nginx nous accédons au chemin /etc/nginx/nginx.conf.
Nous allons basculer entre les deux équilibreurs en branchant à tour de rôle celui à tester, le teste consiste à envoyer progressivement des requêtes HTTP du générateur (clic droit sur le conteneur, Aux console pour accéder au CLI et saisir la commande ab -n c --- 10.10.10.1/test.php) et à chaque envoi relever trois métriques, la quantité de RAM et CPU consommé par le processus d’équilibrage de charge en utilisant la commande Top -pid (figure 24), ainsi que le temps pris pour traiter toutes les requêtes depuis les statistiques fournies par le générateur Apache Benchmark sur son terminal après la fin du teste «Time taken for tests» (figure 25).
On augmente le nombre de requêtes (-n) chaque fois de 5000 avec une concurrence (-c) de 100 requêtes envoyées à la fois, nous avons remarqué que pour ces valeurs nous aurons des résultats plus constants en termes de CPU et de durée de teste pour pouvoir les relever, nous avons obtenu les valeurs de cpu fournies en calculant la moyenne de valeurs affichées par la commande top -pid chaque seconde.
Ci-dessous un exemple de l’affichage des valeurs de nos trois métriques.
![image](https://github.com/walidgithub088/Conteneurisation-d-un-service-d-quilibrage-de-charge/assets/151946258/31b0bb92-3e63-4ff3-b171-7157e6366e9e)
![image](https://github.com/walidgithub088/Conteneurisation-d-un-service-d-quilibrage-de-charge/assets/151946258/07cad244-885e-4c16-9f0c-22745c2f822d)


Estimation des performances
Une comparaison des statistiques des résultats obtenus entre les deux technologies.
La figure 26 représente le pourcentage de consommation du CPU par le processus qui s’exécute sur le conteneur.
![image](https://github.com/walidgithub088/Conteneurisation-d-un-service-d-quilibrage-de-charge/assets/151946258/4db8d28a-e73d-46e3-9578-5156d5cf57d5)
La figure 27 représente le temps en secondes pris pour l’envoi des requêtes et la réception des réponses HTTP par le processus qui s’exécute sur le conteneur.
![image](https://github.com/walidgithub088/Conteneurisation-d-un-service-d-quilibrage-de-charge/assets/151946258/9ab5c5cf-0e53-4721-80f9-e2c6f0cf7059)
La figure 28 nous donne le pourcentage de consommation de RAM par le processus qui s’exécute sur le conteneur.
![image](https://github.com/walidgithub088/Conteneurisation-d-un-service-d-quilibrage-de-charge/assets/151946258/d9e2181f-3fb9-451e-83c2-1975f4e2153c)
La figure 29 nous donne le pourcentage de consommation de CPU par le processus qui s’exécute sur la VM bitnami-nginx.
![image](https://github.com/walidgithub088/Conteneurisation-d-un-service-d-quilibrage-de-charge/assets/151946258/d5a80697-6c52-4198-8cce-0ab17a575116)
La figure 30 représente le temps pris pour l’envoi des requêtes et la réception des réponses HTTP par le processus qui s’exécute sur la VM bitnami-nginx.
![image](https://github.com/walidgithub088/Conteneurisation-d-un-service-d-quilibrage-de-charge/assets/151946258/9e9f9948-df93-48d8-a2ef-e3c3d491ed34)
Et enfin, le graphe de consommation de RAM par la VM.
![image](https://github.com/walidgithub088/Conteneurisation-d-un-service-d-quilibrage-de-charge/assets/151946258/bb5644cd-79a1-4ed5-a855-eb14d0fe4c42)
Ainsi nous affirmons nos hypothèses, la consommation de CPU varie entre 20.1% et 22.2% alors que la VM 29.8% et 33.3%, la consommation de RAM bascule entre 0,3% et 0,4% pour le conteneur, 0,4% et 0,5% pour la VM, et le maximum de temps pris pour le test est de 191,3s pour le conteneur et 224,7s pour la VM.


Conclusion
Nous avons effectué dans ce chapitre l’implémentation et la simulation de nos deux scénarios présentés, également nous avons démontrer que la consommation des ressources est plus optimale en déployant un conteneur qu’avec une machine virtuelle, ce qui conclue notre projet.
