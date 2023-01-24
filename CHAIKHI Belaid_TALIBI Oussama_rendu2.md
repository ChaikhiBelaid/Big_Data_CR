Ce travail est réalisé par : <br/>
- Belaid CHAIKHI
- Oussama TALIBI
<br/>

# Partie 1 : Anagramme
## 1.1 Algorithme 
Dans cette partie, nous avons dévelopé une algorithmes avec map-reduce sur MrJob afin de pouvoir détecter et lister les anagrammes (mots des lettres similaires) à partir d'un fichier du vocabulaire. L'idée de l'algorithme est simple, elle consiste à extraire chaque mot et le transformer en liste des lettres classées selon l'ordre alphabétique, puis grouper les mots qui ont une liste similaire. Le script de l'algorithme se divise en deux étapes :

- Un mapper qui permet de transformer le mot en liste des lettres classées selon l'ordre alphabétique (cela s'effectue à l'aide de la fonction prédifinie en python **sorted**) et il produit cette liste comme clé et le mot comme valeur. 

- Un reducer permet de regrouper tous les mots qui ont la même clé générée par le mapper et il retourne dans chaque ligne la liste des anagrammes qui contient au moins deux mots. 

Vous trouverez ci-dessus le script complet de l'algorithme : 

```python
# Importer les librairies
from mrjob.job import MRJob
from mrjob.protocol import JSONValueProtocol
from mrjob.step import MRStep
import re

WORD_RE=re.compile(r"[\w']+")
class MRAnagramme(MRJob):
    OUTPUT_PROTOCOL=JSONValueProtocol
    
    # Définir le mapper
    def mapper_words(self, _, line):
        """Ce mapper permet de lire le fichier texte 
        et d'extraire les infos concernant chaque mot.

        Args:
            line : la ligne du fichier texte

        Yields:
            sorted_word : list, Liste des lettres du mot en ordre alphabetique
            word : str, le mot extrait à partir de line
        """
        # Extraire le mot
        word=line.lower()
        
        # Vérifier si le mot extrait n'est pas une lettre
        if len(word)>=2:
            # Classer les lettres du mot selon l'ordre alphabetique
            sorted_word=sorted(word)
            
            # yield chaque mot avec sa liste des alphabets comme clé
            yield sorted_word,word
    
    # Définir le reducer        
    def reducer_similair_words(self, sorted_word, words):
        """Ce reducer permet de donner les anagrammes c'est-à-dire 
        les mots qui ont des lettre similaires.

        Args:
            sorted_word : list, Liste des lettres du mot en ordre alphabetique (clé)
            words : L'ensemble des mots avec des lettres similaires.

        Yields:
            similair_words : list, Liste des mots avec des lettre similaires (anagrammes)
        """
        # Définir la liste des mots avec des lettres similaires
        similair_words=[]
        for word in words:
            similair_words.append(word)
            
        # Vérifier si l'nsemble des anagrammes contient au moins deux mots
        if len(similair_words)>=2:
            yield ("",(similair_words))
            
    # Définir les steps
    def steps(self):
        return [
        MRStep(mapper=self.mapper_words,
        reducer=self.reducer_similair_words)
        ]
        
if __name__=='__main__' :
    MRAnagramme.run()

```

# 1.2 Résultats :
### 1.2.1 Test sur des mots anglais
Pour tester cet algorithme, nous avons utilisé la base de données des mots anglais **words_alpha.txt** extraite à partir du lien **https://raw.githubusercontent.com/dwyl/english-words/master/words_alpha.txt**. En exécutant l'algortihme sur cette base, nous avons obtenu les résultats suivants (extrait):

```
["basiparachromatin", "marsipobranchiata"]
["alcazaba", "calabaza"]
["carcinosarcomata", "sarcocarcinomata"]
["anatomicopathological", "pathologicoanatomical"]
["adenosarcomata", "sarcoadenomata"]
["paradisaically", "paradisiacally"]
["paradisaical", "paradisiacal"]
["aracanga", "caragana"]
["ayahausca", "ayahuasca"]
["atacaman", "tamanaca"]
["anacara", "aracana"]
["salamandarin", "salamandrian", "salamandrina"]
["nasopalatal", "palatonasal"]
["tantarara", "tarantara"]
["bombacaceae", "cabombaceae"]
["barabra", "barbara"]
["brachiofacial", "faciobrachial"]
["abranchiate", "antebrachia"]
["clarabella", "racallable"]
["ablactate", "cabaletta"]
["batrachian", "branchiata"]
["batrachia", "brachiata"]
["balachan", "chanabal"]
["arabica", "arbacia"]
["cabala", "calaba"]
```

### 1.2.2 Test sur des mots en français 
Pour vérifier encore l'efficacité de l'algorithme, nous avons cherché une base de donnéees des mots en français avec laquelle on peut tester notre algortihme. Ainsi, nous avons trouvé une base réalisé par Monsieur **Christophe Pallier** et nous l'avons extraite à partir du lien suivant: **https://www.pallier.org/extra/liste.de.mots.francais.frgut.txt**. exécutant l'algortihme sur cette base, nous avons obtenu les résultats suivants (extrait):

```
["haut-de-chausses", "hauts-de-chausse"]
["talkies-walkies", "walkies-talkies"]
["talkie-walkie", "walkie-talkie"]
["bracelets-montres", "montres-bracelets"]
["bracelet-montre", "montre-bracelet"]
["arc-boutent", "contre-buta"]
["duffel-coats", "duffle-coats"]
["duffel-coat", "duffle-coat"]
["cartons-p\u00e2tes", "contre-pass\u00e2t"]
["sous-entendrait", "sous-tendraient"]
["sous-entendait", "sous-tendaient"]
["grand-m\u00e8re", "m\u00e8re-grand"]
["grand-m\u00e8res", "m\u00e8res-grand"]
["quatre-vingt", "vingt-quatre"]
["sous-titrera", "sous-traiter"]
["fait-tout", "tout-fait"]
["cabaneraient", "encabanerait"]
["cabalerais", "calabraise"]
["cabanaient", "encabanait"]
["cabaneras", "sarbacane"]
["bataclan", "cabalant"]
["barattages", "rabattages"]
["barattage", "rabattage"]
["arabiserais", "rabaisserai"]
["ballasterai", "batailleras"]
["alabastrite", "attablerais"]
```

# Partie 2 : Requête  sur le fichier des ventes 
## 2.1 Algorithme 
Dans cette partie, nous avons proposé une requête qui consiste à calculer les prix moyennes par heure de la journée et par ville pour catégorie d'achat. Ceci nous donne les zones géographiques ainsi que les périodes de la journée dans les quelles on a une maximume rentabilité pour chaque catégorie. Pour réailiser cela le script se découpe en trois étapes : 

- Un mapper pour extraire les informations sur l'heure, la ville, la catégorie d'achat et le prix à partir de chaque ligne du fichier de ventes. En sortie, il produit des paires clé-valeur de la forme ((ville, catégorie d'achat, heure), prix).

- Un premier reduceur pour calculer la moyenne des dépenses pour chaque clé ((ville, catégorie d'achat, heure)) produite par le mapper. En sortie, il produit des paires clé-valeur de la forme (catégorie d'achat,(heure, ville, moyenne des prix).

- Un autre reduceur pour diviser donner les heures de la journée ainsi que les ville dans lesquelles on a le maximume de rentabilité où on a le maximume des prix (les trois premier valeurs). En sortie il produit (catégorie d'achat,(heure, ville, moyenne des prix). 

Vous trouverez ci-dessus le script complet : 

```python
# Importer les librairies
from mrjob.job import MRJob
from mrjob.protocol import JSONValueProtocol
from mrjob.step import MRStep
import re

# Définir le MRJob
WORD_RE=re.compile(r"[\w']+")
class MRMaxProfitability(MRJob):
    OUTPUT_PROTOCOL=JSONValueProtocol
    # Définir le mapper
    def mapper_extract_infos(self, _, line):
        """ Ce mapper permet d'extraire des infos concernant 
        la catégorie, l'heure, la ville et le prix de chaque achat à partir 
        du fichier de ventes.

        Args:
            line : ligne du fichie de ventes.

        Yields:
            (category, hour, city) : tuple qui contien la catégorie, l'heure et la ville de chaque achat (clé).
            price : float, le prix de chaque achat.
        """
        # Extraire la data de la ligne sous forme de mots (words) (date, heure, catégorie....)
        words = line.split("\t")
        
        # Vérifier s'il n'ya pas des données manquantes
        if len(words)==6 :
            # Extraire le prix, la ville, la catégorie et l'heure de l'achat
            price=words[4]
            city=words[2]
            category=words[3]
            hour=words[1].split(":")[0]+"h"
            
            # yield les résultats sous forme clé et valeur
            yield (category, hour, city),float(price)
    
    # Définir le premier reducer 
    def reducer_mean_prices(self, key, prices):
        """ Ce reducer permet de calculer la moyenne des prix  de meme catégorie, 
        de meme heure et de meme ville et de les grouper selon la catégorie d'achat.

        Args:
            key : la clé qui contient la catégorie, l'heure et la ville de chaque achat.
            prices: les valeurs (les prix) correspendantes à chaque clé.

        Yields:
            category : la catégorie (clé).
            hour, city, mean(prices) : heure, ville et moyenne des prix pour chaque catégorie d'achat.
        """
        # Liste des prix pour chaque clé 
        prices=list(prices)
        
        # Diviser la clé en catégorie, heure et ville
        category, hour, city=key
        
        # Regrouper les résulats selon la catégorie (clé)
        yield (category, (hour,city, sum(prices)/len(prices)))
    
    # Définir le deuxième reducer
    def reducer_max_profitability(self, category, values):
        """Ce reducer permet de retourner les heures et les villes pour les quelles
        on a le maximume de dépenses (les trois premiers) pour chaque catégorie.

        Args:
            category : la catégorie de l'achat (clé).
            values : les valeurs de chaque catégorie diviée (heure, ville et moyenne des prix).
        """
        # Liste des valeurs
        list_values=list(values)
        # Liste des prix 
        prices=[price for (_, __, price) in list_values]
        # Liste des prix en ordre décroissant
        sorted_prices=sorted(prices, reverse=True)
        # Liste des valeurs classées selon l'ordre de la liste des prix 
        sorted_list_values=[list_values[prices.index(price)] for price in sorted_prices]
        
        # retenir les trois premier valeurs
        sorted_list_values=sorted_list_values[:3]
        for hour,city,price in sorted_list_values:
            yield("",((category, hour, city, price)))
    
    # Définir les steps 
    def steps(self):
        return [
        MRStep(mapper=self.mapper_extract_infos,
        reducer=self.reducer_mean_prices),
        MRStep(reducer=self.reducer_max_profitability)
        ]
        
if __name__=='__main__' :
    MRMaxProfitability.run()
```

## 2.2 Résultats 
En exécutant le script de l'algorithme sur le fichier de ventes **purchases.txt**, nous avons obtenus les résultats suivants:

```
["Baby", "12h", "Rochester", 275.3939830508475]
["Baby", "13h", "Plano", 274.9555416666666]
["Baby", "10h", "San Francisco", 274.77307999999994]
["Books", "17h", "Madison", 280.0346768060837]
["Books", "14h", "Boston", 276.9280912863071]
["Books", "17h", "Honolulu", 276.4006274509804]
["CDs", "09h", "Saint Paul", 275.316377952756]
["CDs", "11h", "Chesapeake", 272.5301694915252]
["CDs", "13h", "Colorado Springs", 272.40014705882356]
["Cameras", "17h", "Corpus Christi", 284.87313432835833]
["Cameras", "15h", "Garland", 277.5809704641349]
["Cameras", "17h", "San Francisco", 274.6658008658008]
["Children's Clothing", "14h", "Sacramento", 280.5273122529644]
["Children's Clothing", "12h", "Plano", 274.2168093385214]
["Children's Clothing", "17h", "Anchorage", 274.15581896551714]
["Computers", "10h", "Seattle", 280.8981893004116]
["Computers", "11h", "Washington", 277.31582644628105]
["Computers", "10h", "Irvine", 275.64123456790134]
["Consumer Electronics", "17h", "El Paso", 277.38377118644075]
["Consumer Electronics", "17h", "Omaha", 276.1001619433199]
["Consumer Electronics", "12h", "Baton Rouge", 272.809646017699]
["Crafts", "11h", "San Bernardino", 278.70227091633467]
["Crafts", "14h", "Jacksonville", 277.44697211155363]
["Crafts", "16h", "Raleigh", 276.9441871921183]
["DVDs", "17h", "Portland", 280.3675111111111]
["DVDs", "13h", "Birmingham", 280.2319521912351]
["DVDs", "15h", "Richmond", 275.26149572649575]
["Garden", "14h", "Greensboro", 280.82034042553204]
["Garden", "13h", "Louisville", 278.9044491525424]
["Garden", "09h", "Scottsdale", 275.5701632653062]
["Health and Beauty", "16h", "Kansas City", 286.5474672489083]
["Health and Beauty", "11h", "Anaheim", 286.42121212121214]
["Health and Beauty", "10h", "Buffalo", 278.06489711934154]
["Men's Clothing", "10h", "Richmond", 283.1957768924302]
["Men's Clothing", "16h", "Miami", 281.5026808510638]
["Men's Clothing", "13h", "Chicago", 274.6020920502093]
["Music", "11h", "Lubbock", 275.7694090909092]
["Music", "12h", "Birmingham", 275.6123505976093]
["Music", "09h", "Newark", 272.9117857142857]
["Pet Supplies", "10h", "Baltimore", 281.09540178571433]
["Pet Supplies", "10h", "Pittsburgh", 281.0693013100438]
["Pet Supplies", "15h", "Anaheim", 276.17063745019914]
["Sporting Goods", "10h", "Madison", 276.2453703703703]
["Sporting Goods", "11h", "Virginia Beach", 276.10096774193556]
["Sporting Goods", "12h", "Madison", 275.4428947368422]
["Toys", "13h", "Kansas City", 277.67724409448834]
["Toys", "10h", "New Orleans", 274.843539823009]
["Toys", "13h", "Hialeah", 273.6598046875]
["Video Games", "13h", "St. Petersburg", 279.1180000000001]
["Video Games", "09h", "Winston\u2013Salem", 276.4099024390244]
["Video Games", "17h", "Irvine", 276.06595652173905]
["Women's Clothing", "09h", "Sacramento", 275.97579365079366]
["Women's Clothing", "15h", "Wichita", 272.82512396694233]
["Women's Clothing", "13h", "Lubbock", 272.7356818181817]
```