> ⚠️ **Novo Desenvolvedor?** [Leia o Guia de Handover (Transferência) aqui antes de começar!](docs/HANDOVER.md)

# 🏥 e-PET: Prontuário e Dashboard Web

Interface web focada no acesso e gestão de saúde pelas equipes multidisciplinares e médicas nas Unidades Básicas de Saúde. O e-PET funciona como um Prontuário Eletrônico (Pec-SUS) enriquecido, provendo inteligência e visualização da estratificação de risco calculada para facilitar o planejamento clínico.

## 🌟 O que o sistema resolve?
*   Exibe de forma consolidada e intuitiva os dados coletados presencialmente no domicílio pelo aplicativo móvel (e-ACS).
*   Destaca a Estratificação de Risco Familiar e Individual (baseada na Metodologia Coelho-Savassi) gerada de forma automática pelo motor (e-TET).
*   Auxilia no direcionamento proativo do cuidado da saúde primária, permitindo que a gestão identifique e priorize as vulnerabilidades críticas no território em tempo real.

## 🛠️ Tecnologias Principais
*   **Framework:** Vue 3 + Vite
*   **Design System / UI:** Vuetify 3
*   **State Management:** Pinia
*   **Linguagem:** TypeScript
*   **Internacionalização:** Vue I18n

## 📦 Instalação e Execução

Pré-requisitos: Node.js

```bash
# 1. Instalar dependências
npm install

# 2. Iniciar servidor em ambiente local
npm run dev

# 3. Empacotar para produção
npm run build
```

## 🤖 Antigravity / Inteligência Artificial

As regras de contexto, stack e diretrizes estritas para a atuação dos agentes de inteligência artificial neste repositório estão documentadas no arquivo [AGENTS.md](AGENTS.md) localizado na raiz.
