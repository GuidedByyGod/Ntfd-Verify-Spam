# Segurança

## Segredos

Nunca versione:

- `.env` (token do bot)
- `data/db.json` e backups (`data/`)
- logs
- `node_modules/`

O `.gitignore` já cobre esses arquivos. Se o token for exposto em um commit,
print, issue ou arquivo compactado, **resete imediatamente** no Discord
Developer Portal.

## Permissões

O bot precisa apenas do necessário para moderação:

- Ver canais
- Enviar mensagens
- Inserir links
- Ler histórico
- Gerenciar mensagens
- Gerenciar cargos
- Gerenciar canais (opcional, para criar `#antispam-logs`)
- Castigar membros

Não dê Administrador ao bot. Mantenha o cargo do bot acima dos cargos que ele
precisa entregar ou punir.

## Intents

- `Guilds` e `GuildMessages` são suficientes para o funcionamento padrão.
- `MessageContent` é opcional e só deve ser ativada se você quiser gravar o
  conteúdo das mensagens no log do anti-spam.

## Dados locais

`data/db.json` guarda IDs de servidores, canais, cargos, mensagens e o contador
de kicks. Faça backup se quiser preservar as configurações, mas não publique o
arquivo.

## Anti-spam

- O canal armadilha pune qualquer humano que escrever nele, inclusive por
  engano. Use o botão **Remover castigo** para reverter falsos positivos.
- O Discord não permite castigar donos, administradores ou membros com cargo
  acima do bot.
- O bot ignora bots, webhooks e mensagens de sistema.
