# 🦞 Guia Definitivo: Transformando seu WSL em um Assistente Pessoal com OpenClaw 🦞

![Se eu fosse uma Lagosta](lobster-me.png "Eu, versão Lagosta")

## 🥸 Sobre o Autor
Bruno é um entusiasta de Inteligência Artificial Generativa e Tech Lead especializado em suporte de aplicações e Engenharia de Confiabilidade de Sites (SRE). Aos 39 anos, divide seu tempo entre a liderança técnica em ambientes corporativos complexos, a exploração de novas tecnologias (como Python, TypeScript e React) e o desenvolvimento contínuo de projetos focados em IA. Este guia nasceu de suas experiências práticas testando, errando e construindo seus próprios ecossistemas de IA locais.

## 📝 Introdução
O conceito de ter um assistente virtual não é novo, mas a possibilidade de ter um totalmente privado, rodando na sua própria máquina e acessível diretamente pelos seus aplicativos de mensagem favoritos, é um divisor de águas. Este guia prático documenta o caminho exato para instalar e configurar o OpenClaw dentro do seu ambiente Windows Subsystem for Linux (WSL).
Neste guia, você não encontrará apenas comandos, mas também insights práticos de quem já percorreu esse caminho (e precisou reinstalar tudo do zero para aprender a forma correta). O objetivo é que, ao final destas páginas, você tenha um assistente rodando de forma segura, inteligente e perfeitamente integrado ao seu fluxo de trabalho.

## 🖥 Um adendo sobre o seu sistema operacional
Todo o passo a passo construído neste guia foi desenhado rodando sobre o WSL2 (Ubuntu no Windows). No entanto, o OpenClaw e suas dependências (Node.js, Ollama, Homebrew) são ferramentas nativas do ecossistema Unix. Se você já possui uma máquina rodando uma distribuição Linux diretamente no hardware, você está com a faca e o queijo na mão. Todos os comandos de instalação, configuração de portas e ativação de serviços (systemd) detalhados nos próximos capítulos se aplicam integralmente ao seu sistema, muitas vezes entregando uma performance ainda melhor para a Inteligência Artificial.

## 📌 O que é o OpenClaw?
O OpenClaw é um programa que transforma a Inteligência Artificial no seu assistente pessoal particular, rodando direto no seu próprio computador (no seu WSL/Ubuntu) para garantir total privacidade. Você vai conversar com ele diretamente pelo seu WhatsApp e Telegram

## 🛠️ Preparando o Terreno (Pré-requisitos)
Vamos preparar o "motor" que o OpenClaw precisa para rodar e o "cérebro" que ele usará para pensar.

> Vou assumir que voce ja esta rodando uma distribuição Linux diretamente ou via WSL. Se não for o caso e precisar configurar o WSL no seu Win 11, de uma olhada nessa pagina: https://learn.microsoft.com/pt-br/windows/wsl/install

### Instalando o "Motor" (Node.js)
O OpenClaw precisa do Node.js (versão 24). No seu terminal Ubuntu (WSL), rode os comandos abaixo:

**Comando A:**
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
```

> 🛑 MUITO IMPORTANTE: Após rodar o comando acima, feche a janela do Ubuntu e abra uma nova para que ele reconheça a instalação.

**Comando B:** 
```bash
nvm install 24
```

### Escolhendo o "Cérebro" (Inteligência)
Você tem duas opções para dar inteligência ao seu assistente:

#### 🟢 OPÇÃO 1: Uso Local via Ollama (100% Grátis e Privado) 🏆

O Ollama permite rodar a IA no seu próprio computador sem precisar de internet para pensar.
**Passo A:** Instalar o Ollama
```bash
curl -fsSL https://ollama.com/install.sh | bash
```

**Passo B:** Baixar o Modelo de IA (Llama 3)
> 💡 Nota: Aqui estamos usando o llama3 por ser um modelo que roda bem na maioria das maquinas. Caso voce prefira um outro modelo, mais leve ou robusto, apenas troque o nome do modelo no comando abaixo e lembre da sua escolha durante o processo de configuração. Voce consegue ver todos os modelos disponiveis no Ollama em: https://ollama.com/search

```bash
ollama run llama3
```

> 💡 Nota: Isso pode demorar alguns minutos dependendo da sua internet. Quando terminar e aparecer uma tela de chat, digite /bye para sair.


#### 🔵 OPÇÃO 2: Uso via Nuvem (Chave de API)

Se você já possui conta de desenvolvedor em provedores de IA (como OpenAI/ChatGPT, Google ou Anthropic/Claude):
1. Acesse o painel da sua IA favorita.
2. Gere uma Chave de API (API Key).
3. Copie esse código longo e guarde-o com segurança para usar na próxima fase.

### Homebrew (O "Instalador de Ferramentas")
Homebrew é uma ferramenta que o OpenClaw usa para instalar algumas de suas **skills**

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

## 🦞 Fase 2: Instalação
Agora vamos instalar o OpenClaw e configurá-lo de uma vez só

**Inicie a Instalação Mágica** 
Copie e cole no seu WSL:
```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

> **O que está acontecendo no meu terminal?**
> Enquanto as linhas de texto rolam, o instalador está:
> 1. Baixando o núcleo: Trazendo os arquivos mais recentes do OpenClaw para sua máquina.
> 2. Configurando o Daemon: Pense no daemon como um mordomo invisível. Ele é um processo que fica acordado em segundo plano, garantindo que o OpenClaw responda suas mensagens mesmo que você feche a janela do terminal.

## 🦞 Configuração Automática
** O Assistente de Configuração (Onboarding)** 
Ao final da instalação, o OpenClaw rodará um diagnóstico (chamado doctor) automaticamente e perguntará se você deseja configurar o gateway (o sistema central dele).

* Selecione **Yes**
* Setup Mode (Modo de Instalação): Escolha **QuickStart**. Isso configura a segurança básica automaticamente para você (criando um "escudo" para que ninguém de fora acesse seu assistente).
* Selecione todas as opções padrão até chegar na opção do Provedor IA/Modelo (LLM Model)

### AI Provider (Provedor de IA)
> 💡 Nota: vamos configurar um setup 100% local e gratis, usando Ollama e rodando um modelo local. Caso você possua uma assinatura de algum modelo premium (Google Gemini, OpenAI ChatGPT, Anthropic Claude, etc.) voce precisara gerar uma chave de API no seu provedor e configrar um novo modelo depois de completar o setup, ou seguir com o setup via API ao inves do local, descrito abaixo.

1. Selecione Ollama (para rodar 100% grátis e privado no seu PC).
2. Base URL: Apenas aperte Enter para manter o texto padrão.
3. Mode: Selecione Local.
4. Default Model (Modelo Padrão): Selecione Keep Current (Manter o atual).

### Channel
O OpenClaw bem mais útil quando você pode falar com ele de qualquer lugar. Vamos dar "ouvidos e boca" ao seu assistente para que você possa conversar com ele pelo celular. 📱

> 💡 Nota: Vamos começar com o setup mais facil, usando o Telegram. Caso ainda não tenha o app instalado, baixe na sua App Store e crie uma conta. Depois de concluir o processo, voce pode configurar o canal de comunicação via WhatsApp.

1. Selecione a opção: **Telegram (Bot API)**
2. Falar com o "Pai dos Bots" Abra o seu aplicativo do Telegram (no celular ou PC) e busque pelo contato @BotFather (certifique-se de que é o oficial, com o selo azul de verificado)
3. Criar o seu Robô
  1. Mande a mensagem /newbot para o BotFather.
  2. Ele vai pedir para você escolher um Nome e um "Username" (que deve terminar com a palavra "bot").
  3. Ao final, ele vai te dar um Token da API (um código bem longo com números e letras). Copie esse código!
4. De volta nas configurações do OpenClaw 💻, selecione a opção: **Enter Telegram bot token**
5. Cole o token que o BotFather gerou

Pronto! Configuração inicial do Telegram completa! 🚀
Apos a conclusão das configuração, iremos ativar e autorizar o chat via Telegram.

> 💡 Dica de Sobrevivência (Memória)
> Sempre que mudar de assunto radicalmente, digite /new na conversa do WhatsApp/Telegram. Isso limpa a "memória de curto prazo" do assistente, deixando ele mais rápido e, caso esteja usando a Opção de um modelo em Nuvem, **economiza muitos créditos!**

### 🔍 Search Provider
Aqui selecione a opção **skip for now**, isso usara as configurações padrão do OpenClaw para busca basica na internet, acesso a sites, etc.

Futuramente, voce podera adicionar um provedor de buscas, que normalmente requer uma chave API, caso possua alguma preferencia ou assinatura.

### 🤹 Skills
As "Skills" (Habilidades) são como os aplicativos que você instala no seu celular. O OpenClaw já vem com "Ferramentas Nativas" (Core Tools) que cuidam do básico, como ler e escrever arquivos no PC ou navegar na web.

Para tarefas específicas que ajudarão a tornar o OpenClaw um assistnte pessoal, aqui estão as Skills que eu recomendo que você marque com a tecla de espaço neste momento:
* 🧩 clawhub (Obrigatório): É a "App Store" do OpenClaw. Instalando isso agora, podemos instalar ou remover qualquer outra skill depois usando apenas uma linha de chat.
* 🧾 summarize: Extremamente útil. Ajuda o assistente a ler artigos, notícias e arquivos gigantescos e te dar um resumo limpo sem gastar muita memória.
* 💎 obsidian: É a melhor opção atual para ele te ajudar a criar, formatar e organizar notas e textos de forma estruturada.
* 🐦 xurl: Ensina o assistente a "ler" os links de produtos, notícias ou sites que você colar para ele.
* 🎤 openai-whisper: Já que você vai usar Telegram (e talvez o WhatsApp), essa skill vai dar a ele a habilidade fantástica de ouvir e transcrever mensagens de áudio!

**Homebrew recommended** e **Show Homebrew install command?**
Caso essas opções aparecem selecione **No** para não ver o comando de instalação, nos ja completamos essa parte na fase 1, no inicio do tutorial.

**Preferred node manager for skill installs**
Selecione a opção **npm**

💡 As skills selecionadas serão instaladas. Apos o termino, aparecerão varias opções de setar API Keys para uso com skills, selecione **No** para todas.

### 🪝 Hooks
Hooks são como "Automações" executadas sob certas condições.

Neste momento iremos selecionar duas opções:
* 💾 session-memory - Sempre a conversa do agente for limpa com o comando `/new`, este hook fará automaticamente um "resuminho" de toda a conversa e salvará isso em um arquivo de anotações seguro no seu computador (uma memória de longo prazo). Assim, ele limpa a memória de curto prazo (ficando rápido), mas mantém um arquivo pesquisável com o histórico das suas conversas para consultar no futuro se você perguntar sobre algo do passad
* 📝 command-logger - Serve para que todos os comandos que o agente executar sejam salvos em um log. Deste jeito, caso tenha algum problema ou queira saber exatamente o que o agente fez depois de voce pedir para ele arrumar sua pasta de fotos, voce consegue pegar tudo que foi executado.


## 🦞🚀 Reta final
Chegamos no ultimo estagio para seu novo crustaceo favorito nascer! 🎉

Neste estagio o Gatway sera iniciado e voce ira receber varias informações sobre como acessar o web console local (algo como o endereço http://127.0.0.1:18789/#token=...)

Quando ver a opção "How do you want to hatch your bot?" selecione **Hatch in TUI (recommended)**

Neste momento, o assistente de configuração sera concluido e o chat com o seu agente sera iniciado.

Aqui voce ja pode conversar com seu agente como se fosse o ChatGPT, Gemini ou outra IA que voce esteja acostumado!

> 💡 Note a mensagem "Enabled systemd lingering", ela é uma excelente notícia e significa que o seu Linux (WSL) foi configurado para manter o motor do OpenClaw rodando para sempre, mesmo que você feche o terminal do Ubuntu.

> ⚠️ Salve bem o token de acesso do OpenClaw, é ele que garante que somente voce possa acessar o seu agente dentro da sua rede. Caso voce perca o token e precise descobri-lo novamente, rode o comando `openclaw config get gateway.auth.token` no terminal.


## 📱 Telegram
Agora que o assistente de instalação e configuração esta completo é hora de ativar o seu chat via Telegram.
Com todas as configurações ja feitas, este passo sera bem simples!

1. Va ao telegram e inicie uma conversa com seu Bot
  * Voce pode encontrar o bot procurando pelo nome na sua lista de contatos ou, na conversa com o BotFather, ele te mandou um link de contato direto com seu novo bot, apos a criação.
2. Diga oi, ou quelquer outra mensagem.
  * O bot respondera com uma mensagem dizendo que acesso não foi configurado. Nesta mesma mensagem ele informara seu user id do Telegram e o codigo de autenticação do OpenClaw. E para facilitar ainda mais, ele te dara o comando exato que voce precisa rodar no seu terminal, algo como `openclaw pairing approve telegram XXXXXXXX`
3. Copie esse comando e rode no seu terminal.

Agora sim, tudo feito e configurado! Voce ja pode conversar com seu agente crustaceo de qualquer lugar! 🦞🚀

# Considerações finais
Espero que este guia tenha sido util, ele foi construido enquanto eu configurava o OpenClaw na minha maquina pela segunda vez, ja que na primeira eu errei a ponto de preferir deletar minha instancia WSL e começar do zero! 😅

> 💡 Dica: basicamente todo comando `openclaw` tem uma pagina de ajuda, adicionando `--help`. Então na duvida, abuse dessa ajuda. Voce tambem pode perguntar para o seu agente como fazr as coisas e normalmente ele consegue te responder. Exemplo: `openclaw --help` `openclaw models --help` `openclaw skills --help`
> 💡 Outra coisa é deixar salvo nos seus favoritos a pagina de documentação do OpenClaw, ela é bem util: https://docs.openclaw.ai/

---
*Este guia foi criado por Bruno Lemos e faz parte do [aggrego-vault](link-do-repo). O conteúdo está licenciado sob a [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/deed.pt-br).*
