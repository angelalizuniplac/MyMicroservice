## Microservice usando .net9.0

Importante ressaltar que este codigo não é desenvolvimento meu. É um codigo que se encontra facilmente na internet e vamos utilizar apenas para praticar com Docker e fazer a implentação de um serviço no azure. 

https://github.com/open-telemetry/opentelemetry-dotnet/blob/main/examples/AspNetCore/Controllers/WeatherForecastController.cs 

# Como esse projeto foi criado? 
Obs.: O projeto ja está criado, estou compartilhando como criei, se você está clonando esse projeto e não criando um novo, ignore esses passos. 

1. Criação do projeto 

```dotnet new webapi -o MyMicroservice --no-https -f net9.0```

2. Navegue até o novo diretório criado pelo comando anterior

```cd MyMicroservice```

3. Criei Controllers/WeatherForecastController.cs e adicionei um codigo aleatorio para API. 

# Docker Hub
O arquivo dockerfile já está criado e com as configurações prontas para gerar uma imagem. 

1. Gere a imagem:

```docker build -t mymicroservice .```

2. Teste criando um container: 

```docker run -it --rm -p 3000:8080 --name mymicroservicecontainer mymicroservice```

E com isso você deve conseguir acessar com sucesso "http://localhost:3000/WeatherForecast" e visualizar o retorno em JSON em seu navegador. 

3. Verificado que está funcional, faça a publicação da imagem para o seu dockerhub: 

Lembrete: Se ainda não tiver realizado o login não esqueça: ```docker login```

```docker tag mymicroservice angeladlizuniplac/mymicroservice```

```docker push angeladlizuniplac/mymicroservice```

Para executar o container do docker hub: 
```docker run -d -p 3000:8080 --name mymicroservice angeladlizuniplac/mymicroservice```

Para validar que deu certo acesse: http://localhost:3000/WeatherForecast


# AZURE

Siga o passo a passo do tutorial: https://dotnet.microsoft.com/en-us/learn/aspnet/deploy-microservice-tutorial/intro

Obs.: no tutorial o arquivo deploy.yaml irá indicar criar na porta 80, se você geralmente tem problemas com essa porta assim como eu, recomendo utilizar outra. 

Após concluir o tutorial da Azure você terá seu primeiro serviço implantado. 

Isso está no tutorial, mas vou trazer para cá como apoio: 

O comando: ```kubectl get services``` irá listar serviços e será possível visualizar o EXTERNAL-IP.
Com esse EXTERNAL-IP é possível acessar o seu serviço, exemplo: http://20.245.254.57:8080/WeatherForecast

---------------------------------------------------------------------------------------------------------------------------------------------------------

Comando depreciado microsoft: 
az aks create --resource-group MyMicroserviceResources --name MyMicroserviceCluster --node-count 1 --enable-addons http_application_routing --generate-ssh-keys

comando atualizado: 
az aks create --resource-group MyMicroserviceResources --name MyMicroserviceCluster --node-count 1 --enable-addons web_application_routing --generate-ssh-keys
-----------------------------------------------------------------------------------------------------------------------------------------------------

# Caso de erro de arquitetura

docker manifest inspect angeladlizuniplac/mymicroservice:latest

1.Verifique se o buildx está habilitado
```docker buildx create --use```

2. Construa e envie a imagem com suporte multi-plataforma
```docker buildx build --platform linux/amd64,linux/arm64 -t angeladlizuniplac/mymicroservice:latest --push .```

Ps: Esse comando gerou o erro no video, pesquisando rapidamente pode ser estouro de memoria, apenas tentei executar novamente e passou. 
Achei o comando abaixo que limita os recusos caso você ainda tenha erro: 
 ```docker buildx build --platform linux/amd64,linux/arm64 -t angeladlizuniplac/mymicroservice:latest --push . --builder-opt network=host --builder-opt env.BUILDKIT_STEP_LOG_MAX_SIZE=10485760``` (Comando não testado)

3. Delete o deployment atual
```kubectl delete deployment mymicroservice --ignore-not-found=true```

4. Faça o deploy novamente 
```kubectl apply -f deploy.yaml```

## Comandos uteis
Verificar status dos pods:
```kubectl get pods -o wide```

listar serviços com seus respectivos ips:
```kubectl get services```