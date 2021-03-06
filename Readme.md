![Imagem do Projeto](https://imagizer.imageshack.com/img922/6745/wSCLM0.png)


## Satisfazendo as dependências
Instale o gerenciador de pacotes [pip](https://pip.pypa.io/en/stable/installing/) no servidor Jenkins que executara a playbook e rode o comando abaixo:
```bash
pip install -r requirements.txt
```

No servidor jenkins baixe os seguintes plugins:

Blue Ocean

Ansible

RocketChat Notifier

## Configurando o boto


Configure o boto e suas credenciais de acesso no servidor jenkins que ira executar a playbook.

*O diretório .aws deve estar na home do usuário que ira executar a job, exemplo root ou o próprio usuário jenkins dependendo de sua configuração.

*Não se esqueça de trocar os valores do segredo e da chave de sua conta no comando abaixo.


```bash
mkdir -p ~/.aws/
echo -e "[default]\\naws_access_key_id = COLOQUE-SUA-ACCESS-KEY-AQUI\\naws_secret_access_key = COLOQUE-SUA-SECRET-ACCESS-KEY-QUI" > ~/.aws/credentials
```
Configure a região (em meu caso eu escolhi Oregon).


Faça um fork do repositório e adicione o mesmo em seu servidor Jenkins.

## Execução da Job 
![Execução da Job](https://imagizer.imageshack.com/img923/4374/wEel0t.png)

Antes de executar a Job deve-se alterar a variável *subnet_id:* no arquivo aws.yml você deve colocar um VPC-ID valido de sua conta AWS.

## Steps da Job

Aguarde a execução dos steps da Jobs:

1. A execução da Playbook se conecta a sua conta da AWS e cria uma instancia. keypair e security-group. Na criação da instancia ele usa o CloudInit com user-data para instalar o git e docker, após estas instações o repositório da aplicação [MongApp](https://github.com/andpupilo0182/MongApp/tree/Prod) e clonado junto com o Dockerfile, nesse momento e gerada uma imagem temporária com os componentes necessarios do flask e um docker pull na imagem do MongoDB (assim como suas configurações de database e collections).

2. Este step consiste em testar o acesso a instancia com o comando ssh usado a chave criada pela execução da playbook anterior

3. Neste step o sleep é maior pois é necessário esperar que o Docker efetue o download de todas as imagens necessárias, apos isso e efetuado curl na instancia recem criada para testar a conexão com a porta 80

4. É efetuada uma carga na API com o nome Jenkins, para verificar basta acessar http://IP-DA-INSTANCIA/api/users/lista ou curl http:/IP-DA-INSTANCIA/api/users/lista 


![Payload](https://imagizer.imageshack.com/img922/6677/N2rFdj.png)


5. Caso tenha um servidor de mensageria [RocketChat](https://rocket.chat/) será enviada uma notificação informando o término da job.

## Executando o Rocket em sua infra-estrutura

Para executar o RocketChat de maneira fácil em sua infra-estrutura usaremos o docker com dois containers.

```bash
docker pull rocket.chat
docker run --name db -d mongo:3.0 --smallfiles
docker run --name rocketchat -p 80:3000 --env ROOT_URL=http://localhost --link db -d rocket.chat
```

Acesse a porta 80 da maquina que possui o container para finalizar a instalação do RocketChat e crie um canal chamado gupy-analistas crie um WebHook entrante clicando em Administração / Integrações e configure o tokem em seu servidor jenkins em configurações / RocketChat.
![RocketChat](https://imagizer.imageshack.com/img922/547/gLXMDB.png)


## Considerações

Ferramentas utilizadas:


Ansible - Gerenciador de configurações agent-less: Optei por usar o Ansible pelo fato do mesmo possuir muito modulos nativos e dispensar a utilização de agentes nas maquinas alvos, alem da DSL da ferramenta ser simples.


Docker - Para simplificar o deploy dos serviços foi um utlizada ferramenta para virtualizar os serviços necessarios como Apache e MongoDB.


Boto - Interface CLI em Python para AWS que possui integração com Ansible.


MongoDB - Banco de dados documental NoSQL segundo o [Google trends](https://trends.google.com/trends/explore?date=all&q=%2Fm%2F05z_r2n,%2Fm%2F04f32m3,%2Fm%2F09gnj_f,%2Fm%2F03wfh72,Neo4j) o banco de dados mais popular do mundo, resolvi coloca-lo no projeto pelo fato de sua simplicidade e robustez.


Apache - Geralmente utilizo Apache ou Lighthttp para pequenos projetos o Nginx utilizo para conteúdos estáticos ou para fazer proxys reversos.


Flask - Tenho uma certa facilidade com PHP e Python para construir a API resolvi usar o FLask.


RockChat - Projeto nacional e um dos melhores mensageiros da atualidade.

Jenkins - Job Scheduler geralmente utilizo Rundeck para projeto de gerencia de configurações, porem como se trata do deploy de uma aplicação o Jenkins tem suas vantagens devido a grande quantidade de plugins disponíveis.

AWS - Foi escolhido esse player simplesmente pela sua popularidade, caso queira aumentar o numero de instancias basta aumentar o numero de COUNT no arquivo aws.yml para efetuar o deploy em múltiplos servidores.
