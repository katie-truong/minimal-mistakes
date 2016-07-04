---
layout: single
title: "How Twitter users reacted to Brexit"
description: "My attempt to use the TwitteR and qdap packages in tweet analysis"
category: R
tags: [r, brexit, twitter, data analysis, politics, TwitteR, qdap]
comments: true
---

"What do you think about Brexit?" asked the interviewer, in my very first job interview in Vietnam, on June 24, 2016. Being unplugged from the Internet for 18 hours on the airplane from US to Vietnam, and another 12 hours of sleep due to jetlag, I was stunned. Being so unexpected as it was, Brexit cost me a job offer. However, also being hugely popular and polarized, it gives me a great opportunity to practice text mining and sentimental analysis. In this very first post, I would demonstrate the process of using the `TwitteR` and `tm` packages to mine and analyze Twitter users' reactions to Brexit on the day June 24, 2016, the very day Brexit happened.

## Getting the data

The process of mining tweets is indeed straightforward, thanks to the awesome `TwitteR` package. However, the prepration is somewhat cumbersome, which I would demonstrate in another post. The goal is get **Consumer Key**, **Consumer Secret**, **Access Token** and **Access Token Secret**, in order for the `setup_twitter_oauth` function to work. 

After setting up authentication, the real process of mining tweets would be fairly easy with only function `searchTwitter` needed, and `twListToDF` in case you want to convert the results to a data frame. 

Let's assume **Consumer Key**, **Consumer Secret**, **Access Token** and **Access Token Secret** be **A**, **B**, **C** and **D** respectively (the real keys and secrets would be series of multiple characters and numbers), the process of getting 10,000 tweets with hashtag **#Brexit** on June 24, 2016 would be:

```r
library(twitteR)

consumer_key = 'A'
consumer_secret = 'B'
access_token = 'C'
access_secret = 'D'

setup_twitter_oauth(consumer_key, consumer_secret, access_token, access_secret)

# Scrape the data, 10000 tweets, language = English
brexit = searchTwitter("#Brexit", n = 50000, since = "2016-06-23", until = "2016-06-24", lang = "en")
brexit = twListToDF(brexit)
```

## Cleaning the data

One important character of tweets is that they contain a lot of emojis, in the form of `latin1` characters, which would be incompatible with the `qdap` package. To eliminate that problem, I convert all characters in the form of `latin1` to `ASCII`.

```r
brexit$text = iconv(brexit$text, "latin1", "ASCII", sub="")
```

URLs are not needed in my analysis, so I also strip all URLs from the text:

```r
brexit$text = gsub(' http[^[:blank:]]+', '', brexit$text)
```

Another thing about sentimental analysis is that it only works with single sentences. If there are more than one sentence in a tweet, it would calculate the result by adding the score of all the sentences together, resulting in an inaccurate calculation. To prevent this, we simply strip all the punctuations in the tweets. Ugly, I know, but it works.

```r
brexit$text = gsub('(\\.|!|\\?)\\s+|(\\++)', ' ', brexit$text)
```

## Calculating the emotional polarity

In order to sentimenal analysis, I use the function `polarity` in the `qdap` package. Note that `qdap` needs `rJava` to run, and in order to install `rJava`, your **Java** and **RStudio** need to be uniform (in either the 32-bit or 64-bit versions). 

The function `polarity` gives use a pretty comprehensive analysis in the polarity of the text (read more about the function [here](http://finzi.psych.upenn.edu/library/qdap/html/polarity.html)). However, in this case, I only need to use the polarity score of each tweet. 

```r
#Using function polarity
brexit$everything = polarity(brexit$text)
#Extracting polarity score
brexit$pol = brexit$everything$all$polarity
```

Positive tweets have positive polarity scores while negative tweets have negative polarity scores. The bigger one's absolute value is, the more positively/negatively polarized it is. Neutral/non-polarized tweets, which have no positive or negative words, have a polarity score of 0. 

To cross check, we look at the top negative tweets (with the lowest polarity scores) in the sample:

```r
brexit$text[head(order(brexit$pol), n = 10)]
```

The results come out to be:

```r
 [1] "For many in the #UK, #Brexit means \"Fuck you, fuck the tories, fuck Cameron, fuck the power, fuck everything\" Sad, but true."
 [2] "Fucking hell #Brexit"                                                                                                          
 [3] "#Remain are confusing hatred with fear and #Brexit are just confused #EUreferendum #EUref"                                     
 [4] "A lot of #Brexit anxiety and really dumb #Brexit jokes on Twitter tonight Time to take a break."                               
 [5] "RT @ThePatriot143: EPIDEMIC - Crazy Violent Left-Wing Extremists #Brexit #DemocraticSitin\nhttps://t.co/ofzYCjRwhx"            
 [6] "RT @felixsalmon: This is a crazy chart The more they have to lose, the more they want to lose it #Brexit"                      
 [7] "'Pot Kettle' methinks #brexit The #incrowd seems to be full of vulgar insults allegations and hatred."                         
 [8] "Squeaky Bum Time #brexit"                                                                                                      
 [9] "MASSIVE BLOW FOR #BREXIT\n\nUKIP ON SUICIDE WATCH"                                                                             
[10] "RT @PrisonPlanet: They interrupted the vote count to exploit Jo Cox's death once again Utterly despicable #Brexit"     
```

We can see that negative tweets are identified very accurately, as negative words and patterns are easy to spot. How about the positive ones?

```r
brexit$text[head(order(-brexit$pol), n = 10)]
```

The results are:

```r
 [1] "Good work  #brexit"                                                                                                                             
 [2] "RT @Bottom_Creek: Y2k type hysteria over #Brexit tells me it's the right move. Tomorrow everyone will be just fine &amp; more secure. Tough shi"
 [3] "And if young Nigel says he's happy\nHe must be happy\nHe must be happy\nHe must be happy in his work\n\n#Brexit"                                
 [4] "#Brexit coverage pretty interesting"                                                                                                            
 [5] "Great result in Sunderland. Massive margin, very promising. I still have hope #EUref #Brexit"                                                   
 [6] "Well done #Swindon, another good win for #LeaveTheEU #InOrOut #Brexit"                                                                          
 [7] "Make Americans Afford Great Britain Again!\n\n#MAGA\n#MAAGBA\n#Brexit"                                                                          
 [8] "#Brexit makes me really thankful for our Supreme Court, which could just go \"haha. No.\""                                                      
 [9] "Good luck #brexit supporters. Hope to see you reclaim your independence."                                                                       
[10] "Very exciting. #Brexit #VoteLeave"                                                                           
```

As we can see, there are sarcastic tweets among genuinely happy tweets. The package is great at detecting happy tweets, but fails to detect sarcasm (which is inevitable, as sarcasm is hard to detect without contexts and relevant information).

As I have the polarity scores, I can proceed to calculate the proportion of positive, negative and neutral tweets in the sample:

```r
#Assign positive, negative and neutral status to the polarity score
orig$polarity = ifelse(orig$pol < 0, "negative", ifelse(orig$pol > 0, "positive", "neutral"))

#Plot
ggplot(brexit, aes(x = as.factor(polarity))) +
  geom_bar(aes(y = (..count..)/sum(..count..))) +
  geom_text(aes(y = ((..count..)/sum(..count..)), label = scales::percent((..count..)/sum(..count..))), stat = "count", vjust = -0.25) + 
  scale_y_continuous(labels = percent) +
  labs(title = "The division in emotional polarity", y = "Percent", x = "Polarity")
```

Which gives me:

![The division in emotional polarity in the whole sample](/images/Brexit-whole-pol.png)

In the whole sample, the neutral tweets accounts for 57.8%, while negative and positive (or sarcastic) tweets have kind of balanced porportions (20.6% and 21.6%). 

## Examine the most retweeted tweets

In this section, I will examine the most retweeted tweets in the sample and see if there is any relation between the number of times a tweet is retweeted and its polarity score.

First, curious as I am, I wanna look at the top 10 retweeted tweets in the sample:

```r
brexit$text[head(order(-brexit$retweetCount), n = 10)]
```

The results come out to be:

```r
 [1] "RT @Snowden: No matter the outcome, #Brexit polls demonstrate how quickly half of any population can be convinced to vote against itself Q"
 [2] "RT @Snowden: No matter the outcome, #Brexit polls demonstrate how quickly half of any population can be convinced to vote against itself Q"
 [3] "RT @Snowden: No matter the outcome, #Brexit polls demonstrate how quickly half of any population can be convinced to vote against itself Q"
 [4] "RT @Snowden: No matter the outcome, #Brexit polls demonstrate how quickly half of any population can be convinced to vote against itself Q"
 [5] "RT @Snowden: No matter the outcome, #Brexit polls demonstrate how quickly half of any population can be convinced to vote against itself Q"
 [6] "RT @Snowden: No matter the outcome, #Brexit polls demonstrate how quickly half of any population can be convinced to vote against itself Q"
 [7] "RT @Snowden: No matter the outcome, #Brexit polls demonstrate how quickly half of any population can be convinced to vote against itself Q"
 [8] "RT @Snowden: No matter the outcome, #Brexit polls demonstrate how quickly half of any population can be convinced to vote against itself Q"
 [9] "RT @Snowden: No matter the outcome, #Brexit polls demonstrate how quickly half of any population can be convinced to vote against itself Q"
[10] "RT @Snowden: No matter the outcome, #Brexit polls demonstrate how quickly half of any population can be convinced to vote against itself Q"
```

Not so surprisingly, the top 10 retweeted tweets all come from a famous source, in this case, [Edward Snowden](https://twitter.com/Snowden). With the same trick, I can check the polarity scores of the tweets:

```r
brexit$pol[head(order(-brexit$retweetCount), n = 10)]
```

Which all come out to be zero.

```r
[1] 0 0 0 0 0 0 0 0 0 0
```

How about the origional tweets? First, we subset the origional tweets in the sample:

```r
orig = subset(brexit, !isRetweet) 
```
Then, we apply the same process to the origional tweets sample:

```r
#Get the text
orig$text[head(order(-orig$retweetCount), n = 10)]
#Get the polarity score
orig$pol[head(order(-orig$retweetCount), n = 10)]
```

The results come out to be:

```r
#Text
orig$text[head(order(-orig$retweetCount), n = 10)]
 [1] "Fascinating poll on Brexit. Broken down by age. @YouGov https://t.co/N4XlA4lF0L #Brexit"                                                                       
 [2] "We have rare footage of the original #BREXIT https://t.co/gH2pZhGL6S"                                                                                          
 [3] "Twitter has censored every #Brexit related hashtag. This is a huge international issue that everyone is talking about."                                        
 [4] "The working class are voting #Brexit  Those paid and elected to represent them have lost touch with their members and voters #Lexit"                           
 [5] ".@MariaBartiromo on #Brexit vote: \"When you come down to it, this is really about immigration.\" #Greta https://t.co/Q7kUS5kzXW"                              
 [6] "Can't sleep, #Brexit will eat me https://t.co/sjffeTbuNa"                                                                                                      
 [7] ".@DanHannanMEP on #Brexit vote: \"This is not about walking away from Europe. It's about repatriating laws.\" #Greta https://t.co/aafQC3pfSZ"                  
 [8] "&gt;Only talk to \"journalists\"\n&gt;Close comments\n&gt;Block people on twitter\n&gt;Should down everyone as racist\n&gt;Be surprised that #Brexit is at 50%"
 [9] "Current #Brexit tally https://t.co/CmtSPW4vAC"                                                                                                                 
[10] "LIVE: Latest results and reaction from global markets as #EuRef swings in favor of #Brexit https://t.co/FAMpDhBto8 https://t.co/VvHLdjjmQX"                    
#Polarity score
orig$pol[head(order(-orig$retweetCount), n = 10)]
 [1]  0.0000000  0.0000000 -0.4242641 -0.2132007  0.0000000  0.0000000  0.0000000
 [8] -0.1961161  0.0000000  0.2085144
```

So what is the pattern? Which kind of tweets tend to get retweeted more? To answer this question, first, I look into the mean of the number of retweets per group:

```r
tapply(brexit$retweetCount, brexit$polarity, mean)
```

Which gives me:

```r
 negative   neutral  positive 
 453.9714 1726.9374 1857.1151 
```

 According to the calculation, positive tweets have the highest average number of retweets; neutral tweets have the second place; and negative tweets have the lowest number of retweets. However, notice that the number of retweets are **really really** high, which suggests there are outliers in the sample which skew the result. In this situation, I decide to look into the median as an alternative:
 
```r
 tapply(brexit$retweetCount, brexit$polarity, median)
```
 
 Which comes out to be:
 
```r
negative  neutral positive 
       6        5        6 
```

As we can see, unfortunately, the mean and median don't agree. The median number of retweets of the neutral group turns out to the lowest at 5, while the median number of retweets of negative and positive tweets are slightly higher at 6. 

What can I do in this case? I decide to make a scatterplot of the polarity score against the number of retweets to see the overall pattern of 10,000 tweets in the sample:

```r
ggplot(brexit, aes(x = pol, y = retweetCount)) +
  geom_point(position = 'jitter') + #avoid overplotting
  geom_smooth() + #confident interval
  ggtitle("Polarity vs Retweet") + 
  xlab('Polarity') +
  ylab("Number of Retweets")
```

Which gives us the plot:

![Polarity vs Number of Retweets](/images/Brexit-pol-vs-retweet.png)

As the plot is hard to look at with the outliers, I decide to make another plot with a closer look:

![Polarity vs Number of Retweets](/images/Brexit-pol-vs-retweet-closer.png)

We can see the overall pattern that, the top retweeted tweets are neutral tweets, while positive (or sarcastic tweets) tend to get retweeted a little bit more. So, the people in the sample are either very happy, or very sarcastic about Brexit, based on the way they retweet. 

## Top favorite tweets

Another indication of sentiment on Twitter are number of favorite. I would simply apply the same process I did with number of retweets in this case.

So the top favorited tweets are:

```r
 [1] "Fascinating poll on Brexit Broken down by age @YouGov #Brexit"                                                                                                 
 [2] "We have rare footage of the original #BREXIT"                                                                                                                  
 [3] "Twitter has censored every #Brexit related hashtag This is a huge international issue that everyone is talking about."                                         
 [4] ".@MariaBartiromo on #Brexit vote: \"When you come down to it, this is really about immigration.\" #Greta"                                                      
 [5] "As minutes tick by, I'm discovering I have extremely strong feelings about #Brexit!"                                                                           
 [6] "The working class are voting #Brexit  Those paid and elected to represent them have lost touch with their members and voters #Lexit"                           
 [7] "&gt;Only talk to \"journalists\"\n&gt;Close comments\n&gt;Block people on twitter\n&gt;Should down everyone as racist\n&gt;Be surprised that #Brexit is at 50%"
 [8] ".@DanHannanMEP on #Brexit vote: \"This is not about walking away from Europe It's about repatriating laws.\" #Greta"                                           
 [9] "Hah The experts on the BBC are now starting to suggest that....gasp.. we WILL #Brexit"                                                                         
[10] "Can't sleep, #Brexit will eat me"                                                                                                
```

And their polarity scores:

```r
 [1]  0.0000000  0.0000000 -0.4242641  0.0000000  0.4992302 -0.2132007 -0.1961161
 [8]  0.0000000  0.0000000  0.0000000
```

The mean numbers of favorites per tweet per group are:

```r
negative  neutral positive 
1.420543 1.492129 1.110441 
```

And the median:

```r
negative  neutral positive 
       0        0        0 
```

Really low numbers, comparing to the number of retweets. 

The graph of polarity score against number of favorites:

![Polarity vs Number of Favorites](/images/Brexit-polarity-vs-favorite.png)

We can see that the neutral tweets tend to be more favorited than positive or negative tweets.

Finally, I would like to see the relationship between number of retweets and number of favorites (although we already see the users in the sample is somewhat lukewarm with the heart button). With a few lines of code, we can easily make a scatterplot:

```r
fav.vs.retweet = ggplot(brexit, aes(x = favoriteCount, y = retweetCount))
fav.vs.retweet + geom_point(position = "jitter") + geom_smooth() + ggtitle("Favorite vs Retweet") + xlab("Number of favorites") + ylab("Number of retweets")
```

The plot turns out to be:

![Favorite Counts vs Retweet Counts](/images/Brexit-fav-vs-retweet.png)

As we can see, there is a very small correlation between the number of retweets and the number of favorites. In fact, many retweeted tweets get zero favorites, just to prove how merr the users in the sample are to the favorite function.

## Conclusion

### About Brexit
- The reactions to Brexit on Twitter are very polarized. There are people who are devastated, and people who are genuinely happy about Brexit.
- People are also very sarcastic toward how Brexit turned out be (as someone said, "sarcasm is the wit")

### About the analysis
- Sentimental analysis by the function `polarity` in the `qdap` package works very well with straightforward tweets, but doesn't work well with sarcasm.
- The top retweeted tweets on Twitter tend to be neutral tweets.
- As a non-Twitter user, I am not sure if the nonchalance towards the favorite function only applies to this sample, or to Twitter as a whole. Maybe a Twitter user can shed me some light on this?

I guess this is the end of my first post on this blog. Thanks for reading through such a very long, detailed post. Please leave a comment so I can improve my contents, and stay tuned for the next posts.

Cheers,

Katie
