# Docker:
## Instalação do Engine:
  - Acesse https://docs.docker.com/desktop/ para conferir o procedimento de instalação para seu sistema operacional
    - Caso esteja usando o Linux, não esquecer do procedimento posterior a instalação, para isso acesse ver https://docs.docker.com/engine/install/linux-postinstall/
  - Para confirmar que o docker esta instalado, use o comando __docker --version__ no prompt
  - No diretório deste repositório, crie um arquivo __.env__ e preencha com as seguintes variáveis de ambiente (em caso de dúvidas, confira o arquivo .env de exemplo dentro deste repositório):
    - MYSQL_ROOT_PASSWORD
    - MYSQL_DATABASE
    - MYSQL_USER
    - MYSQL_PASSWORD
## Instalação do Conteiner:
  - Execute o seguinte comando no prompt __docker-compose up__
    - Caso seja necessário forçar a reinstalação do container, desconsiderando cache e executando novamente todo procedimento do dockerfile e dockercompose, use o comando __docker compose up --build -d__
## Acessando Aplicação:
  - http://localhost:3000/
## Criando imagem no DockerHub:
  ### Preparar imagem local: 
  - __docker build -t [USUARIO DO DOCKER HUB]/infnet-guia-app:latest .__
  - Exemplo: __docker build -t jclaudiodsj/infnet-guia-app:latest .__
  ### Fazer upload para repositório do DockerHub: 
  - __docker push [REPOSITORIO DO DOCKER HUB]:latest__
  - Exemplo: __docker push jclaudiodsj/infnet-guia-app:latest__
  - OBS: Se não foi feito login anteriormente, irá precisar através do comando __docker login__
## Download da imagem do DockerHub:
  ### Dowload da imagem:
  - __docker pull [REPOSITORIO DO DOCKER HUB]__
  - Exemplo: __docker pull jclaudiodsj/infnet-guia-app__
  ### Iniciar o container com a imagem:
  - __docker run -d -p 3000:3000 <ID_DA_IMAGEM>__
  - OBS: Caso não saiba o id da imagem, basta usar o comando __docker images__
# Kubernetes (k8s):
## Instalação do Minikube:
  - Acesse https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fbinary+download para conferir o procedimento de instalação para seu sistema operacional
## Iniciar Minikube:
  - __minikube start –driver=docker__
## Instalação dos pods e services com os arquivos de deployment:
  - __kubectl apply -f [ARQUIVO DEPLOYMENT].yaml__
  - Exemplo: 
    - __kubectl apply -f k8s/app/deployment.yaml__
    - __kubectl apply -f k8s/monitoring/grafana/grafana.yaml__
    - __kubectl apply -f k8s/monitoring/node-export/node-exporter.yaml__
    - __kubectl apply -f k8s/monitoring/prometheus/prometheus.yaml__

## Acessar a aplicação:
  - Executar no prompt o comando __minikube service service-infnet-guia-app__ ou __minikube service --url service-infnet-guia-app__ para descobrir o endereço para acesso a aplicação.

## Acessar o Grafana:
  - Executar no prompt o comando __minikube service grafana__ ou __minikube service --url grafana__ para descobrir o endereço para acesso a aplicação.

## Verificar a quantidade de pods em execução:
  - __kubectl get pods__
