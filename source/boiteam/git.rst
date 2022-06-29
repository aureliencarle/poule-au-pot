############################################
Quelques petites commandes prêtes à l'emploi
############################################


Déplacer un tag
===============

Si durant un developpement un tag a été posé un peu précipitamment. Par exemple on découvre qu'il fallait synchroniser la version du tag avec la version du code dans un obscure CMakeFile (toute ressemblance avec un evenement existant est parfaitement fortuit).
Il faudra se munir du tag que l'on veut bouger (``<tagname>``) et le sha1 du commit que l'on veut tagger (``<commit_sha1>``).

.. code-block:: console

    git tag -d <tagname>                  # delete the old tag locally
    git push origin :refs/tags/<tagname>  # delete the old tag remotely
    git tag <tagname> <commit_sha1>       # make a new tag locally
    git push origin <tagname>             # push the new local tag to the remote



.. note ::
    Les 3 paragraphes suivants sont des traductions des méthodes porposées par les utilisateurs StackOverflow Cascabel et Joel Hoisko.


Passer temporairement à un autre commit
=======================================

Si on veut temporairement revenir en arrière, faire des folies, puis revenir là où vous êtes, tout ce que vous avez à faire est de vérifier le commit désiré :

.. code-block:: console

    git checkout 0d1d7fc32 # This will detach your HEAD, leave you with no branch checked out

Ou si on veut faire des commits pendant qu'on y est, go créer une nouvelle branche `old-state` par exemple :

.. code-block:: console

    git checkout -b old-state 0d1d7fc32

Supprimer définitivement les commits non publiés
================================================

Si, d'un autre côté, on veut vraiment se débarrasser de tout ce qu'on a fait depuis, il y a deux possibilités. La première, si aucun de ces commits n'a été publié, il suffit de réinitialiser :

.. code-block:: console

    git reset --hard 0d1d7fc32      # This will destroy any local modifications.

Autrement, si il y a du travail à conserver :

.. code-block:: console

    git stash                       # This saves the modifications.
    git reset --hard 0d1d7fc32
    git stash pop                   # Then reapplies that patch after resetting.


Undo published commits with new commits (Sorcellerie)
=====================================================

On the other hand, if you've published the work, you probably don't want to reset the branch, since that's effectively rewriting history. In that case, you could indeed revert the commits. With Git, revert has a very specific meaning: create a commit with the reverse patch to cancel it out. This way you don't rewrite any history.

.. code-block:: console

    # This will create three separate revert commits:
    git revert a867b4af 25eee4ca 0766c053

    # It also takes ranges. This will revert the last two commits:
    git revert HEAD~2..HEAD

    # Similarly, you can revert a range of commits using commit hashes (non inclusive of first hash):
    git revert 0d1d7fc..a867b4a

    # Reverting a merge commit
    git revert -m 1 <merge_commit_sha>

    # To get just one, you could use `rebase -i` to squash them afterwards
    # Or, you could do it manually (be sure to do this at top level of the repo)
    # get your index and work tree into the desired state, without changing HEAD:
    git checkout 0d1d7fc32 .

    # Then commit. Be sure and write a good message describing what you just did
    git commit

The `git-revert` manpage actually covers a lot of this in its description. Another useful link is this git-scm.com section discussing git-revert.
If you decide you didn't want to revert after all, you can revert the revert (as described here) or reset back to before the revert (see the previous section).