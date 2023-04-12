## Acesso à Memória
Desenvolvido e mantido pela Artefactual Systems
Customização para atender ao projeto VIVASUS - FESFTech

AtoM (abreviação de Access to Memory) é um aplicativo de código aberto baseado na Web para descrição e acesso de arquivamento baseado em padrões. O aplicativo é multilíngue e multi-repositório. Encomendado pela primeira vez pelo Conselho Internacional de Arquivos (ICA) para tornar mais fácil para as instituições arquivísticas em todo o mundo colocarem seus acervos online usando os padrões descritivos do ICA, o projeto desde então se tornou um projeto conduzido pela comunidade usado internacionalmente. Saiba mais em:

https://www.accesstomemory.org

Você é livre para copiar, modificar e distribuir AtoM com atribuição sob os termos da licença AGPLv3. Consulte o arquivo LICENSE para obter detalhes.

Instalação

Instalação de produção

AtoM destina-se a ser instalado usando um sistema operacional baseado em Linux. Usamos as versões LTS do Ubuntu em desenvolvimento e teste, mas os usuários também instalaram com sucesso em outras distribuições.

Guias de instalação do Linux

Para outras instalações de SO, recomendamos a virtualização.

Ambientes de desenvolvimento

Se você deseja instalar uma cópia local do AtoM para teste e/ou desenvolvimento, mantemos dois ambientes de desenvolvimento:

Docker

Na instalação do VIVA SUS foi disponibilizado atrás do Docker, o Dockerfile e Docker-Compose encontra-se na Pasta `docker`
Instalar o Docker e o Docker Compose

O Docker funciona da mesma forma, esteja você usando Docker para Windows, Docker para Mac ou Docker no Linux. Para o último, verifique se o Docker Engine e o Docker Compose estão instalados seguindo as instruções nos links.

Vamos garantir que o cliente Docker possa alcançar o mecanismo. O comando a seguir listará os contêineres em execução no momento:

`docker ps`

Agora, usando o git, verifique as fontes do AtoM e altere seu diretório atual:

git clone -b stable/2.5.x https://github.com/fesflabs/vivasus.git vivasus
cd vivasus

Observação

O código-fonte (toda a pasta “atom”) é copiado para o contêiner do Docker no processo de compilação e usado para criar a instância. Por exemplo, clone a ramificação “stable/2.5.x”

Agora defina a variável de ambiente COMPOSE_FILE para informar ao Compose qual é a localização do nosso arquivo YAML. Você poderia fazer o mesmo usando o sinalizador -f, mas não queremos fazer isso toda vez que invocarmos o comando docker-compose.

# Para usuários bash (a maioria de vocês)
export COMPOSE_FILE="$PWD/docker/docker-compose.dev.yml"

# Para outros usuários
set -lx COMPOSE_FILE (pwd)/docker/docker-compose.dev.yml
É hora de usar o Docker Compose para provisionar nossos contêineres:

Importante

Em alguns sistemas operacionais, os limites de memória virtual padrão podem ser baixos. O Elasticsearch aumenta um desses limites quando instalado via pacotes e no container que será criado abaixo. No entanto, o mesmo limite precisa ser aumentado no host que executa esse ambiente. Verifique a documentação do Elasticsearch para obter mais informações.

Os contêineres do aplicativo usam php:7.2-fpm-alpine como imagem base. Se uma versão antiga desta imagem já tiver sido baixada pelo mecanismo do Docker no host, execute docker pull php:7.2-fpm-alpine para obter a versão mais recente antes de criar os contêineres. Tem que ser baseado no Alpine v3.8 ou superior para poder instalar alguns pacotes.

# Crie e inicie contêineres. Isso pode demorar um pouco na primeira vez que você executar
# porque todas as imagens devem ser baixadas (por exemplo, percona, memcached)
# e a imagem AtoM deve ser construída.

`docker-compose up -d`

# Execute um comando no container atom em execução: purge database

`docker-compose exec atom php ferramentas do symfony:purge --demo`

# Execute outro comando: build stylesheets
`docker-compose exec atom make -C plugins/arDominionPlugin`

É isso! Você iniciou os contêineres e os colocou em segundo plano, preencheu o banco de dados e iniciou o índice Elasticsearch. Você pode começar a desenvolver imediatamente. As alterações feitas no código-fonte entrarão em vigor imediatamente.

Devido a um bug que ainda não foi resolvido, o trabalhador AtoM precisa ser reiniciado após o preenchimento do banco de dados pela primeira vez:

`docker-compose reiniciar atom_worker`

Importante

Para evitar a redefinição dos arquivos de configuração para sua versão padrão cada vez que os contêineres são criados, os seguintes arquivos são gerados apenas se já não existirem no código-fonte:

apps/qubit/config/app.yml
apps/qubit/config/factories.yml
apps/qubit/config/settings.yml

O Docker Compose permite que você execute muitas ações diferentes. Consulte a documentação para obter mais ajuda. Por exemplo, você pode monitorar a saída de alguns de seus contêineres da seguinte maneira:

`docker-compose logs -f atom atom_worker nginx`

Você também pode dimensionar o trabalhador AtoM conforme necessário:

`docker-compose up -d --scale atom_worker=2`

Vamos verificar se dois trabalhadores se inscreveram no Gearman:

# Estabeleça uma conexão TCP com o gearmand, porta 4730

`docker-compose exec atom bash -c "nc gearmand 4730" `

# Enviar comando de STATUS

STATUS

0a2a58137e05032d1140fdbd0d6dccbb-arInheritRightsJob 0 0 2
0a2a58137e05032d1140fdbd0d6dccbb-arFileImportJob 0 0 2
0a2a58137e05032d1140fdbd0d6dccbb-arInformationObjectXmlExportJob 0 0 2
0a2a58137e05032d1140fdbd0d6dccbb-arActorXmlExportJob 0 0 2
0a2a58137e05032d1140fdbd0d6dccbb-arCalcularDescendenteDatasTrabalho 0 0 2
0a2a58137e05032d1140fdbd0d6dccbb-arXmlExportSingleFileJob 0 0 2
0a2a58137e05032d1140fdbd0d6dccbb-arUpdatePublicationStatusJob 0 0 2
0a2a58137e05032d1140fdbd0d6dccbb-arObjectMoveJob 0 0 2
0a2a58137e05032d1140fdbd0d6dccbb-arInformationObjectCsvExportJob 0 0 2
0a2a58137e05032d1140fdbd0d6dccbb-arUpdateEsIoDocumentsJob 0 0 2
0a2a58137e05032d1140fdbd0d6dccbb-arActorCsvExportJob 0 0 2
0a2a58137e05032d1140fdbd0d6dccbb-arRepositoryCsvExportJob 0 0 2
0a2a58137e05032d1140fdbd0d6dccbb-arFindingAidJob 0 0 2
0a2a58137e05032d1140fdbd0d6dccbb-arGenerateReportJob 0 0 2

Você pode interromper temporariamente todos os serviços com `docker-compose stop` (que precisará de docker-compose up -d posteriormente para iniciar os serviços novamente) ou interromper e remover contêineres, rede e volumes relacionados executando:

`docker-compose down --volumes`

Conecte-se ao AtoM

Você pode executar o seguinte comando para verificar o status e outras informações sobre os contêineres:

$ `docker-compose ps`

         Nome Comando Estado Portas
-------------------------------------------------- -------------------------------------------------- -
docker_atom_1 /atom/src/docker/entrypoin ... Até 9000/tcp
docker_atom_worker_1 /atom/src/docker/entrypoin ... Até 9000/tcp
docker_nginx_1 nginx -g daemon desligado; Up 0.0.0.0:80->80/tcp
docker_elasticsearch_1 /bin/bash bin/es-docker Up 127.0.0.1:63002->9200/tcp, 9300/tcp
docker_percona_1 /docker-entrypoint.sh mysqld Up 127.0.0.1:63003->3306/tcp
docker_memcached_1 docker-entrypoint.sh -p 11 ... Up 127.0.0.1:63004->11211/tcp
docker_gearmand_1 docker-entrypoint.sh gearmand Up 127.0.0.1:63005->4730/tcp
Como você pode ver na coluna da direita:

AtoM e seu worker compartilham a porta 9000, apenas na rede de containers.
O Nginx é acessível na porta 80 do host e de fora (se o host permitir).
O Elasticsearch pode ser acessado pela porta 63002, somente do host.
Percona (MySQL) pode ser acessado através da porta 63003, somente do host.
O Memcached pode ser acessado pela porta 63004, somente do host.
O servidor Gearman pode ser acessado pela porta 63005, somente do host.
O AtoM agora deve estar acessível no seu navegador. Para se conectar do host, use o seguinte endereço: http://localhost:63001.

Os detalhes de login padrão são:

Nome de usuário: demo@example.com
Senha: demo

Dica

Enquanto espera, aproveite para conferir nosso Dockerfile, que descreve as etapas que são executadas para construir a imagem AtoM. É baseado no Alpine Linux + PHP 7.2 e no restante das dependências. Além disso, nosso arquivo docker-compose.dev.yml mostra como o AtoM é orquestrado junto com suas dependências de serviço. É um ambiente destinado a ser usado por desenvolvedores.

Vagrant

Outros recursos

Site - a casa do projeto AtoM!
Documentação - onde você encontrará nossos manuais de usuário, administrador e desenvolvedor. Criamos versões de nossos manuais para cada versão principal.
Wiki - recursos da comunidade e do projeto, documentação de desenvolvimento, notas de versão e muito mais.
Fórum do usuário - Fórum e lista de discussão para perguntas do usuário (técnicas e do usuário final), discussão e muito mais.
SlideShare - onde carregamos todos os conjuntos de slides de nossas apresentações em conferências e campos de treinamento!
Suporte pago: suporte pago, hospedagem, treinamento, temas, migração de dados, consultoria e contratos de desenvolvimento de software da Artefactual.
Contribuindo
Obrigado pelo seu interesse em contribuir com o projeto AtoM!

Consulte nosso arquivo de diretrizes de contribuição para obter mais informações.