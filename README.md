# poc-graviteeio


# dev

```bash
export COMMIT_MESSAGE=" feature (quickstart) : "
export COMMIT_MESSAGE="$COMMIT_MESSAGE provision la plus simple"


URI_GIT=git@gitlab.com:bureau1/pulumi-workshops/poc-api-gateway/poc-graviteeio.git
git clone $URI_GIT ~/.poc.graviteeio
cd ~/.poc.graviteeio


git add --all && git commit -m "$COMMIT_MESSAGE" && git push -u origin master

atom .

```
