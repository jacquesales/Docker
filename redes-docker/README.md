# Docker Pets

Exercicio de redes Docker do curso disponível na plataforma [Digital Innovation One](https://digitalinnovation.one/)


### Arquitetura
Imagem chrch/docker-pets : container front-end do Python Flask que exibe imagens aleatórias de animais domésticos 
Imagem consul: loja KV back-end que armazena o número de visitas que os serviços da web recebem. Ele está configurado para se autoinicializar com 3 réplicas para que tenhamos persistência tolerante a falhas

-------------

##### CENARIO I
- Criando uma rede modo **Host **para comunicação dos containers
`$ docker network create -d bridge petsBridge`
- Backend
`$ docker run -d --name database --network petsBridge consul`
- Frontend
`$ docker run -d -it --env "DB=database" --name web --network petsBridge -p 8000:5000 chrch/docker-pets:1.0`

-------------

##### CENARIO II
- Criando uma rede modo **Overlay** para comunicação dos containers, agora em diferentes hosts (cluster **Swarm**)
Porta TCP 2377 para comunicações de gerenciamento de cluster 
Porta TCP e UDP 7946 para comunicação entre nodes 
Porta UDP 4789 para tráfego de rede de sobreposição

###### *Instância 1 - principal*

`$ docker swarm init --advertise-addr <IP>`

`$ docker network create -d overlay petsOverlay`

`$ docker service create --name database --network petsOverlay consul`

`$ docker service create --name web --network petsOverlay -p 5000:5000 -env 'DB=database' chrch/docker-pets:1.0`

###### *Instância 2*

- Vinculando demais instâncias
`$ docker swarm join --token SWMTKN-1-1hjmrpr9l3rie91k2luzdqutpcl7gzuo4798kgqwxi8efcbi72-5kqln75pjsa8v82pgq5rr7frp <IP>`

- Escalando e balanceando a aplicação
`$ docker service scale web=3`

=============


Inspiração: https://github.com/mark-church/docker-pets