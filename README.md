# TP1 – INFO001

**Auteurs :**

* **Mema Entela**
* **Rey Guillaume**

---

## **Question 1 – Rappel RSA**

Soit un message **M** dont on souhaite assurer la confidentialité.
Le chiffrement RSA utilise :

* **Clé publique** : (n, e)
* **Clé privée** : (n, d)

**Chiffrement :**

```text
C = M^e mod n
```

**Déchiffrement :**

```text
M = C^d mod n
```

---

## **Question 2 – Rôle de Diffie-Hellman**

Diffie-Hellman permet à deux parties d’établir **une clé secrète commune** via un canal non sécurisé.
Même si un attaquant intercepte tous les messages échangés (p, g, P1, P2), il ne peut pas retrouver la clé finale car les secrets privés x et y sont impossibles à recalculer.

Utilisation majeure : **établissement de clés dans SSL/TLS**.

---

## **Question 3 – Contenu important d’un certificat**

Un certificat contient notamment :

* **SAN (Subject Alternative Name)** : domaines protégés.
* **Signature du certificat**.
* **Dates de validité**.
* **Machine où se trouve la clé privée associée**.

---

## **Question 4 – Authentification de [www.alice.com](http://www.alice.com) en HTTPS**

1. Le serveur envoie son certificat signé par **ca1**.
2. Le navigateur vérifie la chaîne de confiance :

   ```
   Alice → ca1 → root-ca
   ```
3. Vérification du domaine + validité + signature.
4. Si tout est correct, Alice est authentifiée.

---

## **Question 5 – Longueur de n et sécurité du publicExponent**

`rsa_keygen_bits:512` → **n fait 512 bits**.

* Chiffrement : `C = M^e mod n`
* Déchiffrement : `M = C^d mod n`

Le **publicExponent (e)** est trivial à deviner (souvent 65537), mais ce n’est **pas un problème** :
la sécurité RSA repose sur la **factorisation de n**, pas sur la discrétion de e.

---

## **Question 6 – Chiffrer une clé publique ou privée ?**

* **Clé publique :** aucun intérêt → elle est destinée à être diffusée.
* **Clé privée :** peut être chiffrée avec un mot de passe pour limiter les dégâts en cas de vol.

---

## **Question 7 – Encodage des clés**

Les clés sont encodées en **Base64** (PEM).
Avantage :

* transportable en ASCII,
* lisible,
* compatible avec mails, fichiers texte, etc.

---

## **Question 8 – Clé publique dans un fichier séparé**

Oui, on retrouve bien **Modulus** et **Exponent**.

Avantage d’un fichier séparé :
facilite la distribution de la clé publique sans exposer la clé privée.

---

## **Question 9 – Quelle clé utiliser pour chiffrer un message RSA ?**

Pour assurer la confidentialité :
→ **on chiffre avec la clé publique du destinataire**.

---

## **4.1 – Génération de clés**

```bash
openssl genpkey -algorithm RSA -out rsa_keys.pem -pkeyopt rsa_keygen_bits:1024
openssl rsa -in rsa_keys.pem -text -noout
```

---

## **4.2 – Exemple de chiffrement OAEP**

```bash
openssl pkeyutl -encrypt \
  -in fichier_clair \
  -out fichier_chiffré \
  -pubin -inkey clé_publique.pem \
  -pkeyopt rsa_padding_mode:oaep
```

---

## **Question 10 – Chiffrement**

```bash
openssl pkeyutl -encrypt \
  -in clair.txt \
  -out cipher.bin \
  -pubin -inkey cle_publique.pem \
  -pkeyopt rsa_padding_mode:oaep
```

---

## **Question 11 – Chiffrement multiple**

En chiffrant plusieurs fois le même fichier, les résultats diffèrent.
→ **Normal** : RSA OAEP utilise un **padding aléatoire**.

Déchiffrement :

```bash
openssl pkeyutl -decrypt \
  -inkey cle_privee.pem \
  -in cipher.bin \
  -out message_dechiffre.txt \
  -pkeyopt rsa_padding_mode:oaep
```

---

## **Question 12 – Rôle de -showcerts**

Avec :

```bash
openssl s_client -connect site.com:443 -showcerts
```

Option `-showcerts` :
→ affiche **tous les certificats envoyés par le serveur**, c’est-à-dire :

* certificat du serveur (leaf)
* certificats intermédiaires
* parfois le certificat racine

---

## **Question 13 – Lecture des champs Subject / Issuer**

Exemple :

```
Subject: C=FR, ST=Auvergne-Rhône-Alpes, ...
Issuer: C=NL, O=GEANT Vereniging, ...
```

Champs principaux :

* **C** : pays
* **ST** : région
* **L** : localité
* **O** : organisation
* **OU** : unité
* **CN** : nom commun

---

## **Question 14**

`Subject` (s) = identité du propriétaire
`Issuer` (i) = autorité ayant signé le certificat

---

## **Question 15 – Validité et SAN**

* Clé publique RSA
* Signé avec RSA-SHA256
* CN = domaine principal
* SAN = autres domaines
* Validité = ~1 an

Lien CRL : vérification de la révocation.

---

## **Question 16 – Signature d’un certificat**

Formule :

```text
S = h^d mod n
```

Avec :

* **h** = hash des données du certificat
* **d** = clé privée de l’émetteur

---

## **Question 18 – Validation d’un certificat**

Le certificat n₂ est validé par le certificat n₁ (autorité intermédiaire).
Il se trouve dans :

* la chaîne envoyée par le serveur, ou
* le magasin local de certificats.

---

## **Question 19 – Auto-signature**

Si **Subject = Issuer**, alors le certificat est **auto-signé** → certificat racine.

Signature : même formule RSA : `S = h^d mod n`.

---

## **Question 20 – Informations sur la clé de la CA**

* Type de clé : **RSA**
* Taille : **2048 bits**
* Courbe : **aucune** (RSA ≠ EC)
* Validité : en général **10 ans**
* Auto-signé : Subject = Issuer
* X509v3 Key Usage : **Cert Sign**, **CRL Sign**

---

## **Question 21 – Dossiers et fichiers**

```
dir = /home/etudiant/ca
Clé privée  → private/ca.key.pem
Certificat → certs/ca.cert.pem
```

---

## **Question 22 – Commande pour créer la clé**

```bash
openssl genrsa -aes128 -out private/intermediate.key.pem 3072
```

CSR :

```bash
openssl req -config openssl.cnf -new -sha256 \
  -key private/intermediate.key.pem \
  -out csr/intermediate.csr.pem \
  -subj "/C=FR/ST=Savoie/L=Chambery/O=TP Sécurité/CN=RA votre_login"
```

---

## **Question 23 – Pourquoi une signature dans un CSR ?**

Le CSR contient une signature faite avec **la clé privée du demandeur**.

But :
→ prouver à l’autorité de certification que **le demandeur possède bien la clé privée associée**.

---

## **Question 24 – Certificats envoyés**

Le serveur peut envoyer :

* 1 certificat serveur
* 1 ou plusieurs certificats intermédiaires
* parfois un certificat racine (peu fréquent)

