While fixing an issue with the github actions workflow, the pipeline job ran for couple of times (100s!)
before I could realise (and fix the issue) 
that the actions cache(https://github.com/actions/cache) was generating unique key for each run and it was creating multiple keys on each run
IT WAS TOO LATE !

# What's Actions Cache?  

Basically is a github actions feature that allows to create a cache from outputs or downloaded 
dependencies and reuse in subsequent runs to make build faster. It works by creating cache
with a a unique key, and key gets computed in each run, and key has to be exactly same for cache to be used[cache hit], or it will create new cache running that step[cache miss]

See https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/caching-dependencies-to-speed-up-workflows

# Problem

Github page to cleanup was very friendly for one who has all the time in the world to go oneby one and "delete" going past the confirmation alert each time, thats fun right?! WRONG!
It Could Have Provided a Multiselect and delete all like any SaneUI may offer

# Automate

Luckily  (gh+sh) commands finds its way

This goes over caches one by one and delete them
```
for cache_id in $(gh api /repos/{owner}/{repo}/actions/caches | jq -r '.actions_caches[].id'); do
  echo deleting $cache_id
  gh api --method DELETE /repos/{owner}/{repo}/actions/caches/$cache_id --silent
done
```

Below command helps to delete all those failed job

```
gh run list --status failure --json databaseId | jq -r '.[].databaseId' | while read id; do
    gh run delete "$id"
    #slow down (rate limit)
    sleep 1 
done
```
