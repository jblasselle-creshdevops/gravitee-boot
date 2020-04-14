# poc-graviteeio




# Gravitee start



## Installation

### Methode 1 : one comamnd quick start


```bash
curl -L http://bit.ly/graviteeio-am | bash -s 8080
```

En partant de `master`, celui-là aussi je pense, présente le même bug que la méthode suivante


### Methode 2 : Une installation testée qui donne un crash gravitee.io (buggé pas d'ouverture d'issue github pour l'instant)

* https://docs.gravitee.io/apim/1.x/apim_installguide_docker_compose.html
* https://docs.gravitee.io/am/2.x/am_installguide_docker.html#docker_compose
* version : https://github.com/gravitee-io/gravitee-gateway/releases/tag/1.30.8


* run  :

```bash

URI_GIT=git@gitlab.com:bureau1/pulumi-workshops/poc-api-gateway/poc-graviteeio.git
git clone $URI_GIT ~/.poc.graviteeio
cd ~/.poc.graviteeio

# pour elastic search
sudo sysctl -w vm.max_map_count=262144
#

# (Optional step: pull to ensure that you are running latest images)
docker-compose pull

# And run...
docker-compose up -d && docker-compose logs -f


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

# Préparattion Kong Au cas où

https://dev.to/_mertsimsek/kong-microservices-api-gateway-with-docker-12g5


### Recette rapide Kong


* ccc :

```bash
# sur l'hôte de test local, disons qu'il ait le nom d'hôte réseau [pegasusio.io]
git clone git@gitlab.com:bureau1/pulumi-workshops/poc-api-gateway/basic-api-kong.git
cd basic-api-kong

docker run -d --name king-kong -p 8000:8000 -p 8443:8443 -p 8001:8001 -p 8444:8444 cresh/king-kong:latest

```


source info pour améliorer cette recette : https://docs.konghq.com/install/docker/?_ga=2.253834817.893832238.1586863912-1018594552.1586863912&_gac=1.182739474.1586863939.EAIaIQobChMI143Q6ejn6AIVybvVCh1RfAJ9EAAYASAAEgJu2vD_BwE


```bash

$ curl -i http://pegasusio.io:8000
HTTP/1.1 404 Not Found
Date: Tue, 14 Apr 2020 11:55:11 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Content-Length: 48
X-Kong-Response-Latency: 1
Server: kong/2.0.3

{"message":"no Route matched with those values"}
```


### Suite : https://docs.konghq.com/2.0.x/getting-started/quickstart/


(Learn Kong)



et le but final ce serait ça : (case créer  un API consumer pour ensuite derrière voir si on arrive à récupéere l'ID des  API consumers dans notre REST API )


https://docs.konghq.com/2.0.x/getting-started/adding-consumers/



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
