# Connection / Reconnection / Throughput Test

## Connection Rate without and with NGiNX

Direct connection means test tool connected directly to Solace Broker

NGiNX connection means keyval table populated with each additional connection

Requirement would be to get all 200K connections connected/reconnected within a time frame defined as a service outage window, usually 5 minutes for Internet connected applications.  This means at a rate higher then 700 connections/reconnections a minute

| # Connections | Offered Rate | Direct Actual Rate | NGiNX Actual Rate |
| ------------- |:-------------:| -----:| ----:|
| 30000 | 3000 | 2300 | 2300 |
| 60000 | 3000 | 2300 | 2200 |
| 100000 | 2x1100 | 2000 | 2000 |
| 200000 | 4x550 | 1600 | 1400 |

Sample test tool cli: 

sudo ./connectChurn --cip=mqtt:172.31.8.41 --contexts=16 --sessions=60000 --start=3000 --mr=200 --s=0 --cleansession=1 --ka=50 --logintimeout=120

## Re-Connection Rate without and with NGiNX

Disconnect was a forced disconnect from the broker side

| # Connections | Offered Rate | Direct Actual Rate  | NGiNX Actual Rate |
| ------------- |:-------------:| -----:| ----:|
| 100000 | 2x1100 | 1000 | 980 |
| 200000 | 4x550 | 1000 | 940 |

Note total connections hovered around 100/200K so reconnect rate might have been limited to actual disconnect rate

## Message throughput performance test

Test is to find aproximate throughput delta between direct and via NGiNX connection. 

Requirement would to be able to forward a single core broker persistent rate which is currently: 300K to 850K msg/sec @ 1K msg size

| Msg Size | Offered Rate | Direct Actual Rate  | NGiNX Actual Rate |
| ------------- |:-------------:| -----:| ----:|
| 1024 | 1000000 |  889000 | 607000 |

Sample test tool cli:

./sdkperf_mqtt_dl -cip=172.31.8.41 -cc=5 -ptl=test/topic1,test/topic2,test/topic3,test/topic4,test/topic5  -stl=test/topic1,test/topic2,test/topic3,test/topic4,test/topic5 -msa=1024 -mr=1000000 -mn=100000000