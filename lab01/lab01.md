# Lab 01 — Fundamentos de Docker: Containers, Rede, Imagens e SSH

> Pré-requisito: VM Ubuntu com Docker instalado. Veja [setup.md](../setup.md) se ainda não tiver isso pronto.

## 🎯 Objetivos deste lab

Ao final, você vai ter:
- Rodado seu primeiro container e entendido a diferença entre container, imagem e Dockerfile
- Explorado a rede interna do Docker (bridge, veth, NAT)
- Construído uma imagem própria do zero
- Subido múltiplas instâncias da mesma imagem se comunicando entre si
- Configurado e debugado um serviço SSH real dentro de um container
- Exposto esse serviço para acesso externo via DNAT (port mapping)
- Feito cleanup completo do ambiente

---

## Parte 1 — Primeiro contato

Teste se o Docker está respondendo corretamente:

```bash
docker run hello-world
```

Esse comando baixa uma imagem minúscula, sobe um container, ele imprime uma mensagem de confirmação, e termina. É o "ping" do mundo Docker — confirma que o Engine está funcionando e consegue baixar imagens do Docker Hub.

---

## Parte 2 — Container interativo e explorando a rede

Suba um Ubuntu interativo:

```bash
docker run -it --name meu-ubuntu ubuntu bash
```

- `-it` → modo interativo com terminal
- `--name` → nome fácil de referenciar depois
- `bash` → comando executado dentro do container (te coloca direto num shell)

### ⚠️ Pegadinha #1: comandos básicos não vêm instalados

Imagens oficiais (como `ubuntu`) são minimalistas — vêm "raspadas ao osso". Comandos como `ip` e `ping` **não vêm por padrão**.

Dentro do container:
```bash
apt update
apt install -y iproute2 iputils-ping
```

Agora explore:
```bash
hostname
ip addr show
ip route show
cat /etc/os-release
ps aux
ping -c 4 8.8.8.8
```

Você deve ver uma interface `eth0` com IP no formato `172.17.0.x` — essa é a rede padrão (`bridge`) do Docker.

### Explorando o lado do host

Em outro terminal (no host, fora do container):

```bash
ip addr show docker0
sudo iptables -t nat -L -n -v
```

- `docker0` é o **bridge virtual** do Docker — o "switch virtual" que conecta todos os containers da rede padrão. O IP dele (geralmente `172.17.0.1`) é o gateway que os containers usam pra sair.
- A regra **MASQUERADE** no iptables é o NAT que reescreve o IP de origem do container quando ele sai para a internet — por isso o tráfego "parece" ter saído do próprio host.

**Caminho completo de um pacote saindo do container:**
```
container (eth0) → veth pair → docker0 (bridge/gateway) → NAT (iptables MASQUERADE) → interface física do host → internet
```

---

## Parte 3 — Construindo sua própria imagem

Saia do container (`exit`) e crie uma pasta de trabalho:

```bash
mkdir ~/docker-lab && cd ~/docker-lab
```

Crie um `Dockerfile` simples:

```dockerfile
FROM ubuntu:22.04
RUN apt update && apt install -y iputils-ping iproute2 curl
WORKDIR /app
COPY hello.sh /app/hello.sh
RUN chmod +x /app/hello.sh
CMD ["/app/hello.sh"]
```

E o script `hello.sh`:

```bash
#!/bin/bash
echo "Olá! Esse container já vem com ping e ip prontos."
echo "Meu IP é:"
hostname -I
```

Builde e rode:

```bash
docker build -t meu-lab:v1 .
docker run --rm meu-lab:v1
```

> `--rm` remove o container automaticamente após terminar — útil para containers que executam uma tarefa e finalizam (diferente do `bash` interativo, que fica "vivo" esperando comandos).

### Comandos úteis para explorar imagens

```bash
docker images              # lista imagens locais
docker history meu-lab:v1  # mostra as camadas (layers) da imagem
docker inspect meu-lab:v1  # metadado completo
docker system df           # espaço ocupado por imagens/containers/volumes
```

**Conceito-chave:** Dockerfile é usado **uma única vez**, no `docker build`, para gerar a imagem. Depois disso, containers nascem da imagem — não precisam mais do Dockerfile.

```
Dockerfile --(build)--> Imagem --(run)--> Container (descartável)
```

---

## Parte 4 — Múltiplas instâncias da mesma imagem

```bash
docker run -d --name instancia-1 ubuntu tail -f /dev/null
docker run -d --name instancia-2 ubuntu tail -f /dev/null
```

- `-d` → modo *detached* (background)
- `tail -f /dev/null` → mantém o container vivo (sem ele, um `bash` sem `-it` finalizaria na hora)

Confirme os IPs de cada uma:

```bash
docker inspect -f '{{.NetworkSettings.Networks.bridge.IPAddress}}' instancia-1
docker inspect -f '{{.NetworkSettings.Networks.bridge.IPAddress}}' instancia-2
```

> Nota: em versões mais novas do Docker, o campo legado `NetworkSettings.IPAddress` vem vazio — o IP real fica dentro de `NetworkSettings.Networks.<nome-da-rede>.IPAddress`.

Teste a comunicação entre elas (substitua pelos IPs reais obtidos):

```bash
docker exec -it instancia-1 bash
apt update && apt install -y iputils-ping
ping -c 4 <IP_da_instancia-2>
```

Isso confirma: containers na mesma rede *bridge* se comunicam diretamente, como hosts no mesmo switch.

### Pausar/despausar (cgroup freezer)

```bash
docker pause instancia-1     # congela os processos (cgroup freezer)
docker inspect -f '{{.State.Status}}' instancia-1   # confere o status
docker unpause instancia-1   # retoma de onde parou
```

> `docker inspect` funciona em qualquer estado (running, paused, stopped) porque lê metadado mantido pelo **Docker Engine no host** — não depende do processo dentro do container estar "vivo" para responder.

---

## Parte 5 — Configurando SSH (com os erros incluídos!)

### Instalando sem precisar abrir o container

```bash
docker exec instancia-1 apt update
docker exec instancia-1 apt install -y openssh-server
```

> `docker exec <container> <comando>` executa algo dentro de um container já rodando, sem precisar do `-it` quando é só um comando pontual (sem necessidade de sessão interativa).

### ⚠️ Pegadinha #2: login root via senha costuma ser bloqueado

Mesmo configurando a senha do root corretamente:

```bash
docker exec instancia-1 bash -c "echo 'root:senha123' | chpasswd"
docker exec instancia-1 ssh-keygen -A
docker exec instancia-1 service ssh start
```

...o SSH pode recusar a senha do `root` mesmo estando correta. Isso geralmente é o `PermitRootLogin` (ou política PAM) bloqueando especificamente logins root por senha, por padrão de segurança.

**Solução mais simples e mais correta: criar um usuário comum.**

```bash
docker exec instancia-1 useradd -m -s /bin/bash moises
docker exec instancia-1 bash -c "echo 'moises:senha123' | chpasswd"
```

Teste o SSH de uma instância para a outra:

```bash
docker exec -it instancia-2 bash
apt update && apt install -y openssh-client
ssh moises@<IP_da_instancia-1>
```

Deve funcionar — usuários comuns não sofrem a mesma restrição.

---

## Parte 6 — Consolidando tudo em um Dockerfile reproduzível

Em vez de repetir os comandos manuais toda vez, documentamos tudo no [`Dockerfile`](./Dockerfile) deste lab. Ele já inclui:

- Pacotes de rede (`iproute2`, `iputils-ping`)
- SSH server pré-configurado, com geração de chaves e diretório necessário
- `PasswordAuthentication yes` já habilitado
- Usuário comum (`moises`) já criado, evitando a pegadinha do `PermitRootLogin`

Build e execução:

```bash
docker build -t meu-lab-ssh:v1 .
docker run -d --name lab-ssh-1 meu-lab-ssh:v1
docker run -d --name lab-ssh-2 meu-lab-ssh:v1
```

> Note que `CMD ["/usr/sbin/sshd", "-D"]` roda o SSH em foreground — ele mesmo é o processo principal (PID 1) que mantém o container vivo. Não precisamos mais do truque `tail -f /dev/null`.

---

## Parte 7 — Expondo para a rede local (DNAT)

Até agora, o SSH só era acessível de dentro da rede interna do Docker (ou do próprio host, que tem rota direta via `docker0`). Para acessar de **outra máquina da rede local**, é preciso mapear a porta:

```bash
docker stop lab-ssh-1 && docker rm lab-ssh-1
docker run -d --name lab-ssh-1 -p 2222:22 meu-lab-ssh:v1
```

- `2222` → porta externa, na VM
- `22` → porta interna, no container

Confirme o mapeamento:

```bash
docker ps
sudo iptables -t nat -L -n -v | grep 2222
```

De outra máquina na rede local:

```bash
ssh -p 2222 moises@<IP_da_VM_Ubuntu>
```

> ⚠️ Se não conectar, verifique o firewall da VM: `sudo ufw status` e, se necessário, `sudo ufw allow 2222/tcp`.

**Esse é o caminho DNAT completo:**
```
máquina externa → IP_da_VM:2222 → iptables DNAT → IP_interno_do_container:22 → sshd
```

---

## Parte 8 — Cleanup

Parar e remover containers:
```bash
docker stop $(docker ps -aq)
docker rm $(docker ps -aq)
```

Remover imagens:
```bash
docker rmi $(docker images -q)
```

Ou, para zerar tudo de uma vez (containers parados, imagens não usadas, redes, volumes, cache):
```bash
docker system prune -a --volumes -f
```

Confirme que zerou:
```bash
docker ps -a
docker images
docker system df
```

---

## 📌 Conceitos-chave deste lab

- **Container ≠ VM**: compartilha o kernel do host via namespaces + cgroups; não virtualiza hardware
- **Imagem** é o molde imutável; **container** é a instância descartável
- **Dockerfile** só é usado no momento do `build` — não tem papel depois disso
- **`docker0`** é o bridge/switch virtual que conecta containers da rede padrão
- **NAT (saída)** disfarça o IP do container como IP do host ao sair para a internet
- **DNAT (entrada / `-p`)** redireciona uma porta externa do host para uma porta interna do container
- **`docker inspect`** lê metadado mantido pelo Engine no host — não depende do container estar rodando
- Containers minimalistas exigem instalar manualmente até ferramentas "básicas" como `ping` e `ip`

## ➡️ Próximo passo

Seguir para o **Lab 02** (em breve): Docker Compose + balanceamento de carga com Nginx, escalando múltiplas réplicas do mesmo serviço.
