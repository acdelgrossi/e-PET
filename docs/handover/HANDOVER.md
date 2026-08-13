# 🤝 Guia do Desenvolvedor (Handover) - e-PET

Bem-vindo ao projeto **e-PET** (Prontuário Web)! Este documento resume o status da aplicação para facilitar sua continuidade.

## 1. O que já está implementado?
A casca da interface e as visualizações clínicas de território já estão criadas dentro de `src/pages/` e `src/components/`:
- **Login:** Autenticação no sistema (`LoginView.vue`).
- **Dashboard:** Tela inicial com painel geral e estatísticas (`DashboardView.vue`).
- **Território:** Listagem e filtros das famílias de uma microárea (`TerritoryListView.vue`).
- **Painel Clínico:** Detalhamento do cidadão com foco total em visualização de saúde (`VisualizacaoCidadao.vue`).

## 2. Padrões de Código e Arquitetura Local
- **UI Framework (Vuetify 3):** Toda a estruturação visual é baseada no sistema de grids e componentes do Vuetify. Evite escrever CSS puro se houver um componente oficial para o trabalho.
- **Consumo de API:** A aplicação atua estritamente como *Consumer* da API Central (e-TET-api). As lógicas de "Cálculo de Risco" (Coelho-Savassi) NÃO ficam aqui no Front, o e-PET apenas recebe a resposta (ex: `riskClass: R2`) da API e estiliza a página.
- **Padrão Vue:** Utilizamos o ecossistema moderno do Vue 3 (`<script setup lang="ts">`) + Vite.

## 3. Débitos Técnicos e Próximos Passos
O que focar agora que você assumiu o projeto:
1. **Integração de APIs de Dashboard:** Melhorar as chamadas Axios da `DashboardView` agregando filtros por competência/mês via backend.
2. **Paginação Server-Side:** Melhorar a `TerritoryListView.vue` aplicando paginação via API para evitar carregar grandes volumes de domicílios na memória do navegador.
