## Set up federation:
1. On **RSH** Create index called index2

2. Create service account called ```fsuser``` with password ```fsuserfsuser```. Uncheck change password.

3. Populate som data with:

```
| makeresults count=3600
| streamstats count as int
| eval gauge_foobar=random() % 10 + 1
| eval _time=_time-int
| eval host="rsh"
| fields gauge_foobar host _time
|collect index=index2 source=sample_rsh
```

4. On **FSH**, set up federated search provider:

Settings, Federated search
```
rsh
172.16.239.11:8089
fsuser
fsuserfsuser
```

5. set up federated index:

```
index2
index2
```
6. Verify the federation

```
index=federated:index2
```
7. Show correlation, create an index called index1 on **FSH**

8. Populate it with data:
```
| makeresults count=3600
| streamstats count as int
| eval gauge_foobar=random() % 10 + 1
| eval _time=_time-int
| eval host="fsh"
| fields gauge_foobar host _time
|collect index=index1 source=sample_fsh
```

9. Run a timechart:
```
| union
  [ search index = federated:index2  ]
  [ search index = index1]
  |timechart avg(gauge_foobar) as gauge by host```
