# 🦉 LinguaQuest

App de aprendizado de idiomas (**React + TypeScript**) inspirado no Duolingo, com sistema de lições, XP, streaks, conquistas e leaderboard — acompanhado de uma suíte de QA própria cobrindo testes unitários, de integração, E2E e manuais.

---

## 📖 Sobre o Projeto

LinguaQuest simula uma experiência de aprendizado gamificada: o usuário avança por unidades e lições, responde perguntas de múltipla escolha, tradução e preenchimento de lacunas, ganha XP e gemas, mantém uma sequência (streak) de dias, sobe de nível, desbloqueia conquistas e acompanha sua posição em um leaderboard.

O foco principal deste repositório é servir como **projeto de portfólio de QA**: o app funcional acompanha um plano de testes completo (`QA_TEST_SUITE.md`) pensado para ser executado com Jest, React Testing Library e Playwright.

---

## 🛠 Tecnologias

| Camada | Tecnologia |
|---|---|
| Frontend | React 19 + TypeScript |
| Estilo | Tailwind CSS |
| Estado global | Context API + `useReducer` |
| Persistência | `localStorage` |
| Testes unitários/integração | Jest + React Testing Library |
| Testes E2E | Playwright |

---

## 📁 Estrutura do Projeto

```
LinguaQuest/
├── src/
│   ├── data/courses.ts        → Dados de cursos, lições e conquistas
│   ├── store/index.tsx        → Estado global (useStore + reducer)
│   ├── types/index.ts         → Interfaces TypeScript
│   └── pages/
│       ├── HomePage.tsx       → Mapa de unidades e navegação
│       ├── GamePage.tsx       → Gameplay da lição
│       ├── ResultPage.tsx     → Tela de resultado (score + XP)
│       ├── ProfilePage.tsx    → Progresso do usuário e conquistas
│       └── LeaderboardPage.tsx → Ranking
├── linguaquest.html           → Build compilado (single-file) do app
├── QA_TEST_SUITE.md           → Suíte de testes completa (unit/integration/E2E/manual)
└── README.md
```

---

## ✨ Funcionalidades

- **Sistema de lições** organizadas em unidades progressivas, com desbloqueio por XP
- **Tipos de pergunta**: múltipla escolha, tradução, preenchimento de lacunas
- **Gamificação**: XP, níveis, gemas, vidas (lives) e streak diário
- **Conquistas** desbloqueáveis com base no progresso do usuário
- **Leaderboard** comparando o usuário com outros perfis
- **Persistência local** do progresso via `localStorage`
- **Reset de progresso** disponível na tela de perfil

---

## ▶️ Como Executar

```bash
# Instalar dependências
npm install

# Rodar em modo desenvolvimento
npm run dev
```

Abra `linguaquest.html` diretamente no navegador para visualizar a build já compilada, sem precisar rodar o projeto localmente.

---

## ✅ Suíte de Testes (QA_TEST_SUITE.md)

O arquivo [`QA_TEST_SUITE.md`](./QA_TEST_SUITE.md) contém o plano de testes completo do projeto, com **46 casos de teste** no total:

| Camada | Ferramenta | Quantidade |
|---|---|---|
| Unitários | Jest | 12 |
| Integração | React Testing Library | 9 |
| E2E | Playwright | 10 |
| Manual (checklist exploratório) | — | 15 |
| **Total** | | **46** |

### O que é coberto

- **Unitários**: lógica do reducer de estado (`COMPLETE_LESSON`, `LOSE_LIFE`, `ADD_XP`, `UNLOCK_ACHIEVEMENT`, `RESET_PROGRESS`) e integridade dos dados de curso (`courses.ts`)
- **Integração**: renderização e interação da tela de jogo (`GamePage`) — perguntas, opções de resposta, feedback visual de acerto/erro
- **E2E**: fluxos completos com Playwright, incluindo conclusão de lição e persistência do progresso após reload da página
- **Manual**: checklist exploratório cobrindo responsividade, gamificação (vidas, XP, streak), perfil, leaderboard, persistência, animações, navegação e acessibilidade

### Setup dos testes

```bash
npm install
npm install --save-dev jest @testing-library/react @testing-library/jest-dom @playwright/test ts-jest
npx playwright install
```

### Rodar testes unitários e de integração

```bash
npx jest --coverage
```

### Rodar testes E2E

```bash
# Inicie o servidor de desenvolvimento primeiro:
npm run dev

# Em outro terminal:
npx playwright test
```

### Gerar relatório HTML dos testes E2E

```bash
npx playwright show-report
```

---

## 🐛 Template de Bug Report

O `QA_TEST_SUITE.md` também inclui um template padrão para reportar bugs encontrados durante os testes exploratórios, com campos para passos de reprodução, resultado esperado x atual, ambiente e evidências.

---

## 📄 Autor

Projeto de portfólio desenvolvido para demonstrar domínio em QA Automation — do desenvolvimento do app à cobertura de testes em múltiplas camadas.

[![LinkedIn](https://img.shields.io/badge/-LinkedIn-0077B5?style=flat-square&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/rodrigo-sousa-qa/)
[![GitHub](https://img.shields.io/badge/-GitHub-181717?style=flat-square&logo=github&logoColor=white)](https://github.com/rodrigodsss)
