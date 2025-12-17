---
description: Generate component tests using Vitest and React Testing Library
model: claude-opus-4-5
---

Generate comprehensive component tests for the following React component or feature.

## Test Target

$ARGUMENTS

---

## MCP Tools Integration

**IMPORTANT:** Use these tools for best practices:

### Required Tools
1. **Context7** - Fetch latest Vitest and React Testing Library documentation
2. **Sequential Thinking** - Plan test coverage systematically

---

## Why Vitest + React Testing Library (2025 Standard)

| Aspect | Vitest | Jest |
|--------|--------|------|
| Speed | 2-5x faster | Baseline |
| ESM/TypeScript | Native support | Requires config |
| Config | Shares Vite config | Separate config |
| API | Jest-compatible | Original |
| Watch mode | Instant | Good |

**Recommendation:** Use Vitest for all new projects, especially with Vite/Next.js.

---

## Testing Stack Setup

### Installation

```bash
# Core testing libraries
pnpm add -D vitest @testing-library/react @testing-library/jest-dom @testing-library/user-event

# Additional utilities
pnpm add -D jsdom @vitest/coverage-v8 @axe-core/react
```

### Vitest Config

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: ['./tests/setup.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: ['node_modules/', 'tests/'],
    },
  },
});
```

### Test Setup

```typescript
// tests/setup.ts
import '@testing-library/jest-dom/vitest';
import { cleanup } from '@testing-library/react';
import { afterEach } from 'vitest';

afterEach(() => {
  cleanup();
});
```

---

## Component Testing Patterns

### 1. **Basic Component Test**

```typescript
// components/__tests__/Button.test.tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect, vi } from 'vitest';
import { Button } from '../Button';

describe('Button', () => {
  it('renders with text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button', { name: /click me/i })).toBeInTheDocument();
  });

  it('calls onClick when clicked', async () => {
    const user = userEvent.setup();
    const handleClick = vi.fn();

    render(<Button onClick={handleClick}>Click me</Button>);
    await user.click(screen.getByRole('button'));

    expect(handleClick).toHaveBeenCalledOnce();
  });

  it('is disabled when disabled prop is true', () => {
    render(<Button disabled>Click me</Button>);
    expect(screen.getByRole('button')).toBeDisabled();
  });
});
```

### 2. **Testing with Context/Providers**

```typescript
// tests/utils/render.tsx
import { render, RenderOptions } from '@testing-library/react';
import { ReactElement } from 'react';

// Add your providers here
const AllProviders = ({ children }: { children: React.ReactNode }) => {
  return (
    <ThemeProvider>
      <AuthProvider>
        {children}
      </AuthProvider>
    </ThemeProvider>
  );
};

const customRender = (ui: ReactElement, options?: RenderOptions) =>
  render(ui, { wrapper: AllProviders, ...options });

export * from '@testing-library/react';
export { customRender as render };
```

### 3. **Testing Async Components**

```typescript
import { render, screen, waitFor } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';

describe('AsyncComponent', () => {
  it('shows loading state then data', async () => {
    render(<AsyncComponent />);

    // Loading state
    expect(screen.getByText(/loading/i)).toBeInTheDocument();

    // Wait for data
    await waitFor(() => {
      expect(screen.getByText(/data loaded/i)).toBeInTheDocument();
    });
  });
});
```

### 4. **Mocking Hooks and Modules**

```typescript
import { vi } from 'vitest';

// Mock a hook
vi.mock('@/hooks/useAuth', () => ({
  useAuth: () => ({
    user: { id: '1', name: 'Test User' },
    isAuthenticated: true,
  }),
}));

// Mock a module
vi.mock('@/lib/api', () => ({
  fetchData: vi.fn().mockResolvedValue({ data: 'mocked' }),
}));

// Mock next/navigation
vi.mock('next/navigation', () => ({
  useRouter: () => ({
    push: vi.fn(),
    back: vi.fn(),
  }),
  useSearchParams: () => new URLSearchParams(),
}));
```

### 5. **Testing Forms**

```typescript
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

describe('ContactForm', () => {
  it('submits form with valid data', async () => {
    const user = userEvent.setup();
    const onSubmit = vi.fn();

    render(<ContactForm onSubmit={onSubmit} />);

    await user.type(screen.getByLabelText(/name/i), 'John Doe');
    await user.type(screen.getByLabelText(/email/i), 'john@example.com');
    await user.type(screen.getByLabelText(/message/i), 'Hello!');
    await user.click(screen.getByRole('button', { name: /submit/i }));

    expect(onSubmit).toHaveBeenCalledWith({
      name: 'John Doe',
      email: 'john@example.com',
      message: 'Hello!',
    });
  });

  it('shows validation errors for invalid input', async () => {
    const user = userEvent.setup();

    render(<ContactForm onSubmit={vi.fn()} />);
    await user.click(screen.getByRole('button', { name: /submit/i }));

    expect(screen.getByText(/name is required/i)).toBeInTheDocument();
  });
});
```

### 6. **Accessibility Testing**

```typescript
import { render } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

describe('Component Accessibility', () => {
  it('has no accessibility violations', async () => {
    const { container } = render(<MyComponent />);
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });
});
```

### 7. **Snapshot Testing**

```typescript
import { render } from '@testing-library/react';
import { expect, it } from 'vitest';

it('matches snapshot', () => {
  const { container } = render(<Component />);
  expect(container).toMatchSnapshot();
});
```

---

## Output Format

### Test File Structure

```
src/
  components/
    Button/
      Button.tsx
      Button.test.tsx        # Co-located test

tests/
  setup.ts                   # Global setup
  utils/
    render.tsx               # Custom render with providers
  __mocks__/
    next/                    # Next.js mocks
```

### npm Scripts

```json
{
  "scripts": {
    "test": "vitest",
    "test:watch": "vitest --watch",
    "test:coverage": "vitest run --coverage",
    "test:ui": "vitest --ui"
  }
}
```

---

## Best Practices

**DO:**
- Test behavior, not implementation
- Use `screen` queries (getByRole, getByLabelText)
- Use `userEvent` over `fireEvent`
- Test accessibility
- Keep tests focused and isolated
- Use meaningful test descriptions

**DON'T:**
- Test internal state directly
- Use `getByTestId` as first choice
- Mock everything (test real integrations when possible)
- Write brittle tests tied to DOM structure
- Skip edge cases and error states

---

Generate complete, well-structured component tests with proper mocking and assertions.

---
*Originally created by [Pras](https://github.com/andhikapraa/pras-claude-code)*
