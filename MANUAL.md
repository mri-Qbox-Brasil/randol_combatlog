# randol_combatlog — Manual

Cria um clone fantasma do jogador que cai do servidor, segurando uma placa de identificação com nome, ID e motivo da desconexão — útil para provar combat log.

---

## Sumário

1. [Dependências](#dependências)
2. [Instalação](#instalação)
3. [Configuração](#configuração)
4. [Comportamento](#comportamento)
5. [Integrações](#integrações)
6. [Estrutura de arquivos](#estrutura-de-arquivos)

---

## Dependências

| Recurso | Obrigatório | Observação |
|---|---|---|
| `qb-core` **ou** `es_extended` | Sim | Bridge automático em `bridge/` |
| `ox_lib` | Sim | `lib.load`, `lib.requestModel`, `lib.requestScaleformMovie`, `lib.registerContext`, `lib.setClipboard`, `lib.notify` |
| `oxmysql` | Sim | Declarado no `fxmanifest.lua`. Lê a aparência do personagem no banco |
| `qb-target` | Sim | O clone é alvo de target (`exports['qb-target']`) |
| `illenium-appearance` | Sim | `exports['illenium-appearance']:setPedAppearance` aplica a skin no clone |

O bridge QB lê a skin de `playerskins` (`citizenid`, `active = 1`); o bridge ESX lê a coluna `skin` da tabela `users`.

---

## Instalação

1. Copie a pasta `randol_combatlog` para `resources/`.
2. Adicione ao `server.cfg`:
   ```
   ensure randol_combatlog
   ```
3. Não há SQL próprio a importar — o recurso apenas lê a tabela de skins já existente do seu servidor.
4. Não há itens de inventário a cadastrar.

---

## Configuração

`sv_config.lua` traz apenas o mapeamento de motivos de desconexão. O motivo bruto do FiveM é convertido em texto amigável: o recurso procura a chave (em minúsculas) dentro do motivo original e usa o valor correspondente. Sem correspondência, o motivo exibido é `Desconhecido`.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `ReasonList` | tabela `[trecho] = rótulo` | Sim | Traduz o motivo do drop. Padrões: `game crashed` → `Crash`, `timed out` → `Timeout`, `exiting` → `F8 Quit`, `you were kicked for being afk` → `Kickado por ficar AFK`, `banned` → `Banido`, `exploit` → `Explorando`, `kicked` → `Kickado` |

Valores fixos no código (não configuráveis sem editar os arquivos): raio de 100 metros para escolher quem vê o clone, 60 segundos de vida do clone e alpha 180 do ped.

---

## Comportamento

1. Ao logar, o servidor cacheia identificador, nome e license do personagem.
2. No `playerDropped`, ele busca a skin ativa do personagem. Sem skin no banco, nada acontece.
3. O clone é enviado apenas para os jogadores num raio de **100 metros** da posição em que o jogador caiu.
4. No cliente, o clone é um ped congelado, invencível, sem colisão e com alpha 180, tocando a animação de criação de personagem, com uma placa (`prop_police_id_board` + `prop_police_id_text`) anexada à mão. O texto da placa é desenhado com o scaleform `mugshot_board_01` e mostra nome, "Desconectado." e o motivo.
5. Opção de target **"Ver informações"** — abre um contexto com nome e ID; ao selecionar, copia para a área de transferência o bloco com nome do personagem, ID, license e motivo.
6. O clone e os props são deletados automaticamente após **60 segundos**, e também no stop do recurso ou no logout do personagem.

---

## Integrações

### illenium-appearance

A skin salva no banco é aplicada no clone via `exports['illenium-appearance']:setPedAppearance(ped, skin)`. É o que faz o clone parecer o personagem que caiu.

### txAdmin

O handler `txAdmin:events:serverShuttingDown` limpa o cache de jogadores, evitando gerar clones em massa quando o servidor é desligado ou reiniciado pelo txAdmin.

---

## Estrutura de arquivos

```
randol_combatlog/
├── bridge/
│   ├── client/
│   │   ├── esx.lua        — notificação e estado de login no ESX
│   │   └── qb.lua         — notificação, estado de login e limpeza no unload (QB)
│   └── server/
│       ├── esx.lua        — dados do personagem e skin (tabela users)
│       └── qb.lua         — dados do personagem e skin (tabela playerskins)
├── cl_dropped.lua         — cria o clone, a placa, o scaleform e o target
├── sv_dropped.lua         — cache de personagens, playerDropped, broadcast por proximidade
├── sv_config.lua          — tradução dos motivos de desconexão
└── fxmanifest.lua
```
