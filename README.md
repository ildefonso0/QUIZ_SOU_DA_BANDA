# 🇦🇴 QUIZ SOU DA BANDA — README (Detalhado)

**Autor:** Joaquim Ildefonso  
**Repositório:** `ildefonso0/QUIZ_SOU_DA_BANDA`  
**Última versão deste README:** 2025-10-24

---

## 🙌 Visão geral
Este repositório funciona como **base de dados** para a aplicação **Quiz Sou da Banda**.  
A app **lê** (fetch) ficheiros JSON e imagens do GitHub (raw) e guarda uma cópia local para modo offline. A sincronização com o repositório acontece **automaticamente a cada 24 horas** (ou quando a app detetar uma nova versão no `config.json`).

---

## 🗂 Estrutura do repositório
QUIZ_SOU_DA_BANDA/
├── data/
│ ├── config.json # meta/informações de versão e tema
│ ├── categorias.json # lista de categorias
│ └── quizzes.json # perguntas (todos os tipos)
├── imagens/
│ ├── historia/
│ ├── geografia/
│ └── cultura/
└── README.md

markdown
Copiar código

---

## 🔁 Fluxo completo do processo (passo-a-passo)

1. **Deploy do conteúdo no GitHub**
   - Coloca/edita `data/config.json`, `data/categorias.json`, `data/quizzes.json` e as imagens correspondentes nas pastas `imagens/...`.
   - Faz `commit` e `push` para o branch principal (`main`).

2. **App inicia (primeira execução)**
   - A app faz `fetch()` ao `config.json` + `quizzes.json` + `categorias.json` a partir dos URLs `raw.githubusercontent.com`.
   - Guarda os dados obtidos localmente (cache) usando `IndexedDB` (recomendado) ou `localStorage`.

3. **Verificação periódica**
   - Um temporizador interno (ou job) verifica o `config.json` remoto a cada 24 horas.
   - Se `remote.versao !== local.versao` **ou** `remote.ultima_atualizacao` for posterior, a app baixa os ficheiros atualizados e substitui o cache local.

4. **Sincronização incremental (opcional)**
   - Se quiseres poupar dados, podes comparar `ETag` ou `last-modified` dos ficheiros raw do GitHub e baixar só ficheiros alterados.

5. **Modo offline**
   - Se não houver rede, a app usa o cache local.
   - Quando a rede volta, verifica o `config.json` remoto e sincroniza se necessário.

6. **Atualização de conteúdo pelo maintainer**
   - Qualquer alteração no repositório (adicionar perguntas, corrigir respostas, trocar imagens) só requer `commit -> push` no GitHub. A app detecta automaticamente na próxima verificação.

---

## 📁 Exemplo de ficheiros (para copiar/colar)

### `data/config.json`
```json
{
  "versao": "1.0.0",
  "ultima_atualizacao": "2025-10-24T00:00:00Z",
  "intervalo_atualizacao_horas": 24,
  "tema": {
    "cor_primaria": "#2A8BF2",
    "cor_secundaria": "#F2A22A",
    "modo": "claro"
  },
  "app": {
    "nome": "Quiz Sou da Banda",
    "descricao": "Aplicativo educativo sobre Angola",
    "autor": "Joaquim Ildefonso"
  }
}
data/categorias.json
json
Copiar código
[
  { "id": 1, "nome": "História", "descricao": "Eventos históricos de Angola", "icone": "https://raw.githubusercontent.com/ildefonso0/QUIZ_SOU_DA_BANDA/main/imagens/historia/1.png" },
  { "id": 2, "nome": "Geografia", "descricao": "Cidades e rios", "icone": "https://raw.githubusercontent.com/ildefonso0/QUIZ_SOU_DA_BANDA/main/imagens/geografia/1.png" },
  { "id": 3, "nome": "Cultura", "descricao": "Dança, música e tradições", "icone": "https://raw.githubusercontent.com/ildefonso0/QUIZ_SOU_DA_BANDA/main/imagens/cultura/1.png" }
]
data/quizzes.json (exemplos variados)
json
Copiar código
[
  {
    "id": 101,
    "categoria": "História",
    "tipo": "multipla_escolha",
    "pergunta": "Em que ano Angola conquistou a sua independência?",
    "opcoes": ["1973", "1974", "1975", "1976"],
    "resposta_correta": 2,
    "imagem": "https://raw.githubusercontent.com/ildefonso0/QUIZ_SOU_DA_BANDA/main/imagens/historia/1.png",
    "nivel": "fácil"
  },
  {
    "id": 102,
    "categoria": "Geografia",
    "tipo": "verdadeiro_falso",
    "pergunta": "O rio Kwanza é o maior rio inteiramente dentro de Angola.",
    "resposta_correta": true,
    "nivel": "médio"
  },
  {
    "id": 103,
    "categoria": "Cultura",
    "tipo": "imagem_para_texto",
    "imagem": "https://raw.githubusercontent.com/ildefonso0/QUIZ_SOU_DA_BANDA/main/imagens/cultura/1.png",
    "pergunta": "Qual é o nome desta dança tradicional?",
    "resposta_correta": "Kizomba",
    "nivel": "fácil"
  },
  {
    "id": 104,
    "categoria": "História",
    "tipo": "completar_frase",
    "pergunta": "O dia da independência de Angola é celebrado a __ de Novembro.",
    "resposta_correta": "11",
    "nivel": "médio"
  }
]
💻 Exemplo prático: Script TypeScript (client-side) para verificar/baixar atualizações
Este snippet pretende ser copiado para a app (Web / React / React Native via fetch). Usa IndexedDB ou localStorage conforme preferires. Aqui uso localStorage por simplicidade; em produção recomendo IndexedDB.

ts
Copiar código
// updater.ts (exemplo simplificado)
const BASE = "https://raw.githubusercontent.com/ildefonso0/QUIZ_SOU_DA_BANDA/main/data";
const URL_CONFIG = `${BASE}/config.json`;
const URL_QUIZZES = `${BASE}/quizzes.json`;
const URL_CATS = `${BASE}/categorias.json`;

async function fetchJson(url: string) {
  const res = await fetch(url, { cache: "no-store" });
  if (!res.ok) throw new Error(`Fetch failed: ${url}`);
  return res.json();
}

function saveLocal(key: string, data: any) {
  localStorage.setItem(key, JSON.stringify(data));
}

function loadLocal(key: string) {
  const s = localStorage.getItem(key);
  return s ? JSON.parse(s) : null;
}

export async function verificarAtualizacaoForcada() {
  try {
    const remotoConfig = await fetchJson(URL_CONFIG);
    const localConfig = loadLocal("config");

    // Compara versão ou timestamp
    const precisaAtualizar =
      !localConfig ||
      remotoConfig.versao !== localConfig.versao ||
      new Date(remotoConfig.ultima_atualizacao) > new Date(localConfig.ultima_atualizacao);

    if (precisaAtualizar) {
      console.log("Atualização detectada — baixando dados...");
      const [quizzes, categorias] = await Promise.all([fetchJson(URL_QUIZZES), fetchJson(URL_CATS)]);
      saveLocal("config", remotoConfig);
      saveLocal("quizzes", quizzes);
      saveLocal("categorias", categorias);
      console.log("Dados atualizados com sucesso.");
      // Aqui podes emitir um evento/notification para UI
    } else {
      console.log("Nada novo no repositório.");
    }
  } catch (err) {
    console.error("Erro ao verificar atualização:", err);
  }
}

// Agendar verificação a cada 24h (24 * 60 * 60 * 1000 ms)
setInterval(verificarAtualizacaoForcada, 24 * 60 * 60 * 1000);
// E correr imediatamente à inicialização
verificarAtualizacaoForcada();
⚙️ Boas práticas técnicas e recomendações
Cache: Em produção usa IndexedDB (maior capacidade e estruturas complexas). localStorage é simples, mas limitado.

ETag/Last-Modified: Para reduzir downloads usa cabeçalhos If-None-Match / If-Modified-Since (fetch com HEAD) — o GitHub Raw dá suporte a ETag.

Atomicidade: Faz download para um objecto temporário e só depois substitui o cache actual para evitar corrupção se o download falhar.

Backups: Mantém uma cópia de segurança da versão antiga até ter sucesso na nova atualização.

Versionamento semântico (semver): Usa MAJOR.MINOR.PATCH em config.json.versao para mudanças grandes/pequenas/correções.

Testes locais: Antes de push, valida quizzes.json com um script que verifica campos obrigatórios (id, tipo, pergunta, resposta).

Imagens: Evita nomes iguais (1.png) em pastas diferentes; usa caminhos descritivos (historia_independencia_1975.png).

🔐 Privacidade e segurança
Os ficheiros no repositório são públicos se estiveres a usar raw.githubusercontent.com sem autenticação.

Não coloques segredos (tokens, chaves) em ficheiros públicos.

Se precisares de repositório privado, usa GitHub Personal Access Token (PAT) e passa-o na chamada fetch (server-side) ou usa um backend seguro—NÃO coloques o PAT no frontend.

🧰 Ferramentas e automações recomendadas
Pre-commit hooks (ex.: husky) para validar JSON.

GitHub Actions para:

Gerar previews automáticos,

Executar validações JSON,

Publicar assets (ex.: otimizar imagens).

CI checks: scripts que confirmem que config.json.versao aumentou se houver mudanças em quizzes.json.

📝 Como editar/atualizar conteúdo (manual)
Clona o repo:

bash
Copiar código
git clone https://github.com/ildefonso0/QUIZ_SOU_DA_BANDA.git
Edita data/quizzes.json (adiciona/remova/actualiza objectos).

Adiciona imagens em imagens/<categoria>/.

Atualiza data/config.json incrementando versao e definindo ultima_atualizacao para a data atual em ISO (ex.: 2025-10-24T13:00:00Z).

Commit & push:

bash
Copiar código
git add .
git commit -m "Atualiza quizzes: +10 perguntas; versao 1.0.1"
git push origin main
❗ Possíveis limitações / armadilhas
Rate limits GitHub: Para uso muito intensivo, tens limites de requests. Para grande escala, usa um cache server ou CDN.

URLs Raw públicas: Se mudares o nome do repositório/usuário, atualiza os links nas apps.

Privacidade dos dados do utilizador: Se começares a gravar scores online, precisas de um backend e autenticação segura.

✅ Resumo Final
O repositório é a única fonte de verdade (single source of truth) para os conteúdos do quiz.

A app puxa ficheiros raw do GitHub, armazena em cache e verifica atualizações a cada 24h, atualizando automaticamente quando config.json muda.

Fácil de manter: editar ficheiros no repo e fazer push é suficiente para publicar novo conteúdo.

Se quiseres, eu posso:

Gerar um script Node.js para validar automaticamente o quizzes.json (verificar campos, tipos e IDs únicos) e adicioná-lo como GitHub Action.

Criar o exemplo de código React/React Native com IndexedDB e UI para mostrar quizzes.

Gerar o ficheiro prettier / husky / lint para garantir qualidade antes do commit.

Quer que eu gere já o validator (GitHub Action) para garantir que os JSONs são válidos antes de fazer merge?

Copiar código






