# QTM 250 HW4 

# Influences of Length and Presence of Slangs in Comments on Sentiment Analysis with Google Natural Language API 
## Group 8: Alisa Chen, Sunoo Lee, Curstynn Marks, Nathaniel Thomas, Tianyi Xu, Laura Yu

## How to use this Github Repo?
You can take a look at the readme file to learn about our project goals and results. The same information can be found on the Google Colab Notebook in the repo. You will also find the original dataset with 34K posts of bodybuilders' comments, the dataset that we created to go through the API which contains a subset from the original dataset, and the final output dataset with visualizations. The graphs in png format are also uploaded here.

## Introduction 
<img src="https://pbs.twimg.com/media/C--XnvaXcAANSHH.jpg" width="600">

**Natural Language API**

The Natural Language API provided by Google Cloud contains powerful and well-trained models to apply natural language understanding (NLU) to text in a varity of languages. Available applications include sentiment analysis, entity analysis, content classification, syntax analysis, and so on. It can process different types of text, and is used widely by developers around the world. 


[Introduction of Natural Language API](https://cloud.google.com/natural-language)

Two of the qualities of sentiment measure by the Natural Language API are polarity and magnitude. Polarity is the distinction between positive and negative attitudes in the text, while magnitude is the amount of emotional contect present in the text. In this study, we are only interested in the polarity output, because it has direct association with whether the comment is positive or negative, and we have less focus on the magnitude of such emotions. 

**Testing the API's Performance**

But how well does the API determine emotional valence within a text? Presumably, a sophisticated text analysis tool within the Google suite of applications would be able to assess emotional context with pinpoint accuracy. But what if the API was tasked with analyzing the vernacular of a group of individuals so far gone that their main mode of communication is essentially screaming and beating rocks with sticks? That's right! We're talking about bodybuilders. 

Naturally, when these meatheads take to the Internet to let out their testosterone-induced gripes with their world (aka. the local LA Fitness), they aren't speaking with the same sophistication expected of an Emory student diligently studying QTM 250. In fact, just like you don't expect one of these neanderthals to bushwhack at you at Chipotle and start talking to you about the various benefits of consuming BCAA proteins, you don't expect them to talk within the same parameters an ordinary person would. For example, when you hop on Reddit's r/bodybuilding thread or the forums on bodybuilding.com, you can see posts ranging from a simple "lol brah blast tren" (translation = Good day, sir; consider taking steroids.) to a 5 paragraph war-torn battle story about how _barbelldaddy441 struggled his whole life being skinny before turning his life around with hard work in at his highschool gym. Furthermore, a community like this has endless jargon. Aside from being frequent users of standard US slang like "bruh," "ya," and "sorta," bodybuilders also use a lot of terms of their own. For example, "juice" or "sauce" means steroids, while "yoked" or "jacked" means muscular in an aesthetic way. 

Clearly, there is a vast range of outcomes possible when someone like a bodybuilder sits down to type at their laptop, perhaps too many for even machine learning to keep up with. To test this, we present Google's Natural Language API with a dataset of posts taken from Reddit's r/bodybuilding - some short, some long, some with slang, some without- and task it with determining the sentiment of each. Then we, the researchers, assess the same posts ourselves to benchmark how well the API performed with the different types of posts. Regarding two variables of the length and the existence of slang, we hypothesize that the API will better predict the sentiments of the posts with less text and slang than posts that are longer (as more complicated emotions are often associated with longer text) and riddled with words that nobody, especially not a machine, has ever heard before. 

## Dataset

[Bodybuilder Comment Dataset](https://docs.google.com/spreadsheets/d/1Bt4vdszWbz7Om_CqfmQCUksE2gwLW4PFFO4ZxPmyeAw/edit?usp=sharing)


The [original data](https://docs.google.com/spreadsheets/d/1codXYGE5NhuDlGAau2o1NeuWuZYtsWyjbZTiMoX9JIA/edit#gid=464655030) we use to test our hypothesis was scraped by one of our researchers (Nathaniel Thomas) with a classmate (Se Eun Kim) in a previous class (QTM 340) using the PRAW Python library. The dataset contained around 34k posts from the subreddit "r/bodybuilding." 

<img src="https://media.istockphoto.com/photos/silhouette-of-a-strong-fighter-picture-id479009182?k=20&m=479009182&s=612x612&w=0&h=sqGloVztJZrk245qEFTRB2EAxZ_0i2QNJytxtLPtoJ4=" width="350">

As mentioned in the introduction, we are interested in two qualities of the comment inputs that may act as variables influencing accuracy of the Natural Language API. One quality is length of the comment, measured by numbers of words. The other quality is whether there are slangs used in the comments. A subset of 100 comments - 50 with slang and 50 without - were selected from the dataset with more than 34K entries for efficiency. 

We used the following code on Excel to count the number of words in each comment, and save the data in the column named "Length". 

```
# =IF(LEN(TRIM(A2))=0,0,LEN(TRIM(A2))-LEN(SUBSTITUTE(A2," ",""))+1)
```

We went through all 100 comments and assigned 0 if there was no slang present, and 1 if there was at least one slang used in the comment. The data were saved in the column named "Slang".

For our purpose to compare human and machine learning sentiment analyses, our group members read the comments and assigned a value of attitude. -1 represents negative comments, 0 represents neutral comments, and 1 represents positive comments. The data were saved in the column named "Sentiment".

## Natural Language API

```
from google.colab import auth
auth.authenticate_user()
from oauth2client.client import GoogleCredentials

import getpass
from google.cloud import language_v1
```
Make sure the API is enabled on your cloud account. If a key is needed when running this cell, enter the key below in the first executable cell.

**The key is AIzaSyDbpkjDnwuTUqNV6D17SYJ8fNnTDxuvf28**
```
APIKEY = getpass.getpass()
CLIENT_ID = '301744682927-jquak0bjs22l247is9revkp1cnn3sbqu.apps.googleusercontent.com'
APIKEY
```
```
!pip install --upgrade pip

!pip install --upgrade google-api-python-client
```
```
from googleapiclient.discovery import build
```
Import the package.
```
lservice = build('language', 'v1beta1', developerKey=APIKEY)
```
Import Bodybuilder Comment Dataset from Google Cloud (Public Access).
```
import pandas as pd
comments = pd.read_csv('https://storage.googleapis.com/qtmgroup8/Copy%20of%20Bodybuilder%20Dataset%20-%20Final.csv')
comments
```
Create a dataframe.
```
final = pd.DataFrame(columns=['comment','polarity','magnitude'])
pol = []
mag = []
com = [] 
```
Run the API.
```
for comment in comments['Comment']:
  response = lservice.documents().analyzeSentiment(
    body={
      'document': {
         'type': 'PLAIN_TEXT',
         'content': comment
      }
    }).execute()
  polarity = response['documentSentiment']['polarity']
  magnitude = response['documentSentiment']['magnitude']
  pol.append(polarity)
  mag.append(magnitude)
  com.append(comment)
```
Mount Google Drive. You would need to have this folder shortcut in your Google Drive account and that will do the job: /content/drive/My Drive/QTM250group8.
```
import os
from google.colab import drive
drive.mount('/content/drive/')
os.chdir('/content/drive/My Drive/QTM250group8') 
```
Export output to the Google Drive Folder (See Directory Above). You will find the csv file named final output in the directory.
```
final['polarity'] = pol
final['magnitude'] = mag
final['comment'] = com
final
final.to_csv('final output.csv',index=False)
```
## Results

The polarity output calculated by the Natural Language API has a range from -1 to 1, and it represents the difference between positive and negative attitudes in the text. Therefore, if the value is closer to -1, the attitude would be more negative; if the value is closer to 1, then the attitude would be more positive.

We calculated the difference between sentiment values assigned by our group members and the polarity output assigned by the machine learning API:

= (Human Sentiment Analysis - Machine Learning API Sentiment Analysis)

If the result is positive, it indicates that the API picks up more negativity than human does. If the result is negative, it indicates that the API picks up more positivity than human does. 

The graphs below were produced by Google Sheet. See link below for our final output.

[Final Output](https://docs.google.com/spreadsheets/d/1OMv1ldRS5YJKDzoKqVzskNDwBkKwANuYtb-8-Rrkxpg/edit?usp=sharing)

**Distribution of Difference between ML and Human Sentiment Analysis**
<img src="https://docs.google.com/spreadsheets/d/e/2PACX-1vSqMJPB3fzpDPJkC9dXd6BVyS-U1NU-2vi1juzmAVdEDCwFttpYpaLFOiyTHkyJhXtw12-XvmmxJGEQ/pubchart?oid=8988792&format=image" width="600">

Based on the graph, we can see that there are slightly more comments with positive values than negative values, but most of the values are in the middle. To be specific, it indicates that over 40% of the results are concentrated in the center, with a value bewteen -0.2 to 0.25. Another 35% of the results have a value above 0.25, and the rest of the results are mainly less than 0. 

With a bell shape, it can be concluded from the graph that the Natural Language API is generally accurate, since most of the differences between the values provided by ML and human are close to 0. However, we can also see a large proportion of the results with an absolute difference value larger than 0.5, which could potentially mean that there are still plenty of differences between human and machine learning sentiment analyses, in either direction. Since the difference was calculated by human analysis - machine learning API analysis, positive values indicate that machine learning API picks up more negativity than human does, and vice versa. As we can see, 35% of all values are above 0.25, which suggests that the API tends to recognise negative sentiments in the comments more than our group members. One possible explanation for this observation is that it might be easier for the API algorithm to pick up vocabularies that are more negative, such as negations, negative adjectives, and curses, while many positive comments might seem to be more neutral and harder for the API to pick up sentiments.

**Difference between ML and Human Sentiment Analysis vs. Length of Comments**
<img src="https://docs.google.com/spreadsheets/d/e/2PACX-1vSqMJPB3fzpDPJkC9dXd6BVyS-U1NU-2vi1juzmAVdEDCwFttpYpaLFOiyTHkyJhXtw12-XvmmxJGEQ/pubchart?oid=1387780684&format=image" width="600">

Given length of text as a variable, the API can still accurately analyze the sentiment of comments, at least not totally opposite, because most of the datapoints are scattered between 1 and -1. 

Regardless of the length of comments, we can see that there are more positive values above the line of "Difference = 0" than negative values below, indicating the same conclusion we drew from the first graph that machine learning API picks up more negativity. However, as length of comments increases, the difference values are more scattered between 1 and -1 (but not heavily skewed in one direction), unlike many datapoints with 0 difference in shorter comments. Therefore, we can also conclude that the length of comment may somewhat affect accuracy of the API analysis, and more emotions included in longer text might potentially be a factor that confuses the machine learning.

**Difference between ML and Human Sentiment Analysis vs. Presence of Slang in Comments**
<img src="https://docs.google.com/spreadsheets/d/e/2PACX-1vSqMJPB3fzpDPJkC9dXd6BVyS-U1NU-2vi1juzmAVdEDCwFttpYpaLFOiyTHkyJhXtw12-XvmmxJGEQ/pubchart?oid=1070952019&format=image">

The boxplot above depicts the difference in sentiment analysis conducted by the researchers and the API for the two types of posts: posts with slang and posts without. The larger box on the right suggests a much larger discrepancy in the researchers’ and machine’s sentiment analysis for the posts with slang. This means that the machine had a much harder time understanding the emotion of comments that used slang, and often misidentified the positive, negative, or neutral qualities of posts. The smaller box on the left suggests that when analyzing the sentiment of comments without slang, the machine is more spot-on, giving a similar assessment to that of the researchers. We can therefore conclude that the presence of slang in a post could be a hidden variable that affects the output, perhaps adversely, of the API’s analysis. 

## Conclusion

Thanks to the ingenuity - or perhaps stupidity - of bodybuilders and their outlandish vernacular, we can conclude that our hypothesis was supported. According to our testing, length of a post and the presence of slangs are both hidden variables that can decrease the accuracy of Google’s Natural Language API. Ultimately, it seems that the Cloud’s sophisticated text analysis tool isn’t as smart as we may have assumed. Maybe it should try bodybuilding!

Some limitations of our study include the fact that we did not include an equal amount of long and short posts within the dataset. This may have prevented us from getting the best conclusion whether the machine did better on analyzing long or short posts. Limitations also include the fact that the researchers only determined the positivity, negativity and neutrality of comments with the values 1, -1, and 0. The machine, on the other hand, can assess valence with every value between -1 and 1. This discrepancy may also lead to some inaccuracy in our analysis. If replicated, these limitations should be controlled for. 

## Architectural Diagram

How does the Natural Language API work?

Text files are processed through the Natural Language API to analyse sentiment, entities, and syntax. The API then outputs the scoring of polarity, magnitude, and the count of text characteristics of the method chosen. The API utilizes machine learning by analyzing thousands of text files to create a baseline for scoring measures, and the API continues learning when other users feed data from their own projects. Users are then also able to construct their own measures for any of the methods. For our project, we utilized the sentiment and syntax methods to develop a measure for the use of slang. We also measured the polarity of sentiment and syntax via the length of comments. 

[Google Architecture Diagramming Tool](https://cloud.google.com/icons)
![Diagram](https://github.com/curstynn/qtm250-example/blob/408dc01085ab0b0bf1c82fa6708a004d98a593af/qtmdia.png?raw=TRUE)

Step 1: Enable API on CLoud

Step 2: Import Bodybuilder Comments from Google Cloud (Public Access)

Step 3: Create a dataframe

Step 4: Run dataframe through the API

Step 5: Mount Google Drive

Step 6: Export output to Google Drive Folder to created directory
