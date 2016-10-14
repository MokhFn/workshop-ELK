# Atelier jouons avec nos données avec ELK
## Dev&Tests days Orange Meylan

### Introduction
Orange a décidé de diversifier sa boutique en vendant du matériel informatique. Pc, imprimantes, clavier, etc. sont désormais en vente sur la boutique Orange. Cette nouvelle offre a fait un carton en janvier, cependant après la dernière mise à jour du portail de nombreux problèmes sont remontés par l'exploitation et le marketing se plaint d'une forte baisse des ventes. 

Le marketing a demandé qu'un super développeur vienne voir si quelque chose va mal et comme tu étais en vacances en Picardie pendant la réunion du département, tu as été désigné volontaire :-).

Ne sachant pas par où commencer, tu te connectes en ssh au serveur Apache de la boutique et tu télécharges tous les fichiers de log depuis juin et tu décides de les charger dans Elasticsearch pour voir ce qui ne va pas.
Ton travail peut commencer jeune padawan !

### Découverte des outils
La première chose à faire est de charger les données dans Elasticsearch, pour ça les développeurs de chez Elastic (la société qui édite le moteur de recherche open source) a développé un outil assez pratique pour lire dans des fichiers et écrire les données dans Elasticsearch. 

Tu décides de t'entrainer sur un petit fichier de logs access_log_1.log (tous les fichiers de logs se trouvent dans le dossier data de ce répertoire).

Heureusement ton collègue Jean Bouffedélog t'a donné quelques instructions pour commencer...

-----
>De : Jean Bouffedélog

>à : moi@orange.com

>Hello,
>Oui jai déjà utilisé la stack ELK (Elasticsearch, Logstash et Kibana) c'est plutôt simple à mettre en place et je t'ai fait un petit package pour les utiliser directement sur ta machine (C:\Program Files\Elastic).
>
>Pour commencer tu devras démarrer elasticsearch et kibana. C'est simple, ouvre deux consoles et lance les commandes suivantes :
>
>Elasticsearch 
```
// TODO: elasticsearch start command
```
>Pour vérifier que la commande a bien marché, ouvre ton firefox et va sur l'adresse http://localhost:9200/

>Si tu vois quelque chose c'est que je ne dis pas n'importe quoi !

>Kibana 
```
//TODO: Kibana start command
```
>Pour vérifier que la commande a bien marché, ouvre ton firefox et va sur l'adresse http://localhost:5601/

>Tu verras c'est plus joli que l'interface d'Elasticsearch mais pour l'instant il n'y a rien dedans à toi d'y mettre des données !
>
>Il faudrait aussi que tu installes sense, ça te permettra de requêter facilement elasticsearch.

>```
>bin\kibana plugin --install elastic/sense
>```
>Pour vérifier que la commande a bien marché, ouvre ton firefox et va sur l'adresse http://localhost:5601/app/sense
>
>Une fois que ça s'est fait tu dois créer un fichier de configuration pour logstash, en général on l'appel logstash.conf et tu mets dedans quelque chose qui ressemble à ça :
>```
>input {
>    file {
>        path => "/home/txbj8305/Documents/devdays/ELK/access_log_1.log" //TODO mettre le path vers ton fichier
>        start_position => beginning
>        sincedb_path => "NUL"
>        ignore_older => 0
>    }
>}
>filter {
>    // ici on peut mettre des filtres pour parser les logs tout comme on veut
>}
>output {
>    elasticsearch {
>        hosts => [ "localhost:9200" ]
>    }
>	#stdout { codec => rubydebug }
>}
>```
>
>Si je me souviens bien, il y a un pattern grok qui permet de lire directement des logs apache (ben oui, les logs apache, c'est toujours formaté pareil). Tu devrais trouver plus d'info dans :

>* la doc elastic : https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html
>* la liste des patterns : https://github.com/elastic/logstash/blob/v1.4.2/patterns/grok-patterns
>

>Pour lancer la lecture de tes logs, démarre logstash avec la commande suivante :
>```
>//TODO logstash command logstash -f logstash.conf
>```
>
>Ensuite tu pourras aller voir tes données dans kibana, retourne dans ton navigateur http://localhost:5601.
>Il va te demander de configurer un pattern d'index.

>//TODO insérer capture acceuil Kibana

>Tu laisses le index name à logstash-* c'est le nom par défaut qui est donné à un index produit par logstash (un index est l'équivalent d'une table dans une base de données classique). Tu dois juste choisir le champ qui contient les dates de tes données et tu es prêt à commencer à visualiser les données. 
>
>Pleins de poutous et bon courage !

>Jean 

>PS : Si les données injectées ne sont pas daté du mois de Juin c'est que tu dois changer quelque chose dans le logstash.conf je te laisse demander à Google :-)  

>PS2 : Si tu veux juste voir à quoi vont ressembler les données sans forcément les injecter dans Elasticsearch tu peux décommenter la ligne stdout dans le fichier de conf et commenter les lignes elasticsearch.

>PS3 : Entre 2 injections, pour tout remettre à 0, tu peux utiliser sense pour faire un DELETE sur logstash-, mets aussi à jour l'index kibana


Tu as réussi à arriver jusque là, bravo ! Tu deviendras bientôt un pro de la stack ELK. En avant !

-----
>De : Jean Bouffedélog

>à : moi@orange.com

>J'oubliai ! Tu vas pouvoir créer des viz dans kibana, du genre :
>* Répartition des verbes HTTP appelés
>* La répartition des codes HTTP (200, 404, 500...) dans le temps
>* Liste des url appelées

### Configuration logstash
Tu montres tes premiers avancements à ton chef et il te renvoie une liste de choses à faire pour mieux exploiter les logs 

----
>De : Jean Demandebocoutro

>à : moi@orange.com

>* Permettre à logstash de lire tous les fichiers de logs 
>* Savoir de quel pays viennent nos utilisateurs
>* Connaitre quels sont les paramètres des requêtes (/search?q=ordinateur tout-en-un
>* Séparer les verbes de l'API (/search, /products...) des paramètres (?q=ordinateur tout-en-un)
>* Ajoute ce filtre Je t'expliquerai plus tard pourquoi
>```
>  	mutate {
>  		split => ["[query_params][q]", "%20"]
>  	}
>```
>Bravo pour ton travail tiens-moi au courant de ton avancement.

>Cordialement,

>Jean Demandebocoutro

### Manipulons Kibana
Une fois tout ça fini tu pensais pouvoir te détendre en lisant tes mails et tu trouves ce mail

---
>De : Jean Vendétonne
>à : moi@orange.com
>Hello,
>
>Jean Demandebocoutro m'a dit que c'est toi qui bosse sur l'amélioration de notre boutique. Merci pour ton dévouement !

>On a eu une réunion très intéressante aujourd'hui avec le reste de l'équipe marketing et Raymond Telémachine de l'exploit et nous avons identifié les KPI qui seront les plus intéressants à monitorer sur la boutique.

>Apparemment ton outils Kibana nous permettra de voir des graphes filtrable avec tout ça 
>* la carte avec la répartition des clients
>* la liste des API appelées
>* les 40 mots clés les plus répandus
>* le détail d'une recherche pour qu'on puisse faire du qualitatif

>Si tu peux nous mettre tout ça dans un dashboard ce serait sympa :-)

>Encore merci,

>Jean Vendétonne


Ni une, ni deux, tu te remets au boulot. Tout fier de toi, tu présentes tes viz à tout le monde.
Durant la présentation, tu remarques deux anomalies dans les logs, si tu arrives à les trouver, préviens un membre de la dream team ELK.
 
Vient le moment des questions...
 
> Euh... C'est bien beau, mais les mots "et", "à", "de", "achat"... dans le top 10 de la recherche, on ne peut pas les enlever ?

> Et pourquoi pour lui wifi, WIFI et wi-fi ou hdmi et HDMI ce n'est pas la même chose ?

### Analyse plein texte

Dépité tu repars pensant ajouter des filtres dans logstash pour enlever tous ces mots lors de l'injection des données tu penses pouvoir t'en sortir mais bon ça à pas l'air simple... et puis dans le bus en rentrant chez toi tu entends une discussion de deux chercheurs du moteur de recherche de Google qui discutent de "mapping" avec des "analyzers", "tokenizers" et de "token filter" tu te dis que tu es déjà tombé sur ce genre de choses dans la doc elasticsearch notamment dans le livre Elasticsearch: The definitive guide section "Dealing with human language" https://www.elastic.co/guide/en/elasticsearch/guide/current/languages.html#languages et la partie analysis de la doc https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html 

Tu te dis que la première chose à faire c'est de regarder le mapping actuel de tes données. Pour celà il suffit d'ouvrir sense http://localhost:5601/app/sense et de faire la requête suivante :
```
GET logstash-*/_mapping
```

Wow ! ça n'a pas l'air super simple tout ça, tu te dis que tu regarderas plus tard ce que signifie tout ça (après l'atelier avec les animateurs par exemple) pour l'instant ce qui t'intéresse c'est de changer le mapping de "query_params.q". Commençons par simplifier le mapping que logstash a créé. Tape ceci dans sense
```
POST logstash-2016.10.19/logs/_mapping
{
  "logs": {
    "properties" : {
      "query_params": {
        "type":"object",
        "properties": {
          "q": {
            "type": "string"
          }
        }
      }
    }
  }
}
```

Rah ! Elasticsearch n'aime pas qu'on modifie le mapping à chaud... Ce n'est pas un bug c'est "by design".

### Un peu de théorie

Pour comprendre ce que c'est le mapping il faut comprendre rapidement comment fonctionne un moteur de recherche.

Un moteur de recherche fonctionne comme un glossaire dans un livre, à chaque fois qu'un mot apparait dans un champ du moteur celui-ci stocke le mot avec une référence vers le document qui contient ce mot.

Prenons un exemple : voici trois documents que nous fournissons à un moteur de recherche
```
doc1: bonjour comment ça va 
doc2: bonjour ça va bien et toi
doc3: bien merci
```
Le moteur de recherche en construira le tableau suivant :
```
mot       | documents
________________________
bonjour   | doc1, doc2
comment   | doc1
ça        | doc1, doc2
va        | doc1, doc2
bien      | doc2, doc3
et        | doc2
toi       | doc2
merci     | doc3
```

ainsi lorsque l'on cherche le mot "bonjour" le moteur de recherche n'a pas à parcourir tous les documents, il parcour uniquement la liste de mot qu'il connait et une fois qu'il a trouvé le mot "bonjour" il retourne la liste de documents de la colonne documents. (n'hésite pas à en discuter avec les animateurs si tu le souhaites)

### Retour à la réalité 

Il faut donc à chaque fois que l'on souhaite modifier le mapping il faut dans sense :
* supprimer l'index Elasticsearch
* créer l'index 
* créer le mapping 

```
DELETE logstash-*

PUT logstash-2016.10.19/
{
  "settings": {},
  "mappings": {
    "logs": {
      "properties" : {
        "query_params": {
          "type":"object",
          "properties": {
            "q": {
              "type": "string",
              "analyzer": "standard"
            }
          }
        }
      }
    }
  }
}

```

* réindexer les données depuis logstash 
```
//TODO commande logstash -f logstash.conf
```

Tu peux déjà commencer à tester comment Elasticsearch indexe les données avec un analyzeur pour celà écrire dans sense : 
```
GET /_analyze
{
  "analyzer" : "standard",
  "text" : ["la phrase que l'on veut tester"]
}
```

Cependant tu ne cherches pas à tester l'analyseur standard mais à le remplacer par un analyzeur propre à toi. Ceci se fait dans les paramètres d'un index :

```
DELETE logstash-*

PUT logstash-2016.10.19/
{
  "settings": {
    "analysis": {
      "filter": {
      },
      "analyzer": {
        "mon_super_analyzeur": {
          "tokenizer": "standard",
          "filter": [
            "asciifolding"
          ]
        }
      }
    }
  },
  "mappings": {
    "logs": {
      "properties" : {
        "query_params": {
          "type":"object",
          "properties": {
            "q": {
              "type": "string",
              "analyzer": "mon_super_analyzeur"
            }
          }
        }
      }
    }
  }
}


```
Essaye donc cet analyzeur, que fait-il ?
```
GET logstash-2016.10.19/_analyze
{
  "analyzer" : "mon_super_analyzeur",
  "text" : ["alors ça a bien marché?"]
}
```

Essaye maintenant d'ajouter de nouveaux filtres pour mieux gérer les spécificités de la langue française, pour celà rien de mieux pour découvrir tout ce qui est possible que de regarder la documentation Elasticsearch ou d'en discuter avec les animateurs de cet atelier.

