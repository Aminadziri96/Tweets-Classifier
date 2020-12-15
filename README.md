# Tweets-Classifier
classification Tweets avec Kmeans aprés les extrairer
![Alt Text](https://miro.medium.com/max/1600/1*K5r1UXVuYmVuqXiKkuX5tg.gif)

## <a style="color:#CDCD43" font=" Bookman"  size="75" > Extraction de tweets   </a> 

Pour faire du machine learning sur des tweets il faut des tweets  Pour cela deux options se présentent. La première est d’utiliser l’API que Twitter lui-même propose. Elle permet de récupérer des tweets en ajoutant certaines conditions sur le type de tweets que vous souhaitez.

   # API credentials here
     consumer_key = "oCXOidLwh4iJdfx7jE1eA92oE"
     consumer_secret = "zOs8rMnEwL5ggheUacMdSIzJZ74QZQV01WZrDjWfeGvZG5USi9"
     access_token = "1327999624171368450-z174EGborRp3e6Tr9irVpUCPDFL3lL"
     access_token_secret = "y6YcPwMDP78FADpvAgZ9zOUEFSt8HRTdJj3nWjAXnVxiw"


![Image of Yaktocat](https://tvsarawak.com/wp-content/uploads/2020/04/1ACOVERPHOTO-6.jpg)

La seconde option est l’utilisation du module Python Twitterscrapper. Vous pouvez extraire un grand nombre de tweets en spécifiant des critères de date, de langues et en vous limitant aux tweets qui contiennent certains mots-clés. Exemple:


      import tweepy #https://github.com/tweepy/tweepy
      import csv
     #Twitter API credentials
     consumer_key = ""
     consumer_secret = ""
      access_key = ""
      access_secret = ""
    def get_all_tweets(screen_name):
    #Twitter only allows access to a users most recent 3240 tweets with this method
    
    #authorize twitter, initialize tweepy
    auth = tweepy.OAuthHandler(consumer_key, consumer_secret)
    auth.set_access_token(access_key, access_secret)
    api = tweepy.API(auth)
    
    #initialize a list to hold all the tweepy Tweets
    alltweets = []  
    
    #make initial request for most recent tweets (200 is the maximum allowed count)
    new_tweets = api.user_timeline(screen_name = screen_name,count=200)
    
    #save most recent tweets
    alltweets.extend(new_tweets)
    
    #save the id of the oldest tweet less one
    oldest = alltweets[-1].id - 1
    
    #keep grabbing tweets until there are no tweets left to grab
    while len(new_tweets) > 0:
        print(f"getting tweets before {oldest}")
        
        #all subsiquent requests use the max_id param to prevent duplicates
        new_tweets = api.user_timeline(screen_name = screen_name,count=200,max_id=oldest)
        
        #save most recent tweets
        alltweets.extend(new_tweets)
        
        #update the id of the oldest tweet less one
        oldest = alltweets[-1].id - 1
        
        print(f"...{len(alltweets)} tweets downloaded so far")
    
    #transform the tweepy tweets into a 2D array that will populate the csv 
    outtweets = [[tweet.id_str, tweet.created_at, tweet.text] for tweet in alltweets]
    
    #write the csv  
    with open(f'new_{screen_name}_tweets.csv', 'w') as f:
        writer = csv.writer(f)
        writer.writerow(["id","created_at","text"])
        writer.writerows(outtweets)
	 pass
      if __name__ == '__main__':
      #pass in the username of the account you want to download
      get_all_tweets("J_tsar")
      
      
      
## <p style="color:#CDCD43" font=" Bookman"  size="75" > J’ai choisi de récupérer les tweets liée avec les cathégories suivantes: economic, health, sport, movies et education  tous sont extrait de la language english fixé dés le début dans notre code avec credentiels et dans le but d'avoir 10000 tweets.   </p> 

# <p style="color:#CDCD43" font=" Bookman"  size="75" > Nettoyage et construction du pipeline NLP </p>

## <p style="color:#CDCD43" font=" Bookman"  size="75" >Une rapide inspection de la base nous permet de voir que la compréhension de certains tweets est difficile. Le nettoyage sera d’autant plus important. </p>

![Alt Text](https://laforgecollective.fr/wp-content/uploads/2018/07/ANIM-NETTOYAGE.gif)

##  <p style="color:#CDCD43" font=" Bookman"  size="75" > En NLP, on commence toujours par construire un pipeline de nettoyage des données. Personnellement j’utilise les Reg-ex avec le module Python re qui permettent de faire cela facilement.

Le nettoyage des tweets comprendra plusieurs choses :

 - Enlever les emojis 
 - Retirer la ponctuation : très facile avec les reg-ex
 - Retirer les caractères spéciaux : très facile avec les reg-ex mais tous les caractères ne seront pas retirés dans un premier temps. Les tweets sont des objets très sales !
 - Retirer les chiffres : avec une Reg-ex aussi
 - Changer les lettres majuscules en minuscules
 </p>

![Image of Yaktocat](https://www.kdnuggets.com/wp-content/uploads/text-preprocessing-framework-2.png)

Voilà à quoi ressemble notre pipeline :
       
     import re
    def nlp_pipeline(text):

    text = text.lower()
    text = text.replace('\n', ' ').replace('\r', '')
    text = ' '.join(text.split())
    text = re.sub(r"[A-Za-z\.]*[0-9]+[A-Za-z%°\.]*", "", text)
    text = re.sub(r"(\s\-\s|-$)", "", text)
    text = re.sub(r"[,\!\?\%\(\)\/\"]", "", text)
    text = re.sub(r"\&\S*\s", "", text)
    text = re.sub(r"\&", "", text)
    text = re.sub(r"\+", "", text)
    text = re.sub(r"\#", "", text)
    text = re.sub(r"\$", "", text)
    text = re.sub(r"\£", "", text)
    text = re.sub(r"\%", "", text)
    text = re.sub(r"\:", "", text)
    text = re.sub(r"\@", "", text)
    text = re.sub(r"\-", "", text)

    return text


## <p style="color:#CDCD43" font=" Bookman"  size="75" > Tout d'abord je réalise cleaning des tweets d'une seule DataSet dans le but de mieux comprendre les fonctions de cleaning géré par: NLP.Ensuite, on fait la concaténation de l'ensemble des fichiers csv en une MegaDataFrame </P>

      import pandas as pd
      # from twitter. Pick out the guys with popularity > 10k.
      _1 = pd.read_csv('economy_tweets.csv')
     _2 = pd.read_csv('education_tweets.csv')
     _3 = pd.read_csv('health_tweets.csv')
     _4 = pd.read_csv('movies_tweets.csv')
     _5 = pd.read_csv('sport_tweets.csv')
                  
    mega_df = pd.concat([ _1, _2, _3, _4, _5])
    mega_df.shape  
    
##  => 10K tweets un nombre important dans le but d'avoir une meilleurs classification