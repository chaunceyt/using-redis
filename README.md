# Using Redis

Forever I've used Redis as a caching layer for web applications.

## Other use cases.

### Querying, Indexing, and Full-Text Search

Create a Legislators search engine using RedisSearch

Requirements

- [Redis Input/Output Tools](https://github.com/redis-developer/riot): roit-file 
- data: legislators-current.csv

```
mkdir redis-search
# Run redis with search module enabled.
docker run -p 6379:6379 --name redis-search -d redislabs/redisearch:latest

# Download redis input/output tool
wget https://github.com/redis-developer/riot/releases/download/v2.9.0/riot-file-2.9.0.zip
unzip riot-file-2.9.0.zip
PATH=$PATH:$(PWD)/riot-file-2.9.0/bin

# Download legislator data
wget https://theunitedstates.io/congress-legislators/legislators-current.csv

# Import csv into Redis
riot-file import legislators-current.csv --header hset --keyspace us:legislator --keys bioguide_id

```

### Confirm you have expected data

```
docker exec -it redis-search redis-cli

127.0.0.1:6379> scan 0 count 10
1) "288"
2)  1) "us:legislator:A000372"
    2) "us:legislator:F000468"
    3) "us:legislator:C001078"
    4) "us:legislator:S001187"
    5) "us:legislator:C001110"
    6) "us:legislator:L000585"
    7) "us:legislator:K000396"
    8) "us:legislator:C001066"
    9) "us:legislator:B001301"
   10) "us:legislator:L000273"
```   

### Create search index

```
docker exec -it redis-search redis-cli

# Create an index
127.0.0.1:6379> FT.CREATE legislators-idx ON HASH PREFIX 1 us:legislator: SCHEMA bioguide_id TAG SORTABLE fullname_name TEXT SORTABLE first_name TEXT SORTABLE  last_name TEXT SORTABLE state TAG SORTABLE party TAG SORTABLE type TAG SORTABLE

# Text searches
FT.SEARCH legislators-idx "Charles"
FT.SEARCH legislators-idx "Jo*"

# Search based on a tag(s)
FT.SEARCH legislators-idx "@bioguide_id:{S000148}"
FT.SEARCH legislators-idx "@state:{TX} @type:{sen} " RETURN 1 full_name
FT.SEARCH legislators-idx "@state:{MO}" RETURN 2 full_name party
 
```