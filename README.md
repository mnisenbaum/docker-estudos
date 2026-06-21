# 🐳 Docker na Prática — Aprendizado Interativo

Bem-vindo(a) ao repositório de estudos Docker do **Moises**, educador de redes e automação (Cisco DevNet, Python/Netmiko, CML2).

Este projeto nasceu de uma necessidade simples: entender Docker não pela teoria solta, mas **pondo a mão na massa**, cometendo erros reais e corrigindo na hora — exatamente como a gente troubleshoot uma rede de verdade.

## 🎯 Objetivo

Construir uma trilha de laboratórios práticos sobre Docker, com foco em quem já vem do mundo de **infraestrutura, redes e virtualização** (Cisco, Proxmox, VMs) e quer entender containers com a mesma profundidade técnica.

Não é "decore esses 10 comandos". É entender:
- O que realmente acontece no kernel quando um container nasce
- Como a rede de um container se conecta ao mundo (bridge, NAT, DNAT)
- Por que containers são minimalistas e descartáveis por design
- Como construir, debugar e expor serviços reais (SSH, web, etc.)

## 🧪 Metodologia

Os laboratórios deste repositório foram construídos em **sessões interativas com o Claude (Anthropic)**, usado como instrutor socrático: em vez de receitas prontas, a gente parte de perguntas reais, erros reais (pacote faltando, autenticação falhando, sintaxe de comando que mudou de versão) e resolve tudo em tempo real, documentando o processo — acertos e erros incluídos.

A ideia é que cada lab carregue não só "o comando certo", mas o **caminho até ele** — porque é isso que fixa o aprendizado de verdade.

> 💡 Esse formato também alimenta o **"O estagIArio"**, minha newsletter no LinkedIn sobre os bastidores (às vezes engraçados, às vezes frustrantes) de aprender com IA.

## 📚 Estrutura do repositório

```
docker-estudos/
├── README.md          → você está aqui
├── setup.md           → como preparar uma VM Ubuntu para acompanhar os labs
├── lab01/
│   ├── lab01.md        → roteiro completo do Lab 01 (fundamentos + rede + SSH)
│   └── Dockerfile      → imagem usada no Lab 01
└── (próximos labs serão adicionados aqui)
```

## 🗺️ Labs disponíveis

| Lab | Tema | Status |
|---|---|---|
| [Lab 01](./lab01/lab01.md) | Fundamentos, rede de containers, NAT/DNAT, build de imagem, SSH | ✅ Completo |
| Lab 02 | Docker Compose + Load Balancing (Nginx) | 🔜 Em breve |
| Lab 03 | Simulação de topologia de rede com containers | 🔜 Em breve |

## 🤝 Quer participar?

Esse repositório é aberto. Se você está aprendendo Docker junto, ou já tem experiência e quer contribuir:

- **Como aprendiz**: clone o repo, siga o [setup.md](./setup.md), rode o [Lab 01](./lab01/lab01.md) na sua própria VM, e abra uma *issue* com suas dúvidas, erros encontrados, ou sugestões de melhoria no roteiro.
- **Como colaborador**: sinta-se à vontade pra abrir um *Pull Request* com correções, novos labs, ou variações de exercícios (ex: testar em outras distros, outros cenários de rede).

Toda contribuição que ajude a tornar o caminho mais claro pra quem está começando é bem-vinda.

## 📫 Contato

Encontrou algo quebrado, confuso, ou quer trocar ideia sobre redes + automação + IA? Abra uma *issue* aqui no repositório.

---

*Repositório de estudos pessoal, mantido como parte de uma jornada de aprendizado em automação de redes e infraestrutura.*
