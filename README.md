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


# Gravitee start



### Installation

* https://docs.gravitee.io/apim/1.x/apim_installguide_docker_compose.html
* version : https://github.com/gravitee-io/gravitee-gateway/releases/tag/1.30.8


* run :

```bash

URI_GIT=git@gitlab.com:bureau1/pulumi-workshops/poc-api-gateway/poc-graviteeio.git
git clone $URI_GIT ~/.poc.graviteeio
cd ~/.poc.graviteeio



docker-compose pull

docker-compose up -d

docker-compose logs -f


```

* upgrade/downgrade to version :

```bash
export GRAVITEE_VERSION=1.30.8
export DWNLOAD_URI=https://raw.githubusercontent.com/gravitee-io/gravitee-docker/master/apim/${GRAVITEE_VERSION}/docker-compose.yml
curl -L https://raw.githubusercontent.com/gravitee-io/gravitee-docker/master/apim/1.x/docker-compose.yml -o "docker-compose.yml"
```


<table class="tableblock frame-all grid-all spread">
<colgroup>
<col style="width: 33.3333%;">
<col style="width: 33.3333%;">
<col style="width: 33.3334%;">
</colgroup>
<thead>
<tr>
<th class="tableblock halign-left valign-top">Component</th>
<th class="tableblock halign-left valign-top">URL</th>
<th class="tableblock halign-left valign-top">Expected</th>
</tr>
</thead>
<tbody>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock">API Gateway</p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><a href="http://localhost:8082" class="bare">http://localhost:8082</a></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock">Must return 404 status code, with a <code>No context-path matches the request URI</code> payload.</p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock">Management API</p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><a href="http://localhost:8083/management/apis" class="bare">http://localhost:8083/management/apis</a></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock">Must return a 200 status code, with a <code>[]</code> payload.</p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock">Management UI</p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><a href="http://localhost:8084" class="bare">http://localhost:8084</a></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock">Must return a 200 status code, with the Gravitee.io Portal</p></td>
</tr>
</tbody>
</table>
