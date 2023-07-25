# BoutiqueEngline
GESTION D’ACHATS EN LIGNE.
Dans notre projet ‘Gestion d’achats en ligne’ on a réalisé un mini site
E-Commerce ‘Boutique.net’, une boutique en ligne pour les vêtements pour les différentes âges, sexes et tailles.
Les utilisateurs de ce site sont de deux types : Administrateur ou le chef de la boutique et les clients, pour cela on a créé la table utilisateurs dans laquelle on ajoute l’attribut TypeUtilisateur de type nchar ou chaine de caractères pour différencier entre les utilisateurs, cet attribut peut prendre deux valeurs soit AD pour l’admin et CL pour les clients.

L’admin a des privilèges à partir de son interface Admin comme :
• Consultation un tableau de bord qui contient le nombre des clients, des articles, des nouvelles commandes et des commentaires dans le site.
• L’ajout, la suppression et la modification sur les articles, les comptes clients, les commentaires et les nouvelles commandes.
• Valideation la livraison d’une commande d’un client.

Le client peut :
• Consulter le site.
• Créer un compte client.
• Connecter à son compte.
• Consulter les nouveaux articles(produits) les articles des FEMMES, HOMMES et ENFANTS,
• Ajouter des articles à son panier
• Ajouter des commentaires sur les articles.
• Passer une commande.
• Bénéficier d’une réduction.
• Payer à livraison ou en ligne.

Chaque client a son propre panier dans laquelle il peut ajouter, modifier et supprimer des articles et ce panier reste active jusqu’à que le client pense à passer une commande et ce panier devient inutilisable (désactive) et un nouveau panier est créé pour lui.
Dans l’étape paiement en ligne on a pensé à utiliser un système de paiement en ligne comme PayPal Express Checkout ou Stripe mais ce n’est pas le but du projet donc on suppose que le client a payé sa facture dans ce cas.

Après la confirmation de la commande on ajouter une facture à la table facture et une livraison à la table livraison.
Si le client est choisi le paiement en ligne l’attribut StatusFacture prend la valeur P ça vous dire que la facture est payée(paiement en ligne) ou NP sinon (paiement cash ou à livraison).

Pour chaque commande par défaut est non livré donc on ajoute à sa facture dans l’attribut StatusLivraison la valeur NLiv (non livré) et l’attribut DateLivraison la date de la confirmation de la commande jusqu’à que l’admin confirme la date de la livraison ou le paiement.

Trigger Pour Gérer La Quantité De Stock Des Articles :
On utilise ce trigger pour diminuer la quantité en stock de chaque des articles après la confirmation d’une commande d’un client.
CREATE TRIGGER tr_Gestion_Quantite_stock ON FACTURES AFTER INSERT AS BEGIN DECLARE @IdArticle AS INT, @QtCmd AS INT DECLARE CurQtStock CURSOR LOCAL FAST_FORWARD FOR SELECT IdArticle,QtCmd FROM COMMANDES, inserted WHERE inserted.IdPanier=COMMANDES.IdPanier OPEN CurQtStock FETCH NEXT FROM CurQtStock INTO @IdArticle,@QtCmd WHILE(@@FETCH_STATUS = 0) BEGIN UPDATE ARTICLES SET QtStock=QtStock-@QtCmd WHERE IdArticle=@IdArticle FETCH NEXT FROM CurQtStock INTO @IdArticle,@QtCmd END CLOSE CurQtStock DEALLOCATE CurQtStock END

Trigger pour la réduction :
On a créé ce trigger pour que les clients bénéficient d’une réduction sur le montant des factures :
CREATE TRIGGER tr_Ruduction ON FACTURES AFTER INSERT AS BEGIN /* une reduction de 5% pour les factures avec un montant entre 300 et 3000dhs*/ UPDATE FACTURES SET MontantFacture = 0.95*MontantFacture WHERE IdFacture = (SELECT inserted.IdFacture FROM FACTURES, INSERTED WHERE FACTURES.IdFacture=inserted.IdFacture AND inserted.MontantFacture between 300 and3000 ) /* une reduction de 15% pour les factures avec un montant >3000dhs*/ UPDATE FACTURES SET MontantFacture=0.85*MontantFacture WHERE IdFacture=(SELECT inserted.IdFacture FROM FACTURES, INSERTED WHERE FACTURES.IdFacture=inserted.IdFacture AND inserted.MontantFacture>3000) END
