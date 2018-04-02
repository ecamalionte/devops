# Exemplos de estratégias de DevOps

Esses exmplos são exercícios de DevOps retirados do livro `Caixa de Ferramentas DevOps` escrito pelo Gleicon

# Comandos
Para virtualizar os exemplos, basta utilizar os comandos de ciclo de vida do Vagrant conforme abaixo:

Para criar máquinas

```
$ vagrant up
```

As máquinas podem ser destruídas com

```
$ vagrant destroy
```

Para se conectar use

```
$ vagrant ssh
```

Se o exemplo criar mais de uma máquina, utilize o nome da máquina:

```
$ vagrant ssh web
$ vagrant ssh db
$ vagrant ssh node1
```

Se quiser repetir apenas o provisionamento, use:

```
$ vagrant provision
```

# Performance

## Injetando carga sintética no sistema

Esse exemplo com Apache Benchmark envia 100 requests, 10 em concorrência, a cada 5 segundos.

 ```
 $ while true; do ab -c 10 -n 100 http://target/; sleep 5;
   done
 ```

## Analisando a performance

Dentro do server, verifica o Load Average
 ```
 $ uptime
 ```

Dentro do server, verifica as estatísticas de de CPU e memória
 ```
 $ vmstat

 procs -----------memory---------- ---swap-- -----io----
  r b   swpd   free   buff   cache   si  so      bi  bo
  6 0    0     492292 27484  38802    0  0       37  98
-system-- ------cpu-----
  in cs    us sy id wa st
 799 1599  30 3  65 1  1
 ```

A última coluna do bloco CPU, chamada st, está com valor 1.
Esta coluna representa o tempo roubado (stolen time) pelo hypervisor.
É um indicador de que a máquina virtual não tem o tempo que esperava para executar.
É um contador importante para ser monitorado.
Caso o tempo roubado cresça ou fique constante acima de 0, é hora de recriar a
máquina virtual em outro host.

## Simulando Limitações de Rede

Vamos usar a documentação do Linux Advanced Routing
How-To (http://lartc.org/howto/lartc.ratelimit.single.html) para
limitar a banda desta máquina em 256kbps. Esse teste pode ser feito
na máquina do Banco de Dados para representar uma limitação de rede entre
a App e DB

Na máquina do banco:

```
$ export DEV=eth0
$ tc qdisc add dev $DEV root handle 1:
cbq avpkt 1000 bandwidth 100mbit
$ tc class add dev $DEV parent 1:
classid 1:1 cbq rate 256kbit allot 1500 prio 5 bounded isolated
$ tc filter add dev $DEV parent 1:
protocol ip prio 16 u32 match ip dst 10.0.0.100 flowid 1:1
```

Para remover limitação de rede

```
tc qdisc del dev $DEV root
```
