# LinguaQuest – Test Suite for QA Junior
# Version: 1.0.0

## ✅ Overview

This file contains a comprehensive test suite for the **LinguaQuest** application —
a gamified language learning app inspired by Duolingo.

It is organized into test cases ready to be used in tools like:
- **Jest** (unit/integration)
- **Cypress / Playwright** (E2E)
- **Manual Testing Checklist**

---

## 📁 Project Structure to Test

```
LinguaQuest/
├── src/
│   ├── data/courses.ts       → Course and achievement data
│   ├── store/index.tsx       → Global state (useStore + reducer)
│   ├── types/index.ts        → TypeScript interfaces
│   └── pages/
│       ├── HomePage.tsx      → Map + unit navigation
│       ├── GamePage.tsx      → Lesson gameplay
│       ├── ResultPage.tsx    → Score + XP result screen
│       ├── ProfilePage.tsx   → User progress + achievements
│       └── LeaderboardPage.tsx → Rankings
```

---

## 🧪 UNIT TESTS (Jest + React Testing Library)

### File: `store.test.ts`

```ts
import { reducer } from '../src/store';

const defaultProgress = {
  completedLessons: [],
  xp: 0,
  streak: 1,
  lastLoginDate: '2025-01-01',
  lives: 5,
  gems: 50,
  level: 1,
  language: 'english',
  achievements: [],
  lessonScores: {},
};

describe('Store Reducer – COMPLETE_LESSON', () => {
  test('TC-STORE-001: adds lesson to completedLessons', () => {
    const action = { type: 'COMPLETE_LESSON', lessonId: 'lesson-1-1', xp: 20, score: 80 };
    const result = reducer(defaultProgress, action);
    expect(result.completedLessons).toContain('lesson-1-1');
  });

  test('TC-STORE-002: does not duplicate completed lessons', () => {
    const state = { ...defaultProgress, completedLessons: ['lesson-1-1'] };
    const action = { type: 'COMPLETE_LESSON', lessonId: 'lesson-1-1', xp: 20, score: 80 };
    const result = reducer(state, action);
    expect(result.completedLessons.filter(l => l === 'lesson-1-1').length).toBe(1);
  });

  test('TC-STORE-003: increases XP correctly', () => {
    const action = { type: 'COMPLETE_LESSON', lessonId: 'lesson-1-2', xp: 25, score: 100 };
    const result = reducer(defaultProgress, action);
    expect(result.xp).toBe(25);
  });

  test('TC-STORE-004: saves best score (higher wins)', () => {
    const state = { ...defaultProgress, lessonScores: { 'lesson-1-1': 60 } };
    const action = { type: 'COMPLETE_LESSON', lessonId: 'lesson-1-1', xp: 20, score: 90 };
    const result = reducer(state, action);
    expect(result.lessonScores['lesson-1-1']).toBe(90);
  });

  test('TC-STORE-005: does not replace higher score with lower', () => {
    const state = { ...defaultProgress, lessonScores: { 'lesson-1-1': 95 } };
    const action = { type: 'COMPLETE_LESSON', lessonId: 'lesson-1-1', xp: 10, score: 60 };
    const result = reducer(state, action);
    expect(result.lessonScores['lesson-1-1']).toBe(95);
  });
});

describe('Store Reducer – LOSE_LIFE', () => {
  test('TC-STORE-006: decrements lives by 1', () => {
    const result = reducer(defaultProgress, { type: 'LOSE_LIFE' });
    expect(result.lives).toBe(4);
  });

  test('TC-STORE-007: lives do not go below 0', () => {
    const state = { ...defaultProgress, lives: 0 };
    const result = reducer(state, { type: 'LOSE_LIFE' });
    expect(result.lives).toBe(0);
  });
});

describe('Store Reducer – LEVEL UP', () => {
  test('TC-STORE-008: level increases after 100 XP', () => {
    const action = { type: 'COMPLETE_LESSON', lessonId: 'x', xp: 100, score: 100 };
    const result = reducer(defaultProgress, action);
    expect(result.level).toBe(2);
  });

  test('TC-STORE-009: level is correct at 200 XP', () => {
    const state = { ...defaultProgress, xp: 190 };
    const action = { type: 'ADD_XP', amount: 10 };
    const result = reducer(state, action);
    expect(result.level).toBe(3);
  });
});

describe('Store Reducer – UNLOCK_ACHIEVEMENT', () => {
  test('TC-STORE-010: adds achievement id to list', () => {
    const action = { type: 'UNLOCK_ACHIEVEMENT', id: 'first-lesson' };
    const result = reducer(defaultProgress, action);
    expect(result.achievements).toContain('first-lesson');
  });

  test('TC-STORE-011: does not duplicate achievements', () => {
    const state = { ...defaultProgress, achievements: ['first-lesson'] };
    const action = { type: 'UNLOCK_ACHIEVEMENT', id: 'first-lesson' };
    const result = reducer(state, action);
    expect(result.achievements.length).toBe(1);
  });
});

describe('Store Reducer – RESET_PROGRESS', () => {
  test('TC-STORE-012: resets all progress to defaults', () => {
    const state = { ...defaultProgress, xp: 500, completedLessons: ['a', 'b'], level: 6 };
    const result = reducer(state, { type: 'RESET_PROGRESS' });
    expect(result.xp).toBe(0);
    expect(result.completedLessons).toHaveLength(0);
    expect(result.level).toBe(1);
  });
});
```

---

### File: `courses.test.ts`

```ts
import { englishCourse, achievements } from '../src/data/courses';

describe('Course Data Integrity', () => {
  test('TC-DATA-001: englishCourse has at least 3 units', () => {
    expect(englishCourse.length).toBeGreaterThanOrEqual(3);
  });

  test('TC-DATA-002: each unit has required fields', () => {
    englishCourse.forEach(unit => {
      expect(unit).toHaveProperty('id');
      expect(unit).toHaveProperty('title');
      expect(unit).toHaveProperty('lessons');
      expect(unit).toHaveProperty('requiredXP');
      expect(unit).toHaveProperty('color');
    });
  });

  test('TC-DATA-003: all lessons have at least 3 questions', () => {
    englishCourse.forEach(unit => {
      unit.lessons.forEach(lesson => {
        expect(lesson.questions.length).toBeGreaterThanOrEqual(3);
      });
    });
  });

  test('TC-DATA-004: all questions have valid correctAnswer', () => {
    englishCourse.forEach(unit => {
      unit.lessons.forEach(lesson => {
        lesson.questions.forEach(q => {
          expect(q.correctAnswer).toBeTruthy();
        });
      });
    });
  });

  test('TC-DATA-005: correctAnswer must be one of the options', () => {
    englishCourse.forEach(unit => {
      unit.lessons.forEach(lesson => {
        lesson.questions.forEach(q => {
          if (q.options) {
            const answer = Array.isArray(q.correctAnswer) ? q.correctAnswer[0] : q.correctAnswer;
            expect(q.options).toContain(answer);
          }
        });
      });
    });
  });

  test('TC-DATA-006: no duplicate question IDs', () => {
    const ids: string[] = [];
    englishCourse.forEach(unit => {
      unit.lessons.forEach(lesson => {
        lesson.questions.forEach(q => ids.push(q.id));
      });
    });
    const unique = new Set(ids);
    expect(unique.size).toBe(ids.length);
  });

  test('TC-DATA-007: achievements have id, title, description, icon', () => {
    achievements.forEach(a => {
      expect(a.id).toBeTruthy();
      expect(a.title).toBeTruthy();
      expect(a.description).toBeTruthy();
      expect(a.icon).toBeTruthy();
      expect(typeof a.condition).toBe('function');
    });
  });

  test('TC-DATA-008: first-lesson achievement triggers after 1 completed lesson', () => {
    const achievement = achievements.find(a => a.id === 'first-lesson')!;
    const progress = { completedLessons: ['l1'], xp: 20, streak: 1, lessonScores: {} };
    expect(achievement.condition(progress as any)).toBe(true);
  });

  test('TC-DATA-009: xp-100 achievement triggers at 100+ XP', () => {
    const achievement = achievements.find(a => a.id === 'xp-100')!;
    const progress = { completedLessons: [], xp: 100, streak: 1, lessonScores: {} };
    expect(achievement.condition(progress as any)).toBe(true);
  });

  test('TC-DATA-010: perfect-score achievement triggers on 100% score', () => {
    const achievement = achievements.find(a => a.id === 'perfect-score')!;
    const progress = { completedLessons: ['l1'], xp: 0, streak: 1, lessonScores: { 'l1': 100 } };
    expect(achievement.condition(progress as any)).toBe(true);
  });
});
```

---

## 🧩 INTEGRATION TESTS (React Testing Library)

### File: `GamePage.test.tsx`

```tsx
import { render, screen, fireEvent } from '@testing-library/react';
import GamePage from '../src/pages/GamePage';
import { StoreProvider } from '../src/store';
import { englishCourse } from '../src/data/courses';

const lesson = englishCourse[0].lessons[0];
const mockOnFinish = jest.fn();
const mockOnQuit = jest.fn();

const Wrapper = ({ children }: any) => <StoreProvider>{children}</StoreProvider>;

describe('GamePage Integration Tests', () => {
  beforeEach(() => {
    mockOnFinish.mockClear();
    mockOnQuit.mockClear();
  });

  test('TC-GAME-001: renders first question prompt', () => {
    render(
      <Wrapper>
        <GamePage lesson={lesson} onFinish={mockOnFinish} onQuit={mockOnQuit} />
      </Wrapper>
    );
    expect(screen.getByText(lesson.questions[0].prompt)).toBeInTheDocument();
  });

  test('TC-GAME-002: renders all answer options', () => {
    render(
      <Wrapper>
        <GamePage lesson={lesson} onFinish={mockOnFinish} onQuit={mockOnQuit} />
      </Wrapper>
    );
    lesson.questions[0].options?.forEach(option => {
      expect(screen.getByText(option)).toBeInTheDocument();
    });
  });

  test('TC-GAME-003: clicking correct answer shows green feedback', () => {
    render(
      <Wrapper>
        <GamePage lesson={lesson} onFinish={mockOnFinish} onQuit={mockOnQuit} />
      </Wrapper>
    );
    const correctAnswer = lesson.questions[0].correctAnswer as string;
    fireEvent.click(screen.getByText(correctAnswer));
    expect(screen.getByText('🎉 Correct!')).toBeInTheDocument();
  });

  test('TC-GAME-004: clicking wrong answer shows error feedback', () => {
    render(
      <Wrapper>
        <GamePage lesson={lesson} onFinish={mockOnFinish} onQuit={mockOnQuit} />
      </Wrapper>
    );
    const wrongAnswer = lesson.questions[0].options?.find(
      o => o !== lesson.questions[0].correctAnswer
    )!;
    fireEvent.click(screen.getByText(wrongAnswer));
    expect(screen.getByText('😅 Incorrect!')).toBeInTheDocument();
  });

  test('TC-GAME-005: after answering, cannot click another answer', () => {
    render(
      <Wrapper>
        <GamePage lesson={lesson} onFinish={mockOnFinish} onQuit={mockOnQuit} />
      </Wrapper>
    );
    const correct = lesson.questions[0].correctAnswer as string;
    const wrong = lesson.questions[0].options?.find(o => o !== correct)!;
    fireEvent.click(screen.getByText(correct));
    fireEvent.click(screen.getByText(wrong));
    // Second click should be ignored – still shows correct feedback
    expect(screen.getByText('🎉 Correct!')).toBeInTheDocument();
  });

  test('TC-GAME-006: progress bar increments on Next', () => {
    render(
      <Wrapper>
        <GamePage lesson={lesson} onFinish={mockOnFinish} onQuit={mockOnQuit} />
      </Wrapper>
    );
    expect(screen.getByText('1/5')).toBeInTheDocument();
    const correct = lesson.questions[0].correctAnswer as string;
    fireEvent.click(screen.getByText(correct));
    fireEvent.click(screen.getByText('Next →'));
    expect(screen.getByText('2/5')).toBeInTheDocument();
  });

  test('TC-GAME-007: quit button shows confirmation modal', () => {
    render(
      <Wrapper>
        <GamePage lesson={lesson} onFinish={mockOnFinish} onQuit={mockOnQuit} />
      </Wrapper>
    );
    fireEvent.click(screen.getByText('✕'));
    expect(screen.getByText('Quit lesson?')).toBeInTheDocument();
  });

  test('TC-GAME-008: confirming quit calls onQuit', () => {
    render(
      <Wrapper>
        <GamePage lesson={lesson} onFinish={mockOnFinish} onQuit={mockOnQuit} />
      </Wrapper>
    );
    fireEvent.click(screen.getByText('✕'));
    fireEvent.click(screen.getByText('Quit'));
    expect(mockOnQuit).toHaveBeenCalled();
  });
});
```

---

### File: `ResultPage.test.tsx`

```tsx
import { render, screen, fireEvent } from '@testing-library/react';
import ResultPage from '../src/pages/ResultPage';
import { StoreProvider } from '../src/store';

const Wrapper = ({ children }: any) => <StoreProvider>{children}</StoreProvider>;

describe('ResultPage Tests', () => {
  test('TC-RESULT-001: shows "Perfect!" on score 100', () => {
    render(
      <Wrapper>
        <ResultPage score={100} xpEarned={20} lessonTitle="Greetings" onContinue={() => {}} onRetry={() => {}} />
      </Wrapper>
    );
    expect(screen.getByText('Perfect!')).toBeInTheDocument();
  });

  test('TC-RESULT-002: shows "Great job!" on score 80', () => {
    render(
      <Wrapper>
        <ResultPage score={80} xpEarned={16} lessonTitle="Greetings" onContinue={() => {}} onRetry={() => {}} />
      </Wrapper>
    );
    expect(screen.getByText('Great job!')).toBeInTheDocument();
  });

  test('TC-RESULT-003: shows "Keep going!" on score 50', () => {
    render(
      <Wrapper>
        <ResultPage score={50} xpEarned={10} lessonTitle="Greetings" onContinue={() => {}} onRetry={() => {}} />
      </Wrapper>
    );
    expect(screen.getByText('Keep going!')).toBeInTheDocument();
  });

  test('TC-RESULT-004: shows earned XP', () => {
    render(
      <Wrapper>
        <ResultPage score={80} xpEarned={20} lessonTitle="Greetings" onContinue={() => {}} onRetry={() => {}} />
      </Wrapper>
    );
    expect(screen.getByText('+20')).toBeInTheDocument();
  });

  test('TC-RESULT-005: Continue button calls onContinue', () => {
    const onContinue = jest.fn();
    render(
      <Wrapper>
        <ResultPage score={90} xpEarned={18} lessonTitle="Greetings" onContinue={onContinue} onRetry={() => {}} />
      </Wrapper>
    );
    fireEvent.click(screen.getByText('Continue →'));
    expect(onContinue).toHaveBeenCalled();
  });

  test('TC-RESULT-006: Retry button visible on imperfect score', () => {
    render(
      <Wrapper>
        <ResultPage score={70} xpEarned={14} lessonTitle="Greetings" onContinue={() => {}} onRetry={() => {}} />
      </Wrapper>
    );
    expect(screen.getByText('Try Again')).toBeInTheDocument();
  });

  test('TC-RESULT-007: Retry button NOT shown on perfect score', () => {
    render(
      <Wrapper>
        <ResultPage score={100} xpEarned={20} lessonTitle="Greetings" onContinue={() => {}} onRetry={() => {}} />
      </Wrapper>
    );
    expect(screen.queryByText('Try Again')).not.toBeInTheDocument();
  });
});
```

---

## 🌐 END-TO-END TESTS (Playwright / Cypress)

### File: `e2e/full-lesson-flow.spec.ts`

```ts
import { test, expect } from '@playwright/test';

test.describe('LinguaQuest – Full Lesson Flow', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('http://localhost:5173');
    await page.evaluate(() => localStorage.clear());
    await page.reload();
  });

  test('TC-E2E-001: Home screen loads with LinguaQuest title', async ({ page }) => {
    await expect(page.getByText('LinguaQuest')).toBeVisible();
  });

  test('TC-E2E-002: First unit is visible and unlocked', async ({ page }) => {
    await expect(page.getByText('Basics')).toBeVisible();
  });

  test('TC-E2E-003: Starting a lesson opens GamePage', async ({ page }) => {
    await page.getByText('Greetings').click();
    await expect(page.getByText('1/5')).toBeVisible();
  });

  test('TC-E2E-004: Answering all questions correctly finishes lesson', async ({ page }) => {
    await page.getByText('Greetings').click();

    const correctAnswers = [
      'Good day, sir.',
      'Good morning!',
      'See you later!',
      'meet',
      "I'm fine, thank you!",
    ];

    for (const answer of correctAnswers) {
      await page.getByText(answer).click();
      await page.getByText('Next →').click();
    }

    await expect(page.getByText(/Perfect|Great job/)).toBeVisible();
  });

  test('TC-E2E-005: XP is updated after completing a lesson', async ({ page }) => {
    const xpBefore = await page.getByText(/\d+ XP/).first().textContent();

    await page.getByText('Greetings').click();
    const answers = ['Good day, sir.', 'Good morning!', 'See you later!', 'meet', "I'm fine, thank you!"];
    for (const a of answers) {
      await page.getByText(a).click();
      await page.getByText('Next →').click();
    }

    await page.getByText('Continue →').click();
    const xpAfter = await page.getByText(/\d+ XP/).first().textContent();
    expect(xpAfter).not.toBe(xpBefore);
  });

  test('TC-E2E-006: Quitting mid-lesson returns to home', async ({ page }) => {
    await page.getByText('Greetings').click();
    await page.getByText('✕').click();
    await page.getByText('Quit').click();
    await expect(page.getByText('LinguaQuest')).toBeVisible();
  });

  test('TC-E2E-007: Profile page shows user stats', async ({ page }) => {
    await page.getByText('Profile').click();
    await expect(page.getByText('My Profile')).toBeVisible();
    await expect(page.getByText('Level 1')).toBeVisible();
  });

  test('TC-E2E-008: Leaderboard opens and shows rankings', async ({ page }) => {
    await page.getByText('Rank').click();
    await expect(page.getByText('Leaderboard')).toBeVisible();
    await expect(page.getByText('Weekly Ranking')).toBeVisible();
  });

  test('TC-E2E-009: Locked unit cannot be clicked', async ({ page }) => {
    const lockedUnit = page.getByText('Requires 50 XP to unlock').first();
    await expect(lockedUnit).toBeVisible();
    // Verify lesson buttons inside are disabled
    const lockedBtn = page.locator('button[disabled]').first();
    await expect(lockedBtn).toBeDisabled();
  });

  test('TC-E2E-010: Progress persists after page refresh', async ({ page }) => {
    await page.getByText('Greetings').click();
    const answers = ['Good day, sir.', 'Good morning!', 'See you later!', 'meet', "I'm fine, thank you!"];
    for (const a of answers) {
      await page.getByText(a).click();
      await page.getByText('Next →').click();
    }
    await page.getByText('Continue →').click();
    await page.reload();
    // Lesson should appear as completed (green checkmark)
    await expect(page.getByText('✓').first()).toBeVisible();
  });
});
```

---

## 📋 MANUAL TEST CHECKLIST

Use this for exploratory & visual testing:

| ID | Area | Test Case | Expected Result | Status |
|----|------|-----------|-----------------|--------|
| MT-001 | UI | Open app on mobile (375px) | Layout is responsive, no overflow | ⬜ |
| MT-002 | UI | Open app on desktop (1280px) | Content centered, max-width respected | ⬜ |
| MT-003 | Game | Answer all 5 questions correctly | 100% score, "Perfect!" screen | ⬜ |
| MT-004 | Game | Answer all 5 questions incorrectly | Low score, lives reduced, "Keep going!" | ⬜ |
| MT-005 | Game | Lose all 5 lives | Lives counter shows 0, heart icons fade | ⬜ |
| MT-006 | Progress | Complete 1st lesson of Unit 1 | 2nd lesson becomes clickable | ⬜ |
| MT-007 | Progress | Complete enough lessons to reach 50 XP | Unit 2 unlocks | ⬜ |
| MT-008 | Profile | Open profile after earning XP | XP bar filled correctly | ⬜ |
| MT-009 | Profile | Open profile after first lesson | "First Step" achievement shown | ⬜ |
| MT-010 | Leaderboard | Open leaderboard | "You" entry visible with correct XP | ⬜ |
| MT-011 | Persistence | Complete lesson, refresh page | Completed lesson shows ✓ | ⬜ |
| MT-012 | Reset | Click Reset Progress, confirm | All XP/lessons cleared, back to Level 1 | ⬜ |
| MT-013 | Animation | Complete lesson with 3 stars | Stars animate in with delay | ⬜ |
| MT-014 | Navigation | Tap all 3 bottom nav tabs | Each screen loads without error | ⬜ |
| MT-015 | Accessibility | Tab through questions with keyboard | Focus visible, options selectable | ⬜ |

---

## 🐛 BUG REPORT TEMPLATE

```
Title: [Short description of the bug]
ID: BUG-XXX
Date: YYYY-MM-DD
Reporter: [Your name]
Severity: Critical / High / Medium / Low

STEPS TO REPRODUCE:
1. 
2. 
3. 

EXPECTED RESULT:

ACTUAL RESULT:

ENVIRONMENT:
- Browser:
- Viewport:
- OS:

SCREENSHOT / VIDEO: [Attach if available]
```

---

## ⚙️ Running Tests

### Setup

```bash
npm install
npm install --save-dev jest @testing-library/react @testing-library/jest-dom @playwright/test ts-jest
npx playwright install
```

### Run Unit + Integration

```bash
npx jest --coverage
```

### Run E2E

```bash
# Start the dev server first:
npm run dev

# In another terminal:
npx playwright test
```

### Generate HTML Report

```bash
npx playwright show-report
```

---

*LinguaQuest QA Suite – Written for Junior QA Engineers*
*Total Test Cases: 12 unit + 9 integration + 10 E2E + 15 manual = 46 test cases*
