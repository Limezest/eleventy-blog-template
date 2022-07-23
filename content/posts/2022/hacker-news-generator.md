---
title: Hacker News Generator
date: 2022-07-20
image: /assets/images/2022/hacker-news-generator/header.png
additionalTags:
    - development
    - personal projects
---

> Transform your ideas into reality on Google Cloud!

The time was 1:30PM, I was just coming back from lunch break at the office.  
«Let's just take a quick look at [Hacker News][hacker-news] before getting back to work» I thought.

I remember one of the top post being _“[My business card runs Linux][business-card-linux]”_ and thinking:  
_«What a HackerNews-y title, I bet if I had a script generate typical titles we would end up with a similar one.»_  
<sup><sub>(not to point the finger at this one in particular)</sub></sup>

Challenge accepted!

There went the rest of my lunch break. In about 30 minutes, I had a link I could share with my colleagues :)

## Iterating from a proof of concept

### The data

First, I had to look for historical data I could build onto. 🧐  
Fortunately I had the intuition these would be accessible in a public BigQuery dataset. A bit of searching later, I found `bigquery-public-data.hacker_news.stories`!

BigQuery is a cloud-based data warehouse with an easily accessible query engine API.  
In a typical project, you would use BigQuery to store any long-term data from your applications, such as logs, that you would then use for analytics, or application data that you archive.  
BigQuery also allows you to query public datasets from other users.

<br/>

![bigquery-dataset-details][bigquery-dataset-details]
<br/><br/>

As you can see on this screenshot, the whole dataset holds 1,959,808 records of stories posted between October 2005 and October 2015!

After a bit of data exploration, I decided to extract only the posts with a score above 100 upvotes to restrict the titles I will be working with.  
With a very simple SQL query I got myself around 43,000 lines of data out of the nearly 2 millions, which BigQuery output in a single second!

```SQL
SELECT
  title
FROM
  `bigquery-public-data.hacker_news.stories`
WHERE
  score > 100
```

Using the bq command line, I ran this query and extracted all titles separated by newlines into a local file.

```shell
bq query \
  --format csv \
  --use_legacy_sql=false \
  'SELECT title FROM `bigquery-public-data.hacker_news.stories` where score > 100' \
  > hn_title.csv`
```

### The logic

My plan to generate new sentences from past ones was to use Markov chains.

Markov chains are relatively simple to understand: they are a way of generating a sequence of random words based on a simple rule:

-   The rule is that the next word in the sentence is picked based on the two previous words.
-   The first two words in the sequence are always randomly picked from a sentence in the corpus.

Using the two previous words to pick a third one means having a "state size" of 2.  
By increasing the size of your state you can avoid jumping from totally unrelated sentences. However, you will be decreasing the number of possible outcomes.  
To keep the sentences less random, since the titles were already using some very precise technical jargon, I settled with a size of 3.

<br/>
To avoid reinventing the wheel, let's import Markovify: 😄
```shell
pip install markovify
```

```py
import markovify

with open("hn_title.csv") as f:
    text = f.read()

text_model = markovify.NewlineText(text, state_size=3)

def get_title():
    return text_model.make_short_sentence(280)

if __name__ == "__main__":
    get_title()
```

```shell
$ python main.py

> [Output] Client-side full text search in milliseconds with PostgreSQL
```

At this point I had my proof of concept, now my next step was to host this piece of code to share it with my colleagues without losing too much time and money in the process.  
Having the title generation defined as a python function, my natural choice was to push it to Cloud Functions.

Cloud Functions is a perfect destination for such use-cases.  
< blablabla cloud functions />

With a command line as easy as:

```shell
$ gcloud functions deploy hn_title \
  --runtime python37 \
  --entry-point get_title \
  --trigger-http \
  --region europe-west1

> [Output] https://location-project.cloudfunctions.net/hn_title
  Why we killed our startup in 100 days
```

My project was online and accessible through an https URL!

Now I only see one small problem: latency.  
As a matter of fact, Cloud Functions are started and stopped as needed.  
So my whole app, especially the `markovify.NewlineText`, would be instanciated right when a user loads the page, taking a few seconds to load the several megabytes of corpus into memory, only to be destroyed when they leave, repeating this process for each user. :(

To solve this issue, I would only need to load the model into memory once and make all following requests reuse it.  
That sounds like how webservers work with concurrency, and fortunately I know just the thing to transform a single function into a webserver that i can customize:  
Enter functions-framework!

Functions-framework will wrap your python function inside a pre-configured Flask webserver.  
To run locally you first, all you have to do is:

```shell
pip install functions-framework
functions-framework --target route_request --debug
```

And voilà! Your function is now accessible at localhost:8080 just like Cloud Functions. Except this time it won't by destroyed whenever a user exits your app.

By adding this Dockerfile next to my main.py file, I was able to build a container running my app inside functions-framework!

<br/>

```Dockerfile
FROM python:3.7-slim

# Copy local code to the container image.
ENV APP_HOME /app
WORKDIR $APP_HOME

# Install production dependencies.
COPY requirements.txt .
RUN pip install gunicorn functions-framework
RUN pip install -r requirements.txt

COPY . .
# Run the web service on container startup. Here we use the gunicorn
# webserver, with one worker process and 8 threads.
# For environments with multiple CPU cores, increase the number of workers
# to be equal to the cores available.
CMD exec gunicorn --bind :$PORT --workers 1 --threads 8 -e FUNCTION_TARGET=get_title functions_framework:app
```

My directory looks like this:

```shell
$ tree
.
├── Dockerfile
├── hn_title.csv
├── main.py
└── requirements.txt
```

Next step is to deploy our new container onto a service that is specially made for this: Cloud Run.

```shell
$ gcloud run deploy hntitlegen \
  --allow-unauthenticated \
  --source=.

> [Output] https://hntitlegen-uanwpkxudq-ew.a.run.app/
  Why Dart is not the language of choice for data science
```

Cloud Run is fantastic! Think running your applications flexibly like in Kubernetes and deploy easily like in Cloud Functions.  
No underlying infrastructure to provide, configure and maintain up-to-date!  
You just build your code into a container (and we've just seen it's as easy as bringing a Dockerfile) and then referencing your newly-built image.

Cloud Run provides scale-to-zero capability, which means once my service stops receiving request for a few minutes the container will entirely stop, saving both money and energy.  
The only drawback to this, is the first request to hit my service back after a scale-down event has to wait until the container comes back up, which in our case can be a few seconds long. We call this a __cold-boot__.  
Fortunately, once instanciated for any single request our container will be warm for all following requests, with our markov text-model retained in memory and ready to serve the next sentence instantly!


[hacker-news]: https://news.ycombinator.com
[github-repo]: https://github.com/Limezest/hacker-news-generator
[business-card-linux]: https://www.thirtythreeforty.net/posts/2019/12/my-business-card-runs-linux/
[bigquery-dataset-details]: /assets/images/2022/hacker-news-generator/bigquery-dataset-details.png "Screenshot of the BigQuery Web UI showing the table details"
[github-functions-framework]: https://github.com/GoogleCloudPlatform/functions-framework-python
