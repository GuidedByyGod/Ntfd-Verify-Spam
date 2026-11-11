# Ntfd Verify Spam

> Bot de Discord com **verificação por captcha** e **anti-spam por canal armadilha**, usando Components V2.

O Ntfd Verify Spam junta dois sistemas em um único bot:

1. **Verificação** — novos membros só veem o canal de verificação. Ao clicar em **Verify/Verificar** e resolver um captcha, recebem o cargo que libera o servidor.
2. **Anti-spam** — um canal armadilha registra e pune contas comprometidas que saem mandando spam. A mensagem é apagada, o usuário recebe timeout e a moderação recebe um log com botão para remover o castigo.

> **Pacote:** o código-fonte está em `Ntfd-Verify-Spam.rar`, protegido por senha (RAR5/AES-256, com cabeçalhos criptografados). Extraia antes de rodar. O hash do arquivo está em `SHA256SUMS.txt`.

---

## Status

| Item | Valor |
|---|---|
| Runtime | Node.js **20+** |
| Biblioteca | discord.js 14.27 |
| Interface | Components V2 |
| Captcha | Imagem (`@napi-rs/canvas`) ou matemático |
| Armazenamento | JSON local (`data/db.json`) |
| Licença | Proprietária — todos os direitos reservados |

---

## Recursos

### Verificação
- Painel público com botão **Verify/Verificar**.
- Captcha em imagem com ruído, rotação e linhas.
- Fallback automático para captcha matemático se o canvas/fontes não estiverem disponíveis.
- Captcha válido por **3 minutos** e **3 tentativas**.
- Resposta por modal, visível apenas para o usuário.
- Entrega automática do cargo configurado.
- Botões continuam funcionando depois de reiniciar o bot (customId persistente).
- Aviso quando o cargo do bot está abaixo do cargo de verificado.

### Anti-spam
- Canal armadilha com mensagem de aviso e contador **Kicks**.
- Apaga qualquer mensagem ou anexo enviado por humano naquele canal.
- Aplica **timeout** (castigo) configurável de 1 a 672 horas; padrão **168 h (7 dias)**.
- Log em Components V2 com usuário, ID, motivo, duração, data, conteúdo (opcional) e total de kicks.
- Botão **Remover castigo** no log, para falsos positivos.
- Deduplicação de 60 s: rajadas apagam todas as mensagens, mas punem/logam uma vez.
- Ignora bots, webhooks e mensagens de sistema.
- Cria automaticamente o canal privado `#antispam-logs` se nenhum for informado.
- Atualiza o contador no canal armadilha com agrupamento de punições.

---

## Comandos

Os comandos são **slash commands** e exigem, por padrão, **Gerenciar servidor**.

| Comando | O que faz |
|---|---|
| `/verify canal:<canal> cargo:<cargo>` | Envia o painel de verificação e define o cargo entregue |
| `/antispam canal:<canal> [canal_logs] [duracao_horas]` | Define o canal armadilha, o canal de logs e a duração do castigo |

Rodar `/verify` ou `/antispam` novamente **reconfigura** o sistema: a mensagem antiga é apagada e uma nova é enviada. O contador de kicks é mantido.

---

## Requisitos

- Node.js **20 ou superior**.
- Um aplicativo no Discord Developer Portal.
- Permissões de administrador no servidor para configurar os canais e cargos.

---

## Instalação

### A partir do pacote

1. Extraia `Ntfd-Verify-Spam.rar` com a senha fornecida pelo autor.
2. Entre na pasta `Ntfd-Verify-Spam-Source`.
3. Instale as dependências e configure o `.env`:

```bash
cd Ntfd-Verify-Spam-Source
npm install
copy .env.example .env    # Windows
# cp .env.example .env    # Linux/macOS
```

Edite o `.env` e preencha o `DISCORD_TOKEN`.

```bash
npm start
```

O bot registra os slash commands sozinho ao iniciar. Para os comandos aparecerem na hora, preencha `DEV_GUILD_ID` com o ID do seu servidor.

### Convite do bot

Troque `SEU_CLIENT_ID` pelo Application ID:

```text
https://discord.com/oauth2/authorize?client_id=SEU_CLIENT_ID&permissions=1099780156432&scope=bot%20applications.commands
```

Permissões incluídas: Ver canais, Enviar mensagens, Inserir links, Ler histórico, Gerenciar mensagens, Gerenciar cargos, Gerenciar canais e Castigar membros.

---

## Configuração do `.env`

| Variável | Obrigatória | Descrição |
|---|---|---|
| `DISCORD_TOKEN` | Sim | Token do bot |
| `DEV_GUILD_ID` | Não | Registra os comandos na hora nesse servidor |
| `BOT_LANGUAGE` | Não | `en` ou `pt` (padrão `en`) |
| `ENABLE_MESSAGE_CONTENT` | Não | `true` para gravar o texto das mensagens no log; exige a intent no portal |

> Sem `ENABLE_MESSAGE_CONTENT=true`, o log mostra apenas que houve mensagem/anexo, sem o conteúdo.

---

## Configurando a verificação

1. Crie o cargo **Verificado** (ou outro nome).
2. Arraste o **cargo do bot** para cima do cargo **Verificado** em Configurações do servidor → Cargos.
3. Permissões dos canais:
   - Canal `#verify`: `@everyone` pode **ver**, mas não escrever.
   - Outros canais: `@everyone` sem **Ver canal**; cargo **Verificado** com **Ver canal**.
4. Rode `/verify` escolhendo o canal e o cargo.

O bot valida:
- O cargo não pode ser `@everyone`.
- O cargo não pode ser gerenciado por integração.
- O cargo do bot precisa estar acima do cargo entregue.
- O bot precisa de Ver canal, Enviar mensagens e Inserir links no canal.

---

## Configurando o anti-spam

1. Crie um canal armadilha (ex.: `#nao-fale-aqui`).
2. Deixe o canal **visível e com permissão de escrita** para quem você quer proteger.
3. Rode `/antispam`:
   - `canal`: o canal armadilha.
   - `canal_logs`: opcional; se vazio, o bot cria/reutiliza `#antispam-logs`.
   - `duracao_horas`: 1 a 672; padrão 168.
4. Libere o `#antispam-logs` para sua equipe de moderação.

O bot precisa de:
- Ver canal, Enviar mensagens, Inserir links e Ler histórico no canal armadilha.
- Gerenciar mensagens para apagar o spam.
- Castigar membros no servidor.
- Ver canal, Enviar mensagens e Inserir links no canal de logs.
- Gerenciar canais apenas se for criar o `#antispam-logs`.

---

## Estrutura do projeto

```text
discord-bot/
├─ src/
│  ├─ index.js                   Inicia o bot e registra os comandos
│  ├─ config.js                  Lê o .env
│  ├─ storage.js                 Persistência em data/db.json
│  ├─ messages.js                Textos em inglês e português
│  ├─ ui.js                      Components V2
│  ├─ commands/
│  │  ├─ index.js                Coleção e JSON dos comandos
│  │  ├─ verify.js               /verify
│  │  └─ antispam.js             /antispam
│  ├─ events/
│  │  ├─ interactionCreate.js    Slash commands, botões e modais
│  │  └─ messageCreate.js        Detecta mensagens no canal armadilha
│  └─ services/
│     ├─ verification.js         Captcha, tentativas e entrega do cargo
│     ├─ captcha.js              Imagem ou matemático
│     └─ antispam.js             Timeout, logs, contador e remoção de castigo
├─ data/                         Banco JSON local (não versionar)
├─ .env.example
├─ package.json
└─ README.md
```

O mapa mental completo está em [`MapaMentalNtfd.md`](MapaMentalNtfd.md).

---

## Personalização

- **Idioma:** `BOT_LANGUAGE=pt` ou `en`; os textos ficam em `src/messages.js`.
- **Textos do log:** `buildLogMessage` em `src/services/antispam.js`.
- **Conteúdo no log:** ative a Message Content Intent e `ENABLE_MESSAGE_CONTENT=true`.
- **Permissão dos comandos:** Configurações do servidor → Integrações.
- **Captcha:** o tempo e as tentativas ficam em `src/services/verification.js` (`TTL_MS`, `MAX_ATTEMPTS`).

---

## Deploy

### PM2

```bash
npm install -g pm2
pm2 start src/index.js --name ntfd-verify-spam
pm2 save
pm2 startup
```

### Hospedagem

- Node.js 20+.
- `MAIN=src/index.js`.
- Não precisa de porta HTTP: é um bot de gateway.
- `@napi-rs/canvas` é opcional; sem ele o captcha matemático funciona.
- Em Linux, para captcha em imagem: `sudo apt install fontconfig fonts-dejavu-core`.
- O arquivo `data/db.json` precisa ser gravável e persistente entre reinícios.

---

## Segurança

- Nunca envie `.env`, `data/`, logs ou `node_modules` para o Git.
- O `.env` contém o token do bot; se vazar, resete no Developer Portal.
- `data/db.json` contém IDs de servidores, canais, cargos e contadores; não é secreto, mas é dado local.
- O bot não precisa de nenhuma privileged intent por padrão.
- `Message Content Intent` só é necessária se você quiser gravar o texto do spammer.
- O botão **Remover castigo** exige a permissão **Castigar membros**.

---

## Problemas comuns

| Problema | Solução |
|---|---|
| Comandos não aparecem | Aguarde alguns minutos ou use `DEV_GUILD_ID`; confirme o escopo `applications.commands` |
| "O cargo do bot precisa estar acima..." | Arraste o cargo do bot acima do cargo de verificado |
| Captcha aparece como conta (`4 + 7`) | Falta `@napi-rs/canvas` ou fontes no sistema; o captcha matemático funciona |
| Castigo falha | O Discord não permite castigar donos, administradores ou cargos acima do bot |
| Log não chega | Confira Ver canal, Enviar mensagens e Inserir links no canal de logs |
| Contador não atualiza | O bot precisa de Ler histórico e a mensagem do canal armadilha não pode ter sido apagada |
| Verificação não libera o servidor | O cargo precisa estar acima do cargo do bot e os outros canais devem liberar o cargo verificado |

---

## Licença

Proprietária — **todos os direitos reservados**. O código, a documentação e os assets não podem ser usados, copiados, modificados ou redistribuídos sem autorização por escrito. Veja [`LICENSE`](LICENSE).

As dependências de terceiros mantêm suas próprias licenças.

---

## Documentação

- [`MapaMentalNtfd.md`](MapaMentalNtfd.md) — arquitetura, fluxos e mapa mental.
- [`SECURITY.md`](SECURITY.md) — segredos, permissões e boas práticas.
