# How to use ElasticSearch and Kibana to visualise data from IMC
This README acts as an example of how to use ES+Kibana to import data and create visualisations and dashboards than can be shared or imported on existing web sites

## Intro: Use ImproveMyCity existing API

ImproveMyCity already has a complete API which can be used to export various data

Calling the `/GET/issues` method returns JSON in the following form 

```
    {
      "id": 30,
      "title": "with pdf",
      "alias": "",
      "stepid": 1,
      "catid": 11,
      "regnum": "",
      "regdate": "0000-00-00 00:00:00",
      "responsible": "",
      "description": "adadfasfa",
      "address": "Via S. Massimo, 26, 10123 Torino TO, Italia",
      "latitude": "45.062851074100740",
      "longitude": "7.687738896337919",
      "state": 1,
      "moderation": false,
      "created": "2018-04-30 03:23:37",
      "updated": "2018-04-30 11:41:12",
      "created_by": 913,
      "hits": 0,
      "extra": "",
      "votes": 0,
      "subgroup": 0,
      "catid_title": "Parking",
      "stepid_title": "Submitted",
      "stepid_color": "#e66317",
      "created_by_name": "Ioannis Tsampoulatidis",
      "category_image": "",
      "comments": 0,
      "photos": [
        {
          "name": "favicon.png",
          "size": 22809,
          "url": "http://160.40.51.94/images/imc/30/favicon.png",
          "mediumUrl": "http://160.40.51.94/images/imc/30/medium/favicon.png",
          "thumbnailUrl": "http://160.40.51.94/images/imc/30/thumbnail/favicon.png"
        },
        {
          "name": "torino-586379_1920.jpg",
          "size": 889638,
          "url": "http://160.40.51.94/images/imc/30/torino-586379_1920.jpg",
          "mediumUrl": "http://160.40.51.94/images/imc/30/medium/torino-586379_1920.jpg",
          "thumbnailUrl": "http://160.40.51.94/images/imc/30/thumbnail/torino-586379_1920.jpg"
        }
      ],
      "attachments": [
        {
          "name": "boardingPass02.pdf",
          "size": 66812,
          "url": "http://160.40.51.94/images/imc/30/boardingPass02.pdf"
        }
      ],
      "created_TZ": "2018-04-30 03:23:37",
      "updated_TZ": "2018-04-30 11:41:12",
      "regdate_TZ": "0000-00-00 00:00:00",
      "created_ts": 1525058617,
      "updated_ts": 1525088472,
      "myIssue": false,
      "hasVoted": false
    }
```

## STEP 1: Call exportES API method of IMC
The above JSON format is not ready to be used by ElasticSearch and that's why, for CUTLER, we introduced two methods to export directly into ES-compatible format

### 1) Using the new integrated IMC API method "exportES"

```/GET/exportES```

### 2) Using the middleware "imc2es" transformer PHP script

```php imc2es issues.json imc.json```


The exported file by using either method is ready to be indexed in ElasticSearch. The first three records of `imc.json` follows:

```
{"index":{"_id":1}}

{"id":1,"title":"Broken traffic light","stepid":2,"catid":10,"description":"Segnalo la presenza da diversi giorni di rifiuti abbandonati lungo la strada. Segnalo la presenza da diversi giorni di rifiuti abbandonati lungo la strada. Segnalo la presenza da diversi giorni di rifiuti abbandonati lungo la strada.","address":"Via Pietro Giannone, 3-5, 10121 Torino TO, Italia","latitude":45.06994829722,"longitude":7.675870171875,"state":1,"moderation":0,"created":"2018-04-04 02:15:25","updated":"2018-04-24 00:10:45","created_by":912,"updated_by":0,"votes":0,"modality":0,"catid_title":"Municipal Police","stepid_title":"Acknowledged","comments":0,"location":"45.06994829722, 7.675870171875"}

{"index":{"_id":2}}

{"id":2,"title":"Huge pothole","stepid":3,"catid":10,"description":"A huge pothole ","address":"Via Giovanni Giolitti, 42, 10123 Torino TO, Italia","latitude":45.064189361453,"longitude":7.6900322354736,"state":1,"moderation":0,"created":"2018-04-04 05:53:12","updated":"2018-04-14 21:31:43","created_by":912,"updated_by":0,"votes":1,"modality":0,"catid_title":"Municipal Police","stepid_title":"On progress","comments":0,"location":"45.064189361453, 7.6900322354736"}

{"index":{"_id":3}}

{"id":3,"title":"Graffiti opposite of the school","stepid":1,"catid":10,"description":"A huge graffiti","address":"Via XX Settembre, 14, 10121 Torino TO, Italia","latitude":45.065219950477,"longitude":7.679303399414,"state":1,"moderation":0,"created":"2018-04-04 05:53:57","updated":"2018-04-04 05:53:57","created_by":912,"updated_by":0,"votes":1,"modality":0,"catid_title":"Municipal Police","stepid_title":"Submitted","comments":0,"location":"45.065219950477, 7.679303399414"}

```

**Note** the transformed JSON contains no personal data (at least no first and last names)

You can find the latest dataset in https://mklab.iti.gr/cutler/doku.php?id=wp8#data_sets


## STEP 2: Create the appropriate mappings if necessary

In order to import data to ElasticSearch, some mappings are necessary

#### for example: 
- "location" is of type "geo_point"
- "created" is of type "date"

We can even do some more advanced things such as tokenizing text (e.g. title or descriiption) in keywords

Even more advanced we can define to tokenize based on a `stop token filter` [more on official documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-stop-tokenfilter.html) for local languages (including greek and turkish)

In order to create your mappings go to your instance of Kibana

 `Kibana->Dev Tools -> Console` and send the following request:

```
PUT /imc
{
  "settings":{
    "analysis":{
        "analyzer":{
          "analyzer_keyword":{
              "tokenizer":"standard",
              "filter":"my_stop"
          }
        },
        "filter": {
            "my_stop": {
                "type": "stop",
                "ignore_case": true,
                "stopwords": "_greek_"
            }
        }
    }
  }, 
  "mappings": {
    "doc": {
      "properties": {
        "location": {
          "type": "geo_point"
        },
        "created": {
          "type": "date",
          "format": "yyyy-MM-dd HH:mm:ss"
        },
        "title": { 
          "type": "text", 
          "analyzer": "analyzer_keyword",
          "fielddata": true,  
          "fields": {
            "raw": {
              "type": "keyword"
            }
          }
        },
        "description": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword"
            }
          }
        }
      }
    }
  }
}
```

you will receive

```
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "imc"
}
```
which means that a new index called "imc" with its settings and mappings has been created (still empty though...)

## STEP 3: Import imc.json

The next step is to import the actual ES-compatible JSON data

You post the exported `imc.json` file to the ElasticSearch server by executing the following

`curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/imc/doc/_bulk?pretty' --data-binary @imc.json`

## STEP 4: Discover & create index

Go to your instance of Kibana

`Management -> Index pattern`

- type `imc*`
- Next Step -> Create Index Pattern

Set the `created` field for time filtering

If everything works as expected, Kibana will display a list of all available fields... pay extra attention to "location" which is of type "geo_point" (according to the mapping)

## STEP 5: Visualise

Go to your instance of Kibana

`Visualise -> Create a visualisation`

Select a visualisation type
e.g. Coordinate Map

Select index == imc*


# What's next

- Create more visualisations and then combine them under one (or more) dashboards
- Create reports and export data in CSV format for furher analysis
- Create more advanced filters 
- Import other datasets beyond IMC and combine data from different sources to create useful insights
