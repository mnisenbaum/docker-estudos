# 🛠️ Setup — Preparando sua VM Ubuntu para os Labs

Este guia descreve como preparar uma VM Ubuntu do zero para acompanhar os laboratórios deste repositório.

## Por que VM e não WSL/Docker Desktop?

Esses labs foram desenhados pra rodar numa **VM Ubuntu "pura"** (não WSL, não Docker Desktop no Windows). O motivo é didático: numa VM Linux real, você vê o comportamento genuíno do Docker Engine — namespaces, iptables, cgroups — sem nenhuma camada de tradução por trás escondendo o que está realmente acontecendo.

Depois que os conceitos estiverem sólidos, usar WSL2 + Docker Desktop no dia a dia é perfeitamente válido (e é o que muita gente usa em produção/dev). Mas para *aprender*, Linux puro elimina uma variável de confusão.

## 1) Requisitos

- Um hypervisor para criar a VM: Proxmox, VirtualBox, VMware, ou até uma VM na nuvem (AWS, Oracle Cloud Free Tier, etc.)
- Imagem ISO da última versão Desktop LTS do Ubuntu. Atualmente é **Ubuntu 26.04 LTS** — [ubuntu.com/download](https://ubuntu.com/download)
- Recursos recomendados: 2 vCPU, 4 GB RAM, 100 GB de disco

## 2) Criando a VM

Se estiver usando **Proxmox** (como no setup do autor deste repo):

1. Faça upload da ISO do Ubuntu Server para o storage do Proxmox
2. Crie uma nova VM, com a ISO montada
3. Configure a rede como **Bridge** (não NAT do hypervisor) — isso garante que a VM recebe um IP da sua rede local de verdade, facilitando o acesso de outras máquinas (relevante no Lab 01, etapa de exposição via SSH)
4. Siga o instalador padrão do Ubuntu Server (idioma, teclado, partição de disco, usuário)

> 💡 Durante a instalação do Ubuntu Server, há uma tela que pergunta se você quer instalar o **OpenSSH Server**. Marque "sim" — isso facilita acessar a VM remotamente do seu terminal principal, sem precisar usar a console do hypervisor.

## 3) Após a instalação — atualizando o sistema

Acesse a VM (via console do hypervisor ou SSH) e rode:

```bash
sudo apt update && sudo apt upgrade -y
```

## 4) Instalando o Docker Engine

Siga o método oficial (via repositório do Docker, garante versão mais recente):

```bash
# Remove versões antigas/conflitantes, se existirem
sudo apt remove docker docker-engine docker.io containerd runc -y

# Instala dependências
sudo apt install ca-certificates curl gnupg -y

# Adiciona a chave GPG oficial do Docker
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Adiciona o repositório
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Instala o Docker Engine
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

## 5) Permitindo rodar Docker sem `sudo` (opcional, mas recomendado)

```bash
sudo usermod -aG docker $USER
```

Depois disso, **saia e entre de novo na sessão** (logout/login, ou reinicie a VM) para o grupo ser aplicado.

## 6) Validando a instalação

```bash
docker --version
docker run hello-world
```

Se aparecer a mensagem de boas-vindas do `hello-world`, está tudo certo — pode seguir para o [Lab 01](./lab01/lab01.md).

## 7) (Opcional) Firewall

Se for seguir até a etapa de expor serviços para a rede local (como fazemos no Lab 01), é bom já deixar o `ufw` configurado:

```bash
sudo ufw status
```

Se estiver ativo e você precisar liberar uma porta específica mais adiante:
```bash
sudo ufw allow <porta>/tcp
```

---

✅ Pronto! Sua VM está preparada. Siga para o **[Lab 01 — Fundamentos de Docker](./lab01/lab01.md)**.
