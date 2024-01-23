# Sécurité des réseaux avancés

Par [Mathis Boultoureau](https://github.com/mboultoureau/) et [Ronan Renoux](https://github.com/ronanren).

## Prise en main de l'environnement de tests

**Q1. Combien de réseaux sont actuellement créés ? Quels sont les différents types de réseaux listés par docker ?**

```bash
`sudo docker network ls
```

Il y a 4 réseaux listés :

| Nom du réseau | Type | Détail |
| --- | --- | --- |
| bridge | bridge | Réseau par défaut. Réseau isolé de l'hôte mais accessible depuis les conteneurs dans le même réseau. |
| exo1_net_pub | bridge | Réseau isolé de l'hôte mais accessible depuis les conteneurs dans le même réseau. |
| host | host | Même réseau que l'hôte sans aucune couche d'isolation |
| none | null | Réseau spécifique à chaque conteneur complétement isolé de l'hôte et des autres conteneurs |

**Q2. Quelle est l’adresse IP de la Busybox ?**

```bash
sudo docker inspect --type=network exo1_net_pub
```

L'adresse IP de la Busybox est 192.168.20.2/24.

Le réseau exo1_net_pub est sur l'adresse IP 192.168.20.0/24.

| Nom du conteneur | Adresses IP |
| --- | --- |
| exo1_web1_1 | 192.168.20.11/24 |
| exo1_web2_1 | 192.168.20.12/24 |
| exo1_busy1_1 | 192.168.20.2/24 |

```bash
sudo docker exec -it exo1_busy1_1 sh
ifconfig
```

Avec la commande *ifconfig*, nous avons bien l'ip du conteneur exo1_busy1_1 qui est 192.168.20.2.

**Q3. Que constatez-vous ? Quelle fonctionnalité est mise en avant en faisant ce test ?**

Toujours depuis la CLI du conteneur Busybox, nous lançons des pings vers les conteneurs *web1* et *web2* avec leurs IP et noms :

```bash
# Web1
ping web1
ping 192.168.20.11

# Web2
ping web2
ping 192.168.20.12
```

Nous constatons que le ping fonctionne que ça soit avec les adresses IP ou avec les noms. Docker propose un DNS qui par défaut lie les noms des conteneurs à leur IP (au sein du même réseau).

**Q4. A partir de l’inspection des streams TCP/HTTP, quelles sont les informations notables dans les entêtes HTTP échangées avec les deux serveurs web ? Est-ce qu’il y une différence entre les deux serveurs Web ? Quels sont les codes de retours HTTP et qu’indiquent-ils ? Quelle est la réponse bonus ?**

Nous regardons la liste des interfaces réseaux disponibles et la comparons avec la liste des réseaux Docker :

```bash
sudo docker network ls
sudo tcpdump -D
```

![Liste des réseaux Docker et tcpdump](getting_started/img/network_list_tcpdump_docker.png)

Nous pouvons lancer une capture réseau sur l'interface réseau qui correspond (en comparant les identifiants de réseau) et l'enregister dans le fichier `dump.pcap` (disponible dans `getting_started/files/dump.pcap`).

```bash
sudo tcpdump -w dump.pcap -i br-aaf49bcbc957
```

Nous analysons la capture d'écran avec Wireshark :

- Pour chaque site, le client envoie une requête avec la procotole et version (HTTP 1.1), la page demandé (/), la méthode (GET), l'user Agent, etc.
- Dans la réponse, le serveur renvoit la date, la date de modification, le code HTTP (ici 200), le protocole et version utilisé (HTTP 1.1), le type et la version du serveur, etc.

Le serveur *web1* est un serveur Nginx (1.25.3) tandis que le serveur *web2* est un serveur Apache (2.4.58).

![Capture Wireshark web1](getting_started/img/wireshark_web1.png)

![Capture Wireshark web2](getting_started/img/wireshark_web2.png)

Les codes de retours HTTP sont :
- 200 : la ressource est bien trouvé
- 404 : la ressource n'est pas trouvé (le navigateur par défaut demande `favicon.ico` qui n'est pas sur le serveur)

La réponse bonus est un kangouru (à la question `Who is Skippy?`).

## Mise en place d'un reverse proxy simple basé sur NGINX

**Q5. A**