# Préparation négociation — mission Fiabila (extension du modèle à tous les produits)

Profil retenu : **étudiant**, contrat visé **freelance / auto-entrepreneur**, périmètre encore flou, tu disposes du **nombre de lots/an** et du **taux de défaut**.

Ce document est volontairement direct. Là où ton profil te limite, c'est écrit noir sur blanc.

---

## 1. Le constat de départ, sans flatterie

Tu n'es pas en position de junior diplômé. Tu es **étudiant**, sans diplôme en poche. Le marché paie un data scientist junior salarié 42–55 K€/an et un freelance data junior 250–400 €/jour — mais ces chiffres supposent un diplôme et souvent une première expérience. Pris au pied de la lettre, ton « tarif marché » est dans le **bas** de cette fourchette, voire en dessous.

Ce que le tarif marché ignore, et qui est ton vrai levier :

1. **Tu as déjà livré.** Le modèle B430 existe, tourne, et donne 85,7 % de précision. Tu ne vends pas une promesse, tu vends une preuve.
2. **Tu connais leur process.** Un cabinet ou un nouvel embauché passerait des semaines à comprendre ce que tu sais déjà.
3. **Tes alternatives leur coûtent beaucoup plus cher.** Embaucher un data scientist : 45–55 K€ + ~45 % de charges patronales ≈ **65–80 K€/an chargé**. Un cabinet de conseil data : **800–1 200 €/jour**. Toi, tu fais une partie du travail pour une fraction de ça.

**La bonne façon de te vendre n'est pas « tarif étudiant ». C'est « le même résultat qu'un cabinet, pour 3 à 4 fois moins cher ».** Tu ancres sur ce que tu remplaces, pas sur ton CV.

---

## 2. L'avertissement technique que tu dois intégrer AVANT de chiffrer

Ton modèle repose sur **28 lots** (20 BON, 8 MAUVAIS). C'est minuscule. Conséquences que tu ne dois pas te cacher :

- Rien ne garantit que les autres produits se séparent aussi proprement que le B430. La température explique ~51 % de la décision sur le B430 ; sur un autre produit, ce sera peut-être autre chose, ou rien d'exploitable.
- Certains produits n'auront **pas assez de données étiquetées** (lots BON/MAUVAIS historisés) pour entraîner un modèle. Sans données, pas de modèle.
- 85,7 % de précision, ça veut dire **~1 lot sur 7 mal classé**. Ce n'est pas un détecteur parfait.

**Ce que ça implique pour la négo :**
- Ne promets **jamais** « le même résultat partout ». Tu vends une **méthode** + un **pilote payant par produit**, pas une garantie de performance.
- N'accepte **jamais** d'être tenu responsable d'une décision de production prise sur la base du modèle (voir §6, clause de responsabilité). C'est le point le plus important du contrat.

---

## 3. Salarié vs freelance : la logique des chiffres

Tu pars sur **auto-entrepreneur**, donc voici ce qui compte concrètement.

| | Salarié | Toi (auto-entrepreneur, micro-BNC) |
|---|---|---|
| Ce que tu touches | Salaire net (~78 % du brut) | CA encaissé **− 25,6 % de cotisations URSSAF** (taux 2026, prestations libérales BNC) |
| Impôt | Prélevé à la source | En plus : soit barème classique, soit versement libératoire (+2,2 % → 27,8 % total) si tu y es éligible |
| TVA | Sans objet | **Pas de TVA à facturer** tant que tu restes sous **37 500 €** de CA/an (seuil 2026 inchangé). Tu factures HT, tu es donc moins cher pour le client. |
| Congés payés | Oui | **Non.** Pas de jour facturé = pas de revenu. |
| Chômage / sécurité | Oui | **Non.** Pas de filet. |
| Première année | — | **ACRE** possible : cotisations réduites (~moitié) la 1ʳᵉ année. À vérifier selon ta situation. |

**Le point clé à défendre :** un TJM freelance n'est PAS comparable à un salaire ramené à la journée. Sur 350 €/jour, tu gardes ~260 € après URSSAF, sans congés ni protection. Le client te paie plus cher à la journée qu'un salarié justement parce que tu portes ces risques et que tu n'es engagé que le temps de la mission. C'est normal, assume-le.

---

## 4. Ta fourchette de TJM

| Niveau | TJM | Justification | Net estimé après URSSAF |
|---|---|---|---|
| **Plancher** (en dessous, tu refuses) | **250 €/j** | Bas de la fourchette marché data junior. En dessous, vu que le modèle est déjà prouvé, ça ne vaut pas ton temps. | ~186 €/j |
| **Cible** | **350 €/j** | Tu livres en autonomie un travail de ML industriel de niche, avec un résultat déjà démontré. | ~260 €/j |
| **Plafond ambitieux** | **450–500 €/j** | À viser si le périmètre est large (tout le catalogue) et que tu cadres la discussion sur la valeur / le coût de remplacement. | ~335–372 €/j |

**Stratégie d'ancrage :** annonce **450 €/j** en premier, justifié par le coût d'un cabinet (800–1 200) et d'une embauche (65–80 K€ chargés). Laisse-les négocier vers 350. Tu auras l'air d'avoir fait un effort, et tu seras à ta cible.

**Alternative à proposer si le périmètre est gros : le forfait par produit.** En facturant au jour, tu es pénalisé d'être rapide (tu réutilises ton code). En facturant un forfait par produit (ex. 1 500–2 500 € le produit selon complexité), tu captes la valeur de ta vitesse. **Mais attention :** comme étudiant tu vas sous-estimer le temps. Donc l'ordre recommandé :
1. **Phase de cadrage payée au TJM** sur 1–2 produits → tu mesures ta vraie vitesse et tu vérifies que les données existent.
2. **Forfait par produit** ensuite, une fois que tu sais ce que ça te coûte réellement.

---

## 5. Les questions à poser au CEO AVANT d'annoncer un prix

N'annonce aucun chiffre tant que tu n'as pas les réponses. Chacune change le devis :

1. **Combien de produits exactement, et lesquels en priorité ?** « Tous » n'est pas un périmètre, c'est un piège (voir §6).
2. **Pour chaque produit, existe-t-il un historique de mesures avec l'étiquette qualité (BON/MAUVAIS) ?** Sans données, pas de modèle — et c'est leur problème, pas le tien.
3. **Quelle est la deadline / durée souhaitée ?**
4. **Qui maintient et ré-entraîne les modèles après la livraison ?** Si c'est toi → c'est une **récurrence facturable** (contrat de maintenance), pas du gratuit.
5. **Le livrable attendu, c'est un modèle, ou un outil utilisable par les opérateurs ?** Les deux n'ont rien à voir en charge de travail. Un outil = beaucoup plus cher.
6. **Aviez-vous déjà un budget en tête ?** Toujours utile de les faire parler en premier sur le budget si tu peux.

---

## 6. Red flags et points à border dans le contrat

- **Responsabilité (LE point critique).** Le modèle se trompe ~1 fois sur 7. Clause obligatoire : tu fournis un **outil d'aide à la décision**, la décision finale de production reste au client. **Limitation de responsabilité** explicite. Ne signe jamais sans ça.
- **Périmètre évolutif.** « Tous les produits » doit être **découpé par phases**, chacune avec son devis. Toute extension = nouveau bon de commande. Sinon tu travailles gratuitement sur un périmètre qui gonfle.
- **Propriété du code / des modèles.** Décide ce que tu cèdes. Tu peux livrer les modèles entraînés tout en **gardant la propriété de ton outillage réutilisable** (tes scripts génériques). À négocier, ne cède pas tout par défaut.
- **Conditions de paiement.** Tu n'as pas de trésorerie : exige un **acompte (30 %) à la commande**, paiement à **30 jours max**, pénalités de retard. Pour un forfait, paiement échelonné par jalons.
- **Exclusivité.** Ne l'accorde pas gratuitement. Si le CEO veut t'empêcher de bosser ailleurs, ça se paie.
- **Requalification en salariat.** Si tu travailles quasi exclusivement pour eux, avec leurs outils, sous leurs directives, l'URSSAF peut requalifier la prestation en contrat de travail. Pour toi ce n'est pas forcément négatif (ça t'ouvrirait des droits), mais l'entreprise en a peur — c'est un argument que tu peux utiliser s'ils te traitent comme un salarié déguisé tout en te payant comme un freelance.

---

## 7. Réponses aux objections classiques

- **« Tu es étudiant, tu n'as pas de diplôme. »**
  → « C'est exact, et c'est pour ça que je suis à 350 et pas à 900 comme un cabinet. Ce que vous payez, ce n'est pas mon diplôme, c'est le B430 qui tourne déjà. »

- **« On n'a pas le budget. »**
  → Ne baisse pas ton tarif, **réduis le périmètre.** « On commence par les 2 produits les plus coûteux en rebuts, ça s'autofinance avec les économies, et on étend ensuite. »

- **« 350 € par jour, c'est cher pour un étudiant. »**
  → Ramène au coût réel : « Un cabinet vous facturerait 4 fois ça, un embauché vous coûterait 70 K€ chargés à l'année. Moi vous me payez à la mission. »

- **« Tu nous garantis le même résultat partout ? »**
  → « Non, et méfiez-vous de quiconque vous le garantit. Je vous garantis une méthode et un pilote par produit. Si les données ne permettent pas un bon modèle sur un produit, je vous le dis tout de suite plutôt que de vous vendre du vent. » *(Cette honnêteté est un argument de vente, pas une faiblesse.)*

---

## 8. Le levier décisif : le ROI (deux sources de valeur)

Ta meilleure carte, c'est de transformer la discussion « combien tu coûtes » en « combien tu rapportes ». Si tu montres que le modèle rapporte X €/an et que ton coût est une fraction de X, le prix cesse d'être un débat.

Il y a **deux flux de valeur distincts** — ne les confonds pas :

**Flux A — Qualité (rebuts évités).** Détecter les lots MAUVAIS avant de les produire/expédier.
Économie ≈ `N × d × c × R × f` − coût des fausses alertes.

**Flux B — Capacité (temps de broyage réduit) — probablement le plus gros.** Le modèle montre qu'on obtient du BON avec une Durée plus courte → moins de temps machine et ouvrier par lot.
Valeur = (temps gagné par lot) × N, convertie en € **uniquement si** :
1. le broyeur est un **goulot** et il y a de la **demande** → les heures libérées produisent plus (marge × lots en plus) ; ou
2. on réduit réellement les coûts (heures ouvrier, énergie, usure).

⚠️ Si la machine a déjà du temps mort et que la demande est plate, le gain capacité se limite à l'énergie/usure — réel mais petit. **Ne présente jamais un gain de capacité comme du CA garanti sans avoir validé que la demande suit.** C'est l'objection n°1 que te sortira le CEO.

**Bénéfices qualitatifs (à citer, pas à chiffrer) :**
- Accélération de la R&D / mise au point des futurs produits (remplace des mois d'essais-erreurs).
- Capitalisation de la connaissance process dans un modèle réutilisable.
→ Réels, mais **ne leur colle pas de chiffre** : un € inventé décrédibilise tout le tableau.

**Le piège du ROI :** si tu surpromets et que tu ne livres pas, tu perds le contrat ET ta réputation. Le scénario « prudent » doit être **vraiment** prudent.

**Chiffres encore manquants pour le calcul :** N (lots/an), d (taux de défaut), c (coût d'un lot raté), et surtout **la réduction de temps de broyage** (ex. 90 → 60 min) qui pilote le flux B.

---

## 9. La vidéo drone : à ne PAS offrir gratuitement maintenant

Offrir du travail gratuit en pleine négociation de prix **sabote ton ancrage** — ça signale que tu ne vaux pas ton tarif. Donc :
- Ne la mets pas sur la table comme argument de vente.
- Geste de bonne volonté **après signature** seulement (« cadeau de lancement »).
- Ou prestation séparée, payante ou pour ton portfolio. Une vidéo drone a sa propre valeur, ne la dilue pas dans la mission data.
