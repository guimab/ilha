# Ilha — protótipo Three.js

Protótipo de jogo 3D no navegador, inspirado no Fazendilha (app.fazendilha.com):
uma ilha tropical com uma fazendeira controlada por clique, coleta de cocos e plantio de palmeiras.

Página única, sem build e sem backend.

## Como rodar

O `importmap` e o carregamento de `.glb` exigem um servidor HTTP (abrir via `file://` não funciona).

```bash
cd ~/ilha
python3 -m http.server 8765
# abrir http://localhost:8765
```

### Parâmetros de URL

| Parâmetro      | Efeito                                                      |
|----------------|-------------------------------------------------------------|
| (nenhum)       | Qualidade alta (padrão)                                     |
| `?q=low`       | Qualidade baixa, para máquinas fracas                       |
| `?hero=hero`   | Usa o fazendeiro modelado do zero (`models/hero.glb`)       |
| `?hero=<nome>` | Carrega `models/<nome>.glb` (padrão: `fazendeira`)          |

Parâmetros podem ser combinados: `?q=low&hero=hero`.

## Controles

| Ação                         | Mouse / teclado                    | Touch      |
|------------------------------|------------------------------------|------------|
| Andar até um ponto           | Clique no chão                     | Toque      |
| Colher coco                  | Clique numa palmeira               | Toque      |
| Modo Andar / Plantar         | Teclas `1` / `2` ou botões da barra | Botões     |
| Plantar palmeira (custa 1 🥥) | Modo Plantar + clique no chão      | Toque      |
| Arrastar o mapa              | Botão esquerdo + arrastar          | 1 dedo     |
| Girar a câmera (só horizontal)| Botão direito + arrastar          | 2 dedos    |
| Zoom                         | Scroll                             | Pinça      |
| Centralizar no personagem    | Tecla `C` ou botão 📍              | Botão      |

Regras: cocos voltam à palmeira em 20 s; não dá para andar na água; palmeiras novas precisam de distância mínima de outras.

## Estrutura

```
ilha/
├── index.html              # jogo inteiro (HTML + CSS + JS module)
├── README.md
└── models/
    ├── fazendeira.glb      # personagem padrão (usado pelo jogo)
    ├── woman_src.glb       # modelo original baixado (fonte, não usado em runtime)
    ├── hero.glb            # fazendeiro low-poly feito do zero via Blender MCP
    └── hero.blend          # cópia do arquivo Blender (cenas "Hero" e "Fazendeira")
```

## Stack

- **Three.js 0.160.0** via `importmap` (unpkg), sem bundler.
- `OrbitControls` (pan/rotate/zoom com rotação vertical travada) e `GLTFLoader`.
- `AnimationMixer` com crossfade entre clipes `Idle` / `Walk` / `Action`.
- Terreno procedural (`PlaneGeometry` + ruído) com contorno irregular, raio ~44.
- Mar em shader (`onBeforeCompile`): ondas animadas por `uTime` e 6 tons de azul por profundidade
  (profundidade pré-calculada em `DataTexture`).
- Palmeiras, pedras e nuvens procedurais; sombras via `DirectionalLight` (8192 px em HQ).

### Qualidade

| Item         | HQ (padrão) | `?q=low` |
|--------------|-------------|----------|
| pixelRatio   | até 3       | 1        |
| Shadow map   | 8192        | 2048     |
| Terreno      | 480 seg.    | 240 seg. |
| Mar          | 400 seg.    | 180 seg. |
| Textura prof.| 1024        | 256      |

## Personagem

O jogo carrega `models/${hero}.glb`. Se falhar, mostra o boneco procedural antigo (fallback).

Mapeamento de animações no código:

- Nomes `Armature|Clip` têm o prefixo removido.
- `Idle` ← `Idle_Neutral` quando existir (o `Idle` original da fazendeira torce um pé).
- `Action` ← `Interact` quando não houver `Action`.
- Usados no jogo: `Idle` (parado), `Walk` (andando), `Action` (colher/plantar, uma vez).

### Trocar ou criar personagem

1. Modelo rigado em Blender (ou baixado), virado para **-Y** no Blender (= +Z no glTF).
2. Animações com nomes `Idle`, `Walk`, `Action` (ou `Interact`).
3. Exportar glTF Binary (`.glb`) com *Selected Objects*, skins e animações
   (modo *Actions* ou *NLA Tracks*).
4. Salvar em `models/<nome>.glb` e abrir `?hero=<nome>`.

### Blender MCP

O Blender está conectado ao Hermes via MCP (`mcp-for-blender==2.2.6`, addon na porta 9876),
o que permite modelar, editar e exportar por comando. Com o Blender aberto e o addon ativo,
basta pedir (ex.: "re-exporta a fazendeira").

## Créditos e licenças

- **Fazendeira**: "Animated Woman" — Quaternius, via Poly Pizza
  (https://poly.pizza/m/qJ2gsTUBHL). Licença **CC0** (domínio público).
  Modificações: cores, chapéu de palha, cabelo preto médio.
- **Fazendeiro (`hero.glb`)**: criado do zero para este projeto.
- **Three.js**: licença MIT.

## Limitações conhecidas

- Sem persistência: cocos e palmeiras plantadas somem ao recarregar.
- Sem backend, contas ou multiplayer.
- `fazendeira.glb` (~1,9 MB) inclui animações não usadas (tiro, espada); dá para remover e reduzir para ~600 KB.
- Depende de CDN (unpkg) para o Three.js; offline não carrega.

## Próximos passos sugeridos

- Salvar progresso em `localStorage`.
- Itens na mão (cesta, enxada), NPCs e animais.
- Migrar para Vite + TypeScript quando o código crescer; backend (ex.: Hono/Node) para contas e progresso, no estilo do Fazendilha.
