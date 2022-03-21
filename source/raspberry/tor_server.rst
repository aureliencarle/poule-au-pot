########################################
Deployer un site .onion sur Raspberry pi
########################################

.. note ::
    Le text qui suit et une adaptation d'un tuto proposé par ZeR0-@bSoLu 
    pour kali [#]_.

.. [#] https://www.kali-linux.fr/configuration/heberger-son-site-sous-tor


Les prérequis
=============

Ce tuto a été vérifié sur une Raspberry Pi 3B+ avec Raspbian GNU/Linux 11 armv :

.. figure:: /images/raspberry/raspberry_neofetch.png
   :width: 600
   :align: center

|

Nous aurons besoin de trois choses : 
    #. **python3**  (à partir de maintenant je dis python parce que bon ... j'écris ces lignes en 2022...)
    #. **tor**
    #. **TorBrowser** (pour le tester)

Pour l'installation, python est certainement déja installé donc il nous reste qu'à installer les outils 
de tor :

.. code-block :: console

    $ sudo apt-get update -y 
    $ sudo apt-get install torbrowser-launcher tor 


.. attention ::
    Au moment où je fais ce tuto, il n'y a pas de release officiel tor pour Raspian, on utilise donc le 
    tor du repo... Quand la release sera disponible, il est conseillé d'installer tor par le dépot 
    fourni directement par le torproject : https://support.torproject.org/apt/   


Pour éviter tout problèmes et travailler proprement on va s'assurer que tor ne s'est pas mis en route
à l'installation (c'est du vécu...).

.. code-block :: console

    $ sudo service tor stop


.. note ::
    Pour manipuler les élements coté serveur il faut les droits administrateur. Soit vous pouvez passer
    root soit balacer du sudo dans tout les sens. Dans le suite du tuto je serais root :

    .. code-block :: console

        $ sudo su 

    Et zé bartiiii !!!!

Création du site web
=====================

Pour commencer, nous allons avoir besoin d’un site web. 
Dans le dossier ``/var/www/`` on va creer un dossier dans lequel on va faire un site très simple, mais on 
va le faire avec un css et tout et tout... 

Les clichés sur le réseau tor l'impose, nous allons faire un message couleur sang sur fond noir ...
Et oui c'est le darknet .. ça fait peur ...

.. code-block :: console

    $ mkdir /var/www/onion_frit
    $ touch /var/www/onion_frit/index.html
    $ touch /var/www/onion_frit/style.css

Remplissons le fichier ``index.html`` avec un code simple et le fichier ``style.css`` 
avec les couleurs de satan :

.. code-block :: html
    :linenos:
    :caption: /var/www/onion_frit/index.html

    <!-- index.html -->
    <head>
        <link rel="stylesheet" href="style.css" />
    </head>
    <html>
        <body>
            Hello Anonymous World !
        </body>
    </html>

.. code-block :: css
    :linenos:
    :caption: /var/www/onion_frit/style.css

    body {
        background-color: black;
    }

    h1 {
        color: red;
        display: flex;
        flex-direction: column;
        justify-content: center;
        text-align: center;
    }

Normalement ça ressemble à ça :

.. figure:: /images/raspberry/tor_site_apparence.png
   :width: 600
   :align: center

|

En fait c'est pas "normalement"... c'est exactement ça.

Création du serveur avec Python
===============================

En suite nous allons démarrer notre serveur à l’aide de python et de la librarie ``http.server``. 
A ce stade il faut comprendre que deux ports sont en jeux. Le premier, le port ``80`` est le port par
lequel tor va écouter les requêtes du monde exterieur. Le second est celui que va écouter notre 
serveur python et avec lequel tor va communiquer. On a choisi dans ce tuto le port ``5631``.

On a alors deux choix totalement équivalents :

Choix 1 : Appeler le module python en ligne de commande
"""""""""""""""""""""""""""""""""""""""""""""""""""""""

Pour se faire on va lancer la commande : 

.. code-block :: console

    $ python3 -m http.server --directory /var/www/onion_frit/ --bind 127.0.0.1 5631
    Serving HTTP on 127.0.0.1 port 5631 (http://127.0.0.1:5631/) ...

Quelques explications : 
    #. ``--directory /var/www/onion_frit/`` : indique le dossier où trouver votre index.html
    #. ``--bind 127.0.0.1`` : Permet notamment de ne pas être référencé par Shodan lorsque nous connecterons notre serveur à Tor par la suite. 

----------

Choix 2 : Ecrire un script python (un poil plus complexe si on début en python)
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

L'écriture d'un joli script ``tor_simple_server.py`` où l'on désire :

.. code-block :: python
    :caption: tor_simple_server.py

    #!/usr/bin/env python3
    # -*- coding: utf-8 -*-

    import http.server
    import socketserver

    PORT = 5631
    DIR  = '/var/www/onion_frit/'

    # Héritage de la class SimpleHTTPRequestHandler pour avoir accès au paramètre 'directory'.
    class Handler(http.server.SimpleHTTPRequestHandler):
        def __init__(self, *args, **kwargs):
            super().__init__(*args, directory=DIR, **kwargs)

    with socketserver.TCPServer(('', PORT), Handler) as httpd:
        print('Serving at port', PORT)
        httpd.serve_forever()

On rend executable le script puis on le lance avec : 

.. code-block :: console    

    $ chmod +x tor_simple_server.py
    $ ./tor_simple_server.py
    Serving at port 5631


.. note ::
    Le serveur doit être actif lors de toute la session. 
    Il est judicieux de le faire tourner en fond ou d'ouvrir un nouveau terminal pour la suite.

Configuration de Tor
====================

Maintenant nous allons modifier le fichier ``/etc/tor/torrc`` avec 
votre éditeur de texte préféré (donc vim). Decommentez les lignes parametrant HiddenServiceDir et
HiddenServicePort et modifiez le ServicePort avec le port que vous aurez choisi pour votre serveur 
python précédemment lancé.

.. code-block :: nginx
    :caption: /etc/tor/torrc
    :lineno-start: 61
    :emphasize-lines: 11-12

    [...]
    ############### This section is just for location-hidden services ###

    ## Once you have configured a hidden service, you can look at the
    ## contents of the file ".../hidden_service/hostname" for the address
    ## to tell people.
    ##
    ## HiddenServicePort x y:z says to redirect requests on port x to the
    ## address y:z.

    HiddenServiceDir /var/lib/tor/hidden_service/
    HiddenServicePort 80 127.0.0.1:5631

    #HiddenServiceDir /var/lib/tor/other_hidden_service/
    #HiddenServicePort 80 127.0.0.1:80
    #HiddenServicePort 22 127.0.0.1:22

    ################ This section is just for relays #####################
    [...]

Dans mon fichier, ligne 72 : ``HiddenServicePort 80 127.0.0.1:5631``, on remarque le nombre ``80``. Il s'agit du 
port par lequel on va communiquer avec la Raspberry Pi depuis l'exterieur. Il est différent du port ``5631`` 
que l'on a choisi précédemment. 

----------------

Nous y sommes presque ! Il ne reste plus qu’a lancer Tor, 
pour se faire lancez simplement la commande :

.. code-block :: console

    $ service tor start

Normalement une fois ces lignes affichées tout est prêt !

Il nous reste plus qu’a récupérer l’adresse de notre serveur sous Tor, 
pour ce faire il suffit d’aller lire le fichier hostname dans le path suivant 
(si vous n’avez pas modifié le path de la ligne 71 du fichier ``/etc/tor/torrc`` vu précédemment, 
le cas contraire rendez-vous dans le path spécifié) : 

.. code-block :: console

    $ cat /var/lib/tor/hidden_service/hostname
    quandcestgenerecommecaonamoinslimpressionquecestdiaboliqueledarknet.onion

Cet URL est généré aléatoirement par Tor, il est possible de la changer mais nous ne couvriront pas ce cas 
précis dans ce tuto. Ceci est donc l’adresse de votre site sous Tor, c’est l’URL.


Accès depuis l'exterieur
========================

C'est bien beau mais maintenant il faut y acceder depuis l'exterieur (de votre reseau local je veux dire).


La première chose à faire c'est un scan des adresses IP du réseau sur lequel est connéctée la Raspberry. 
Il y a moulte manières de faire, moi je vais utiliser la commande ``ifconfig`` pour récupérer l'adresse IP 
locale. 

.. code-block :: console
    :emphasize-lines: 19

    $ ifconfig
    eth0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
            ether b8:27:eb:bf:fc:65  txqueuelen 1000  (Ethernet)
            RX packets 0  bytes 0 (0.0 B)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 0  bytes 0 (0.0 B)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

    lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
            inet 127.0.0.1  netmask 255.0.0.0
            inet6 ::1  prefixlen 128  scopeid 0x10<host>
            loop  txqueuelen 1000  (Local Loopback)
            RX packets 191  bytes 152057 (148.4 KiB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 191  bytes 152057 (148.4 KiB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

    wlan0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
            inet 192.168.1.5  netmask 255.255.255.0  broadcast 192.168.1.255
            inet6 fe80::997a:2174:4155:d5e3  prefixlen 64  scopeid 0x20<link>
            ether b8:27:eb:ea:a9:30  txqueuelen 1000  (Ethernet)
            RX packets 60027  bytes 18804738 (17.9 MiB)
            RX errors 0  dropped 1  overruns 0  frame 0
            TX packets 23390  bytes 8768745 (8.3 MiB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

Comme ma Raspberry est connectée en WiFi, je recupère l'adresse IP dans la section ``wlan0``. J'obtient donc 
l'adresse : ``192.168.1.5``.

Maintenant, l'idée est de rediriger toutes les requêtes pour l'adresse IP de ma box vers ma Raspberry.z


Lançons le serveur
==================

Si ce n'est pas déja fait on lance tout ! : 

.. code-block :: console    

    $ service tor start
    $ python3 -m http.serveur --directory /var/www/onion_frit/ --bind 127.0.0.1 5631

Et bah c'est bon... Bravo vous venez de déployer un site .onion sous Tor !

.. image:: /images/tropcool.jpg
   :width: 600
   :align: center

|

On va tester pour être sûr du coup ... (les québecois ont compris en une phrase que je suis français)
=====================================================================================================

Finalement, depuis un autre ordinateur lancez TorBrowser et rendez vous sur l’URL.

.. figure:: /images/raspberry/tor_connection.png
   :width: 600
   :align: center

|

.. figure:: /images/transpi.gif
   :width: 600
   :align: center

|

.. figure:: /images/raspberry/tor_connected.png
   :width: 600
   :align: center

|

Bravo ça marche !!!

Vous devriez voir également une requête sur votre serveur python ! Du style : 

.. code-block :: console
    :emphasize-lines: 3-6

    $ python3 -m http.serveur --directory /var/www/onion_frit/ --bind 127.0.0.1 5631
    Serving HTTP on 127.0.0.1 port 5631 (http://127.0.0.1:5631/) ...
    127.0.0.1 - - [08/Mar/2022 15:32:55] "GET / HTTP/1.1" 200 -
    127.0.0.1 - - [09/Mar/2022 15:44:24] "GET /style.css HTTP/1.1" 200 -
    127.0.0.1 - - [08/Mar/2022 15:32:55] code 404, message File not found
    127.0.0.1 - - [08/Mar/2022 15:32:55] "GET /favicon.ico HTTP/1.1" 404 -

L'erreur 404 proviens de l'absence de favicon (l'icon du site web), pour ce tuto c'est pas bien grave...

