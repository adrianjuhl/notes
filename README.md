


Clone all the repos in a given group from gitlab:

```
curl -s --header "PRIVATE-TOKEN:<token>" "https://gitlab.adelaide.edu.au/api/v4/groups/integration/projects?per_page=100&page=1" | json_pp | grep ssh_url_to_repo | cut -d \" -f 4 | xargs -L1 git clone
```


Search for a particular string in all the git repos under the current directory:

```
find . -name .git -type d -prune -exec bash -c "cd '{}/..' && pwd && git grep search_string \$(git rev-list --all)" \;
```

Find image tag references:
```
oc describe all --namespace=namespaceName | grep Image: | grep "docker-registry" | awk '{print $2}' | sort | uniq -c
OR
oc describe all --selector=app=appName --namespace=namespaceName | grep Image: | grep "docker-registry" | awk '{print $2}' | sort | uniq -c
```

Delete old openshift istag values:

```
oc get istag --selector=app==app-name
oc get istag | grep imagestream_name
oc get istag --selector=app==app-name | cut -f 1 -d ' ' | sort
oc get istag --selector=app==app-name | cut -f 1 -d ' ' | sort | grep tag_search_string
oc get istag --selector=app==app-name | cut -f 1 -d ' ' | sort | grep tag_search_string | xargs -i oc delete istag/{}

When the imagestream doesn't have a label that can be used for the selector, use:
oc get istag | grep 'app-name:' | cut -f 1 -d ' ' | sort | grep tag_search_string | xargs -i oc delete istag/{}
```


# Monitor orientations
Inverted laptop with normal others:

    xrandr --output eDP-1 --rotate inverted --output DP-2-1 --rotate normal --output DP-2-2 --rotate normal

All normal (restore to default):

    xrandr --output eDP-1 --rotate normal --output DP-2-1 --rotate normal --output DP-2-2 --rotate normal

