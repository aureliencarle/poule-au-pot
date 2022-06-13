############################################
Quelques petites commandes prêtes à l'emploi
############################################


Déplacer un tag
===============

Si durant un developpement un tag a été posé un peu précipitamment. Par exemple on découvre qu'il fallait synchroniser la version du tag avec la version du code dans un obscure CMakeFile (toute ressemblance avec un evenement existant est parfaitement fortuit).
Il faudra se munir du tag que l'on veut bouger (``<tagname>``) et le sha1 du commit que l'on veut tagger (``<commit_sha1>``).

.. code-block :: console

    git tag -d <tagname>                  # delete the old tag locally
    git push origin :refs/tags/<tagname>  # delete the old tag remotely
    git tag <tagname> <commit_sha1>       # make a new tag locally
    git push origin <tagname>             # push the new local tag to the remote