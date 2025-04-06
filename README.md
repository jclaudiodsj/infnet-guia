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
## Anexos:
### Arquivo DOCKERFILE:
Este arquivo Dockerfile está criando uma imagem Docker para uma aplicação Node.js com pnpm como o gerenciador de pacotes. O arquivo é estruturado em várias etapas (conhecidas como build stages), cada uma com um propósito específico, para otimizar a criação da imagem e fornecer diferentes configurações para desenvolvimento e produção.
#### FROM node:18-alpine AS base
  - FROM: Este comando define a imagem base para o Docker. No caso, a imagem node:18-alpine é uma versão leve do Node.js baseada no Alpine Linux, que é otimizada para conter um número reduzido de pacotes.
  - AS base: A palavra-chave AS nomeia esta etapa como base, o que permite usá-la mais tarde como base para outras etapas do Dockerfile.
#### WORKDIR /app
  - WORKDIR: Define o diretório de trabalho dentro do container. O Docker criará o diretório /app e qualquer comando subsequente será executado a partir deste diretório.
#### RUN npm i -g pnpm
  - RUN: Executa um comando dentro do container durante a construção da imagem.
  - Aqui, o comando npm i -g pnpm instala o pnpm globalmente no container. O pnpm é um gerenciador de pacotes para Node.js que é mais rápido e eficiente que o npm.
#### FROM base AS install
  - FROM base AS install: Esta linha cria uma nova etapa de construção chamada install baseada na etapa anterior base.
  - Essa etapa usará a imagem base criada na primeira etapa, permitindo reutilizar as configurações e dependências que já foram definidas.
#### WORKDIR /app
  - Novamente, define o diretório de trabalho para /app dentro do container (embora já tenha sido definido antes, isso é repetido aqui para a clareza e organização).
#### COPY package.json ./
  - COPY: Copia arquivos ou diretórios do sistema de arquivos do host para o sistema de arquivos do container.
  - Este comando copia o arquivo package.json para o diretório de trabalho /app dentro do container. O arquivo package.json é necessário para instalar as dependências do projeto.
#### RUN pnpm install
  - RUN pnpm install: Instala as dependências do projeto usando o pnpm no container. O comando pnpm install instalará os pacotes listados no package.json que foi copiado na etapa anterior.
#### FROM install AS build
  - FROM install AS build: Cria uma nova etapa chamada build baseada na etapa install. Isso permite que você crie o projeto após a instalação das dependências.
#### COPY . .
  - COPY . .: Copia todo o conteúdo do diretório atual do host (onde o Dockerfile está localizado) para o diretório de trabalho /app no container. Isso inclui o código-fonte da aplicação.
#### RUN pnpm build
  - RUN pnpm build: Executa o comando de build da aplicação, que normalmente é usado para transpilar, compilar ou gerar os artefatos finais da aplicação (por exemplo, um bundle ou uma versão otimizada para produção).
#### FROM install AS development
  - FROM install AS development: Cria uma etapa chamada development baseada na etapa install. Esta etapa é configurada para desenvolvimento e tem um comportamento diferente do build ou produção.
#### WORKDIR /app
  - Define o diretório de trabalho como /app novamente (como nas etapas anteriores).
#### COPY . .
  - COPY . .: Copia o código-fonte da aplicação para dentro do container. Isso é necessário para garantir que o container de desenvolvimento tenha todos os arquivos atualizados do código.
#### CMD ["pnpm", "dev"]
  - CMD: Define o comando padrão a ser executado quando o container for iniciado. Aqui, o comando pnpm dev é configurado para iniciar a aplicação no modo de desenvolvimento.
#### FROM base AS production
  - FROM base AS production: Cria uma nova etapa baseada na etapa base para construir a versão de produção da aplicação.
  - Aqui, o container é configurado para ser otimizado para produção, com variáveis de ambiente e usuário específicos.
#### WORKDIR /app
  - Define o diretório de trabalho como /app.
#### ENV NODE_ENV production
  - ENV: Define uma variável de ambiente no container. Aqui, NODE_ENV production define que o ambiente é de produção, o que pode influenciar o comportamento da aplicação (por exemplo, usando configurações específicas para produção, como desabilitar o modo de desenvolvimento).
#### RUN addgroup --system --gid 1001 nodejs
  - RUN addgroup: Cria um novo grupo no sistema dentro do container. Aqui, o comando cria o grupo nodejs com o ID de grupo 1001.
#### RUN adduser --system --uid 1001 nextjs
  - RUN adduser: Cria um novo usuário nextjs no container com o ID de usuário 1001. O uso de um usuário dedicado é uma prática de segurança para evitar que a aplicação seja executada como o usuário root.
#### COPY --from=build /app/next.config.mjs ./next.config.mjs
  - COPY --from=build: Copia arquivos de uma etapa anterior chamada build. Aqui, o arquivo next.config.mjs é copiado da etapa build para o diretório de trabalho do container de produção.
#### COPY --from=build /app/public ./public
  - Copia a pasta public da etapa build para o container de produção. A pasta public geralmente contém arquivos estáticos, como imagens e fontes, usados pela aplicação.
#### COPY --from=build /app/dist ./dist
  - Copia a pasta dist da etapa build. A pasta dist pode conter os artefatos gerados durante o processo de build, como código minificado ou otimizado para produção.
#### COPY --from=install /app/node_modules ./node_modules
  - Copia a pasta node_modules da etapa install, garantindo que as dependências sejam preservadas para o ambiente de produção.
#### COPY --from=install /app/package.json ./package.json
  - Copia o package.json da etapa install para garantir que a configuração das dependências seja usada no container de produção.
#### USER nextjs
  - USER: Define o usuário com o qual o container será executado. Aqui, estamos configurando o container para rodar como o usuário nextjs (criado anteriormente), ao invés de rodar como root, o que é uma boa prática de segurança.
#### EXPOSE 3000
  - EXPOSE: Informa ao Docker que o container escutará na porta 3000. Isso é importante para que você saiba em qual porta acessar a aplicação quando o container estiver em execução.
#### CMD ["pnpm", "start"]
  - CMD: Define o comando padrão a ser executado quando o container de produção for iniciado. Aqui, ele executa pnpm start, que provavelmente inicia a aplicação em modo de produção.
### Arquivo DOCKER-COMPOSE:   
#### version: '3.8'
  - O que faz: Especifica a versão do Docker Compose que você está usando. Versões mais altas têm mais recursos, mas a versão 3.8 é bastante comum e estável para a maioria dos projetos.
  - Por que é importante: A versão define quais funcionalidades e opções estarão disponíveis no arquivo docker-compose.yml.
#### services:
  - O que faz: Define os serviços que serão executados no seu ambiente Docker. Cada serviço pode ser um container isolado, como o app e o banco de dados.
#### app:
  - Este é o serviço que representa a aplicação principal (no caso, provavelmente uma aplicação web).
  ##### restart: always
  - O que faz: Define que o container sempre será reiniciado se ele falhar ou após o sistema reiniciar.
  - Por que é importante: Isso garante que o container da aplicação esteja sempre ativo.
  ##### build:
  - O que faz: Define como a imagem Docker para este serviço será construída.
  ##### context: .
  - O que faz: Especifica o diretório onde o Docker Compose deve procurar o Dockerfile para construir a imagem. O . significa o diretório atual.
  ##### dockerfile: Dockerfile
  - Especifica o nome do arquivo Dockerfile. No caso, ele está sendo nomeado como Dockerfile, que é o padrão.
  ##### container_name: "infnet-guia-website"
  - O que faz: Define o nome do container que será criado para o serviço. Em vez de um nome aleatório gerado pelo Docker, o container terá o nome infnet-guia-website.
  - Por que é importante: Isso facilita a identificação do container.
  ##### stdin_open: true
  - O que faz: Mantém o input do terminal aberto (se necessário) para o container, o que é útil para interagir com o processo em execução (por exemplo, para um processo interativo).
  - Por que é importante: Permite que você envie comandos para o container interativamente.
  ##### ports:
  - O que faz: Mapeia as portas do container para as portas da máquina host.
  ##### 3000:3000
  - O que faz: Indica que a porta 3000 do host será mapeada para a porta 3000 do container. Isso é útil para expor a aplicação da sua máquina para acesso externo (como o navegador).
  - Por que é importante: Faz o container "escutar" na porta 3000 da máquina local.
#### database:
  - Este é o serviço que representa o banco de dados MySQL.
##### image: mysql:8.0
  - O que faz: Especifica que o serviço usará a imagem do MySQL na versão 8.0. O Docker irá baixar e usar esta imagem do Docker Hub, a menos que a imagem já esteja presente localmente.
  - Por que é importante: Usar uma imagem oficial do MySQL permite que você tenha rapidamente um banco de dados funcional sem precisar configurar manualmente.
##### container_name: "infnet-guia-website-database"
  - O que faz: Define o nome do container para o banco de dados, que será infnet-guia-website-database.
  - Por que é importante: Facilita a identificação e acesso ao container do banco de dados.
##### restart: always
  - O que faz: Assim como no serviço app, o container do banco de dados será reiniciado automaticamente em caso de falhas ou após um reinício do sistema.
##### environment:
  - O que faz: Define variáveis de ambiente que são passadas para o container do banco de dados, essencial para configuração do MySQL.
  - Por que é importante: As variáveis de ambiente são essenciais para inicializar o MySQL corretamente com configurações específicas, como o banco de dados, usuários e senhas.
  ###### MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
  - Define a senha do usuário root do MySQL. O valor será obtido de uma variável de ambiente, provavelmente definida em um arquivo .env.
  ###### MYSQL_DATABASE: ${MYSQL_DATABASE}
  - Define o nome do banco de dados que será criado automaticamente ao iniciar o container.
  ###### MYSQL_USER: ${MYSQL_USER}
  - Define o nome de um usuário não-root do MySQL.
  ###### MYSQL_PASSWORD: ${MYSQL_PASSWORD}
  - Define a senha do usuário MySQL especificado acima.
##### ports:
  - O que faz: Mapeia as portas do container para as portas da máquina host.
  - Por que é importante: Permite que o banco de dados MySQL seja acessado a partir da máquina host ou de outros containers.
  ###### 3306:3306
  - Isso significa que a porta 3306 do host será mapeada para a porta 3306 do container. A porta 3306 é a porta padrão do MySQL.
##### volumes:
  - O que faz: Define um volume persistente para o banco de dados. O volume chamado mysql_data será montado no diretório /var/lib/mysql dentro do container.
  - Por que é importante: O MySQL armazenará seus dados no volume mysql_data para que, mesmo se o container for removido ou recriado, os dados não sejam perdidos.
  ###### mysql_data:/var/lib/mysql
  - Esse volume garante que os dados do banco de dados (como tabelas e registros) sejam persistidos entre reinicializações do container.
#### volumes:
  - O que faz: Define volumes que podem ser compartilhados entre containers ou usados para persistência de dados.
  - Por que é importante: Volumes são importantes para garantir que dados importantes (como os dados do banco de dados) sejam persistidos entre a reinicialização ou remoção de containers.
  ##### mysql_data:/var/lib/mysql
  - Define o volume chamado mysql_data, que será utilizado para armazenar dados persistentes do MySQL.
