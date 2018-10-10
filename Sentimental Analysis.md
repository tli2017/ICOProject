# ICO Project Code

## Sentimental Analysis

The URLs of ICOs were read and converted into `tidy text` format for further anlaysis in `R`. Then extract sentiment variables through 3 lexicons namely `afinn, bing, nrc` and put them in a linear regression with initial price.

```R
# Real all CSV file containing price and URLS
library(readr)
sample_data<- read_csv("ICO_Rong.csv")
Prices=c(sample_data$Prices)
URLs=c(sample_data$URL)

#extract afinn sentiment  
n=nrow(sample_data)
affinscore=rep(0,n)
for (k in 1:n){
    tryCatch({
        tidydf=tidy_text_func(URLs[k])
        afinn <-tidydf %>%
        inner_join(get_sentiments("afinn")) %>%
        summarise(sentiment = sum(score)) %>%
        mutate(method = "AFINN")
        affinscore[k]=afinn$sentiment
    }, error=function(e){})
}

#extract bing sentiment
positive=rep(0,nrow(sample_data))
negative=rep(0,nrow(sample_data))
for (k in 1:nrow(sample_data)){
    tryCatch({
        tidydf=tidy_text_func(URLs[k])
        bing_sent<-tidydf %>%
        inner_join(get_sentiments("bing"))%>%
        count(sentiment)        
        negative[k]=bing_sent$n[1]
        positive[k]=bing_sent$n[2]
    }, error=function(e){})
}

# extract nrc sentiment with 10 categories
anger=rep(0,nrow(sample_data))
anticipation=rep(0,nrow(sample_data))
disgust=rep(0,nrow(sample_data))
fear=rep(0,nrow(sample_data))
joy=rep(0,nrow(sample_data))
negative=rep(0,nrow(sample_data))
positive=rep(0,nrow(sample_data))
sadness=rep(0,nrow(sample_data))
surprise=rep(0,nrow(sample_data))
trust=rep(0,nrow(sample_data))
for (k in 1:nrow(sample_data)){
    tryCatch({
        tidydf=tidy_text_func(URLs[k])
        nrc <-tidydf %>%
        inner_join(get_sentiments("nrc")) %>%
        count(sentiment)
        anger[k]=nrc$n[1]
        anticipation[k]=nrc$n[2]
        disgust[k]=nrc$n[3]
        fear[k]=nrc$n[4]
        joy[k]=nrc$n[5]
        negative[k]=nrc$n[6]
        positive[k]=nrc$n[7]
        sadness[k]=nrc$n[8]
        surprise[k]=nrc$n[9]
        trust[k]=nrc$n[10]
    }, error=function(e){})
}

# linear regression between prices and sentiments
linearaffin<- lm(Prices ~ affinscore,data=sample_data) 
summary(linearaffin)

linearbing<- lm(Prices~negative+positive, data=sample_data) 
summary(linearbing)

linearnrc<- lm(Prices ~ anger+anticipation+disgust+fear+joy+negative+positive+sadness+surprise+trust, data=sample_data)
summary(linearnrc)
```

