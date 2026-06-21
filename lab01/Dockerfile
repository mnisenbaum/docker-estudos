FROM ubuntu:22.04

# Evita prompts interativos durante o build (ex: timezone)
ENV DEBIAN_FRONTEND=noninteractive

# Pacotes de rede e SSH
RUN apt update && apt install -y \
    iproute2 \
    iputils-ping \
    curl \
    openssh-server \
    && rm -rf /var/lib/apt/lists/*

# Gera as chaves do host SSH (necessário, senão o sshd não inicia)
RUN ssh-keygen -A

# Permite autenticação por senha
RUN sed -i 's/^#\?PasswordAuthentication.*/PasswordAuthentication yes/' /etc/ssh/sshd_config

# Cria usuário não-root
# (login root via senha costuma ser bloqueado por padrão de seguranca / PAM)
RUN useradd -m -s /bin/bash moises && \
    echo 'moises:senha123' | chpasswd

# Diretório necessário para o sshd rodar
RUN mkdir -p /run/sshd

WORKDIR /app

# Documenta a porta esperada (não abre porta por si só - isso é feito no `docker run -p`)
EXPOSE 22

# Roda o SSH em foreground - ele é o processo principal (PID 1) que mantém o container vivo
CMD ["/usr/sbin/sshd", "-D"]
