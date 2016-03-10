## Check if the Elasticsearch is up and running.
GET /

## Create an index into Elasticsearch.
PUT /phpbg
{
  "number_of_shards": 1,
  "number_of_replicas": 0
}

## Get index settings.
GET /phpbg/_settings
GET /_settings

## Cannot change the number of shards after the
## index is created.
PUT /phpbg/_settings
{
  "number_of_replicas": 0
}

## Delete an index.
DELETE /phpbg

## Get cluster healt and state
GET /_cluster/health
GET /_cluster/state

# Using Cat API.
GET /_cat/indices
## Cat all the installed plugins.
GET /_cat/plugins


#########################################################
##               CRUD DOCUMENT

## Create a document.
## Specify the /[INDEX]/[DOCUMENT TYPE]/[ID]
PUT /phpbg/news/1
{
  "title": "A news title",
  "author": {
    "firstname": "Nikolay",
    "lastname":  "Ignatov"
  },
  "tags": [
    "it",
    "elasticsearch"
  ],
  "commentcount": 100,
  "rating": 4.2,
  "publish_date": "2016-02-29T20:00:00-0200"
}

## Check the mapping (schema).
GET /phpbg/news/_mapping


# Delete update the mapping.
DELETE /phpbg

# Update the schema.
PUT /phpbg/_mapping/news
{
  "properties": {
    "author": {
      "properties": {
        "firstname": {
          "type": "string"
        },
        "lastname": {
          "type": "string"
        }
      }
    },
    "commentcount": {
      "type": "long"
    },
    "publish_date": {
      "type": "date",
      "format": "strict_date_optional_time||epoch_millis"
    },
    "rating": {
      "type": "float"
    },
    "tags": {
      "type": "string"
    },
    "title": {
      "type": "string"
    }
  }
}

# Get the schema of the types in the cluster.
GET /phpbg/_mapping



## Retrive the document.
GET /phpbg/news/1

## Retrive a non existing document.
GET /phpbg/news/nonexists

## Put other document
PUT /phpbg/news/2
{
  "title": "News about PHP UG meetup",
  "author": {
    "firstname": "Shay",
    "lastname":  "Banon"
  },
  "publish_date": "2016-03-01T20:00:00-0200"
}

## Retrive the second document.
GET /phpbg/news/2

## 1. Override it - PUT it again. COMPLETELY OVERRIDE!
## 2. Partial update via UPDATE API - POST
POST /phpbg/news/2/_update
{
  "doc": {
    "commentcount": 20,
    "facebook_comments": 20
  }
}

## Retrive the update document.
GET /phpbg/news/2

## You can see how the new fields are added to the mapping.
GET /phpbg/news/_mapping

## Using the version of the document.
GET /phpbg/news/2
PUT /phpbg/news/2?version=5
{
  "title": "News about PHP UG 2016",
  "author": {
    "firstname": "Shay",
    "lastname":  "Banon"
  },
  "publish_date": "2016-03-01T20:00:00-0200"
}

## Delete document
DELETE /phpbg/news/2

########################################################
##               BULK API
POST /phpbg/news/_bulk
{"index":{"_id": 3}}
{"title": "PHP UG 2016 BULK API","author":{"firstname": "Shay","lastname":  "Banon"},"publish_date": "2016-03-02T20:00:00-0200"}
{"index":{"_id": 4}}
{"title": "Title 4 testing","author":{"firstname": "Nikolay","lastname":  "Ignatov"},"publish_date": "2016-03-01T18:00:00-0200","commentcount": 99}
{"index":{"_id": 5}}
{"title": "Title 5 testing","author":{"firstname": "Nikolay","lastname":  "Banon"},"publish_date": "2016-03-01T13:00:00-0200"}
{"delete":{"_id": 2}}

#########################################################

# SEARCH API
GET /phpbg/news/_search

## Search using the DSL
POST /phpbg/news/_search
{
  "query": {
    "match": {
      "author.firstname": "nikolay"
    }
  }
}

## Aggregation search.
POST /phpbg/news/_search
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "name_of_aggs": {
      "terms": {
        "field": "author.firstname",
        "size": 10
      },
      "aggs": {
        "comment_sum": {
          "sum": {
            "field": "commentcount"
          }
        }
      }
    }
  }
}

## The power of DSL
GET /phpbg/news/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {
          "author.firstname": "nikolay"
        }}
      ],
      "should": [
        {"match": {
          "title": {
            "query": "news"
          }
        }},
        {"match": {
          "author.lastname": "ignatov"
        }}
      ]
    }
  },
  "highlight": {
    "fields": {
      "title": {},
      "author.lastname": {}
    }
  }
}

## Text analysis phase = Tokenization + Normalization.
## Tokenization is done by one tokenized and normalization
## is made by 0 or more token filters like lowercase and etc.
GET /_analyze
{
  "analyzer" : "standard",
  "explain" : true,
  "text" : "PHP UG meetup has been great!"
}
