ElasticSearch性能测试esrally

# 一、简介

官方文档：https://esrally.readthedocs.io/en/stable/

GitHub：https://github.com/elastic/rally



# 二、安装配置esrally

## 1、安装esrally

```bash
pip3 install esrally

brew install pbzip2
```

## 2、配置



# 三、命令详解

## 1、命令格式

```bash
esrally [-h] [--version] {race,list,info,create-track,generate,compare,download,install,start,stop} 

可选参数:
  -h, --help            show this help message and exit
  --version             show program's version number and exit

子命令:
  {race,list,info,create-track,generate,compare,download,install,start,stop}
    race                Run a benchmark
    list                List configuration options
    info                Show info about a track
    create-track        Create a Rally track from existing data
    generate            Generate artifacts
    compare             Compare two races
    download            Downloads an artifact
    install             Installs an Elasticsearch node locally
    start               Starts an Elasticsearch node locally
    stop                Stops an Elasticsearch node locally

Find out more about Rally at https://esrally.readthedocs.io/en/2.2.0/
```

## 2、子命令

### ①列出内置的测试数据

```bash
$ esrally list tracks
```

| 测试数据      | 测试数据描述                                                 | 文档个数    | 压缩后大小 | 未压缩大小 | Default Challenge       | All Challenges                                               |
| ------------- | ------------------------------------------------------------ | ----------- | ---------- | ---------- | ----------------------- | ------------------------------------------------------------ |
| geonames      | POIs from Geonames                                           | 11,396,503  | 252.9 MB   | 3.3 GB     | append-no-conflicts     | append-no-conflicts,append-no-conflicts-index-only,append-sorted-no-conflicts,append-fast-with-conflicts,significant-text |
| percolator    | Percolator benchmark based on AOL queries                    | 2,000,000   | 121.1 kB   | 104.9 MB   | append-no-conflicts     | append-no-conflicts                                          |
| http_logs     | HTTP server log data                                         | 247,249,096 | 1.2 GB     | 31.1 GB    | append-no-conflicts     | append-no-conflicts,runtime-fields,append-no-conflicts-index-only,append-sorted-no-conflicts,append-index-only-with-ingest-pipeline,update,append-no-conflicts-index-reindex-only |
| geoshape      | Shapes from PlanetOSM                                        | 60,523,283  | 13.4 GB    | 45.4 GB    | append-no-conflicts     | append-no-conflicts                                          |
| metricbeat    | Metricbeat data                                              | 1,079,600   | 87.7 MB    | 1.2 GB     | append-no-conflicts     | append-no-conflicts                                          |
| geopoint      | Point coordinates from PlanetOSM                             | 60,844,404  | 482.1 MB   | 2.3 GB     | append-no-conflicts     | append-no-conflicts,append-no-conflicts-index-only,append-fast-with-conflicts |
| nyc_taxis     | Taxi rides in New York in 2015                               | 165,346,692 | 4.5 GB     | 74.3 GB    | append-no-conflicts     | append-no-conflicts,append-no-conflicts-index-only,append-sorted-no-conflicts-index-only,update,append-ml,date-histogram |
| geopointshape | Point coordinates from PlanetOSM indexed as geoshapes        | 60,844,404  | 470.8 MB   | 2.6 GB     | append-no-conflicts     | append-no-conflicts,append-no-conflicts-index-only,append-fast-with-conflicts |
| so            | Indexing benchmark using up to questions and answers from StackOverflow | 36,062,278  | 8.9 GB     | 33.1 GB    | append-no-conflicts     | append-no-conflicts                                          |
| eventdata     | This benchmark indexes HTTP access logs generated based sample logs from the elastic.co website using the generator available in https://github.com/elastic/rally-eventdata-track | 20,000,000  | 756.0 MB   | 15.3 GB    | append-no-conflicts     | append-no-conflicts,transform                                |
| eql           | EQL benchmarks based on endgame index of SIEM demo cluster   | 60,782,211  | 4.5 GB     | 109.2 GB   | default                 | default                                                      |
| nested        | StackOverflow Q&A stored as nested docs                      | 11,203,029  | 663.3 MB   | 3.4 GB     | nested-search-challenge | nested-search-challenge,index-only                           |
| noaa          | Global daily weather measurements from NOAA                  | 33,659,481  | 949.4 MB   | 9.0 GB     | append-no-conflicts     | append-no-conflicts,append-no-conflicts-index-only,top_metrics,aggs |
| pmc           | Full text benchmark with academic papers from PMC            | 574,199     | 5.5 GB     | 21.7 GB    | append-no-conflicts     | append-no-conflicts,append-no-conflicts-index-only,append-sorted-no-conflicts,append-fast-with-conflicts |

### ②创建测试实例

```bash
esrally create-track \
	--track=http_logs \
	--target-hosts=127.0.0.1:9200 \
	--client-options="timeout:60,basic_auth_user:'elastic',basic_auth_password:'*****'" \
	--indices="products,companies" \
	--output-path=~/tracks
```

- `track.json` contains the actual Rally track. For details see the [track reference](https://esrally.readthedocs.io/en/stable/track.html).
- `companies.json` and `products.json` contain the mapping and settings for the extracted indices.
- `*-documents.json(.bz2)` contains the sources of all the documents from the extracted indices. The files suffixed with `-1k` contain a smaller version of the document corpus to support [test mode](https://esrally.readthedocs.io/en/stable/adding_tracks.html#add-track-test-mode).

### ③

### ④

### ⑤

### ⑥

### ⑦

### ⑧

### ⑨





# 三、性能测试

## 1、安装esrally

```bash
pip3 install esrally
```

## 2、创建测试任务和数据

```bash
esrally create-track \
	--track=http_logs \
	--target-hosts=127.0.0.1:9200 \
	--client-options="timeout:60,basic_auth_user:'elastic',basic_auth_password:'*****" \
	--indices="products,companies" \
	--output-path=~/tracks
```

- 

## 3、

```bash
esrally race --distribution-version=6.0.0 --track=geopoint --challenge=append-fast-with-conflicts
```



# 三、



```bash
esrally list tracks
esrally list races

esrally create-track \
	--track=http_logs \
	--target-hosts=127.0.0.1:9200 \
	--client-options="timeout:60,basic_auth_user:'elastic',basic_auth_password:'*****'" \
	--indices="products,companies" \
	--output-path=~/tracks
	
esrally race \
--target-hosts=127.0.0.1:9200 \
--client-options="timeout:60,basic_auth_user:'elastic',basic_auth_password:'*****'" \
--track=geonames
```

