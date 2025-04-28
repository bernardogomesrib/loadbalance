# Descrição do projeto
é um projeto que mostra html com um loadbalance com a estratégia round-robin, embora existam outras estratégias,
para o que foi passado de atividade esta é a que mais facilmente se encaixa com o que foi pedido e com o tipo de 
serviço prestado pelo servidor.
outras estratégias não teriam o comportamento esperado 100% das vezes. a estratégia round-robin é boa para exibição
de html estático por dividir as requisições entre os servidores que foi configurado por dividir as requisições entre
todos os servidores. se existisse uma diferença de capacidade de processamento de requisições entre servidores, seria
melhor utilizar a Weighted Round-Robin que define pesos para cada maquina para definir de forma equivalente a capacidade
de processamento de cada uma. por exemplo, foi identificado que o servidor 1 dentre os 3 que a empresa aluga, tem a capacidade
de processar 20% a mais que os servidores 2 e 3, do qual ambos tem capacidade de processamento proxima com margem de erro baixa,
para o servidor 1 poderia se por peso 12 para ela enquanto as outras ficariam com peso 10, dessa forma 20% a mais das requisições
iriam para o servidor 1

esta configuração deve ser feita no arquivo nginx.conf
alterando o atual para:
```
events {}

http {
    upstream backend {
        server server1:80 weight=12;
        server server2:80 weight=10;
        server server3:80 weight=10;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend;
        }
    }
}
```
como o meu caso foi todo o processo feito na mesma maquina, e no docker, onde está escrito server1, server2 e server3,
é interpretado pelo dns do docker como o url ou ip da maquina

existem outras formas de balanceamento, porém cada aplicação deve ser estudada para que seja definido corretamente a estratégia de balanceamento para ela.

# foi utilizado neste projeto:

- NGINX para hostear o arquivo html
- NGINX para loadbalance
- HTML para fazer o arquivo
- docker para criar as maquinas virtuais
- docker-compose para ler o arquivo docker-compose.yml e gerar as maquinas virtuais

# Arquitetura do projeto

![Arquitetura do Projeto](./round_robin.png){ width=528px height=384px }


# Como a capacidade poderia ser expandida no futuro

no caso deste projeto que é só exibição de html a possibilidade de expansão é só por mais maquinas, já se fosse algo que eu tenho que usar
um banco de dados e fileserver ai já complica um pouco mais.
se a estratégia fosse utilizar um banco de dados só entre todas as maquinas, e um backend em cada maquina nova, e um file server para
responder a todos os backs então só crescer a quantidade de maquinas que rodam o backend é o suficiente, caso o fileserver ou banco de dados
for posto em cada uma das maquinas, é importante por uma forma de igualar todos os fileservers e bancos de dados durante o funcionamento via 
websocket durante o funcionamento normal de várias maquinas ou caso seja inciada uma nova maquina algum script de cópia dos arquivos 
necessários junto com a conversa via websocket, também é importante que cada requisição seja mantido um protocolo de retorno apenas caso 
seja feito o processamento de alterações ou criações em mais de uma das maquinas para evitar que alguma alteração seja feita apenas em uma 
maquina, e ao fechar ou ela ter problemas, a alteração ter ocorrido em mais de um servidor. 
esta conexão via websocket também deve ser observada corretamente, pois deve ser feita em uma rede privada ou via TLS, para que tenha algum 
nivel de segurança no trasnporte das atualizações entre servidores. qualquer socket que seja configurado de forma incorreta ou que não usam 
TLS estão sugeitos a ataques como Man in the middle ou packet sniffing.
