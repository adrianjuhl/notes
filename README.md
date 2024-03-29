


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
Setup - portrait, portrait, landscape

    xrandr \
        --output DP-1-1 --mode 1920x1080 --pos 0x0      --scale 1x1 --rotate left \
        --output DP-1-2 --mode 1920x1080 --pos 1080x0   --scale 1x1 --rotate left \
        --output DP-1-3 --mode 1920x1200 --pos 2160x280 --scale 1x1 --rotate normal

    xrandr \
        --output DP-2-1 --mode 1920x1080 --pos 0x0      --scale 1x1 --rotate left \
        --output DP-2-2 --mode 1920x1080 --pos 1080x0   --scale 1x1 --rotate left \
        --output DP-2-3 --mode 1920x1200 --pos 2160x280 --scale 1x1 --rotate normal

Setup - portrait, landscape, landscape

    xrandr \
        --output DP-2-1 --mode 1920x1080 --pos 0x0      --scale 1x1 --rotate left \
        --output DP-2-2 --mode 1920x1080 --pos 1080x420 --scale 1x1 --rotate normal \
        --output DP-2-3 --mode 1920x1200 --pos 3000x300 --scale 1x1 --rotate normal

Setup - landscape, landscape, landscape

    xrandr \
        --output DP-2-1 --mode 1920x1080 --pos 0x0    --scale 1x1 --rotate normal \
        --output DP-2-2 --mode 1920x1080 --pos 3840x0 --scale 1x1 --rotate normal \
        --output DP-2-3 --mode 1920x1200 --pos 1920x0 --scale 1x1 --rotate normal --primary

Setup - landscape, landscape

    xrandr \
        --output DP-2-1 --mode 1920x1080 --pos 0x0    --scale 1x1 --rotate normal \
        --output DP-2-2 --off \
        --output DP-2-3 --mode 1920x1200 --pos 1920x0 --scale 1x1 --rotate normal --primary

Setup - inbuilt, portrait, portrait

    xrandr \
      --output eDP-1  --mode 1920x1080 --pos 0x1920 --scale 1x1 --rotate normal \
      --output DP-1-1 --mode 1920x1080 --pos 0x0    --scale 1x1 --rotate left \
      --output DP-1-2 --mode 1920x1080 --pos 1080x0 --scale 1x1 --rotate left

Setup - inbuilt, landscape, landscape

    xrandr \
      --output eDP-1  --mode 1920x1080 --pos 1920x1200 --scale 1x1 --rotate normal \
      --output DP-2-1 --mode 1920x1080 --pos 0x0       --scale 1x1 --rotate normal \
      --output DP-2-2 --off \
      --output DP-2-3 --mode 1920x1200 --pos 1920x0    --scale 1x1 --rotate normal --primary

Query for display names and settings:

    xrandr --query

Inverted laptop with normal others (display names can vary):

    xrandr --output eDP-1 --rotate inverted --output DP-1-1 --rotate normal --output DP-1-2 --rotate normal
    xrandr --output eDP-1 --rotate inverted --output DP-2-1 --rotate normal --output DP-2-2 --rotate normal

Inverted laptop with left rotated others (display names can vary):

    xrandr --output eDP-1 --rotate inverted --output DP-1-1 --rotate left --output DP-1-2 --rotate left
    xrandr --output eDP-1 --rotate inverted --output DP-2-1 --rotate left --output DP-2-2 --rotate left

All normal (restore to default) (display names can vary):

    xrandr --output eDP-1 --rotate normal --output DP-1-1 --rotate normal --output DP-1-2 --rotate normal
    xrandr --output eDP-1 --rotate normal --output DP-2-1 --rotate normal --output DP-2-2 --rotate normal


# Laptop Network Dropouts
Keep an eye on inotify watches:
lsof | grep inotify | wc -l

Current max_user_watches:
cat /proc/sys/fs/inotify/max_user_watches

echo -n "inotify count: " && lsof | grep inotify | wc -l && echo -n "max_user_watchers: " && cat /proc/sys/fs/inotify/max_user_watches

Do they correlate?
Answer (12/04/2020): No, (results were 2948 and 8192) but wifi returned after running the command below.

To increase the limit temporarily:
echo 16384 | sudo tee /proc/sys/fs/inotify/max_user_watches
(does this work?)

To track these values:
while true; do (echo && date && echo -n "inotify count: " && lsof | grep inotify | wc -l && echo -n "max_user_watchers: " && cat /proc/sys/fs/inotify/max_user_watches) | tee --append .gitignored/network_dropout_data; sleep 60; done

See also:
https://askubuntu.com/questions/828779/failed-to-add-run-systemd-ask-password-to-directory-watch-no-space-left-on-dev
https://askubuntu.com/questions/770374/user-limit-of-inotify-watches-reached-on-ubuntu-16-04
https://unix.stackexchange.com/questions/285529/internet-stops-working-failed-to-add-run-systemd-ask-password-to-directory-wa

# Outlook

## Decline a meeting but keep it displayed in calendar

1. Decline the meeting so the person knows you aren't attending
1. Go to the deleted items folder and open the invite
1. Click "Tentative" and then "Do not send a response"
1. Open the appointment and then change your time to "Free"

See https://superuser.com/a/387347

# Maven

clean recursively:
```
find . -type d -name "target" | xargs -I{} find '{}/..' -type f -maxdepth 1 -name 'pom.xml' | xargs -I{} mvn clean -f '{}'
```

# Camel

A route that runs once, and in this example to also stop the application (used for testing):
```
    from("timer://runOnce?repeatCount=1&delay=5000")
      .process(new Processor() {
        @Override
        public void process(Exchange exchange) throws Exception {
          System.exit(0);
        }
      })
    ;
```
Then build and initial run timed with:
```
$ time ./bin/run_app_local.sh
```




