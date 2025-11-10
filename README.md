# info001-tp1

* Mema Entela
* Rey Guillaume

QUESTION 1
Soit un message M dont on souhaite assurer la confidentialité, rappeler le calcul réalisé
pour chiffrer ce message en utilisant RSA. On notera C le message chiffré. Votre calcul doit faire intervenir les paramètres trouvés à la question 1. Rappeler comment l’opération inverse
(déchiffrement) est réalisée.

RSA repose sur deux clés :
Clé publique : (n,e)(n, e)(n,e)
Clé privée : (n,d)(n, d)(n,d)
Chiffrement : C = M^e mod n
Déchiffrement : M = C^d mod n


QUESTION 2
Rappeler le rôle de la méthode Diffie-Hellman.

L’algorithme Diffie-Hellman est un algorithme d’échange de clés, utilisé notamment lors de l’ouverture d’une connexion à un site sécurisé via le protocole SSL/TLS.
L’objectif de Diffie-Hellman est de permettre l’établissement d’une clé privée entre deux parties, via l’échange de messages sur un canal non sécurisé. Lors de l’établissement d’une clé avec Diffie-Hellman, les messages sont en effet envoyés en clair sur le réseau, et toute personne qui intercepte les messages transmis ne doit pas pouvoir en déduire la clé générée.
La confidentialité est garantie par le fait qu’un éventuel attaquant, qui intercepterait les communications entre Alice et Bob, n’aurait aucun moyen de retrouver la clé privée à partir des informations transmises publiquement. x et y étant de très grands nombres, il est en effet extrêmement complexe de retrouver leur valeur à partir des informations transmises en clair : p, g, P1 et P2. Et sans x ni y, il est impossible de retrouver la clé finale.

QUESTION 3 
Donner 4 informations importantes contenues dans un certificat.

"Subject Alternative Name"  : les noms domaines pour qui le certificat a été généré (ex: www.amazon.fr).
La signature du certificat:  (ex: 11:a1:61:66:66:1f:b6:70:42:50:87:bd:23:23:67:08:e0:86)
Date de validité 
Sur quelle machine est la clé privée associée à la clé publique présente dans le certificat

QUESTION 4 
Lorsque Bob se connecte en https au site www.alice.com, indiquer, en les détaillant un
minimum, toutes les étapes pour l’authentification du site de Alice. On considère que le certificat d’Alice a été délivré par ca1 et que le certificat de ca1 a été délivré par root-ca.

Serveur envoie son certificat signé par ca1.
Le navigateur de Bob vérifie la chaîne de confiance : Alice > ca1 > root-ca.
Vérification de la validité du certificat et du domaine.
Si tout est correct, le serveur est authentifié.

Question 5
Quelle est la longueur du « n » généré ? Rappeler les calculs mis en œuvre pour chiffrer un message M puis pour déchiffrer ? Le « publicExponant » est-il difficile à deviner pour un pirate ? Estce problématique ?

rsa_keygen_bits:512 signifie que la clé privée aura une taille totale de 512 bits, donc la longueur de n est de 512 bits.
Le publicExponant n’est pas compliqué à trouver mais ce n’est pas un problème de sécurité, car la sécurité de RSA repose sur la difficulté de factoriser n.

Question 6
Y-a-t-il un intérêt à chiffrer une clé publique ? À chiffrer une clé privée ?

Pas d'intérêt à chiffrer Kpub car elle est vouée à être publique.
Chiffrer la Kpriv est un peu plus sensé (en cas de vol ?) mais doit rester privée dans tous les cas.

Question 7
Rechercher sur Internet quel est l’encodage utilisé pour la clé ? Quel est l’avantage de cet encodage ?

La clé est un format texte encodé en Base64. Elle peut donc être lue en ascii

Question 8
Retrouve-t-on bien les éléments attendus ? Pourquoi est-il intéressant de disposer d'un fichier à part contenant uniquement la clé publique ?

On retrouve bien le Modulus et le Exponent attendu dans une clé publique, comme rappelé dans la question 1.
L'intérêt de séparer en deux fichiers est un intérêt pratique pour partager plus facilenent la clé publique 



Question 9 
Quelle clé doit-on utiliser pour chiffrer un message RSA lorsqu'on veut l’envoyer de
manière confidentielle ?

Quand on veut envoyer un message de manière confidentielle avec RSA, on chiffre le message avec la clé publique du destinataire.


4.1
Dans man openssl-genpkey :
openssl genpkey -algorithm RSA -out rsa_keys.pem -pkeyopt rsa_keygen_bits:1024
openssl rsa -in rsa_keys.pem -text -noout

4.2.1
Commandes interessantes :
openssl pkeyutl -encrypt \
  -in <fichier_clair> \
  -out <fichier_chiffré> \
  -pubin -inkey <clé_publique> \
  -pkeyopt rsa_padding_mode:oaep


Question 10

openssl pkeyutl -encrypt -in clair.txt -out cipher.bin -pubin -inkey cle_publique.pem -pkeyopt rsa_padding_mode:oaep

Question 11
Chiffrer plusieurs fois le même fichier clair.txt, et enregistrer le résultat dans les fichiers cipher.bin.1, cipher.bin.2. Comparer les contenus des différents messages chiffrés. Sont-ils identiques ? Est-ce normal ? Justifier.

Les messages chiffrés ne sont pas identiques, dû au padding aléatoire.

DECRYPYTER
openssl pkeyutl -decrypt \
  -inkey cle_privee.pem \
  -in cipher.bin \
  -out message_dechiffre.txt \
  -pkeyopt rsa_padding_mode:oaep


Pour décrypter un message chiffré avec notre clé publique:
openssl pkeyutl -decrypt -inkey cle_privee.pem -in cipher.bin -out message_dechiffre.txt -pkeyopt rsa_padding_mode:oaep

openssl pkeyutl -decrypt -inkey memae.pem -in ciphergui.bin -out message_dechiffre.txt -pkeyopt rsa_padding_mode:oaep

Question 12
Lire dans la page de manuel de s_client le rôle de l’option -showcerts. Expliquer le
rôle de cette option. Après avoir parcouru le résultat de la commande s_client, indiquer le nombre de certificats qui ont été envoyés par le serveur.

D’après la page de manuel (man openssl-s_client) :
-showcerts
 affiche tous les certificats envoyés par le serveur pendant l’établissement de la connexion TLS (le certificat du serveur + les certificats intermédiaires).
Autrement dit :
Sans -showcerts, openssl s_client affiche uniquement le certificat final du serveur.


Avec -showcerts, tu vois toute la chaîne de certification :


Le certificat du serveur (leaf certificate)


Un ou plusieurs certificats intermédiaires


(Parfois) le certificat racine si le serveur l’envoie (ce n’est pas obligatoire).
