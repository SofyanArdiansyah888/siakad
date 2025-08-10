# Frontend Testing Guide

## 🧪 Testing Strategy

### Testing Pyramid
- **Unit Tests** (70%) - Individual components and functions
- **Integration Tests** (20%) - Component interactions
- **E2E Tests** (10%) - User workflows

### Testing Tools
- **Vitest** - Unit testing framework
- **React Testing Library** - Component testing
- **Playwright** - E2E testing
- **MSW (Mock Service Worker)** - API mocking

## 🧩 Unit Testing

### Setup
```bash
# Install dependencies
npm install -D vitest @testing-library/react @testing-library/jest-dom @testing-library/user-event jsdom

# Configure vitest
# vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
    globals: true,
  },
});
```

### Test Setup
```tsx
// src/test/setup.ts
import '@testing-library/jest-dom';
import { vi } from 'vitest';

// Mock API client
vi.mock('../lib/api-client', () => ({
  apiClient: {
    get: vi.fn(),
    post: vi.fn(),
    put: vi.fn(),
    delete: vi.fn(),
  },
}));
```

### Component Testing Examples

#### Basic Component Test
```tsx
// __tests__/components/Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from '../components/ui/button';

describe('Button', () => {
  it('renders with correct text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button', { name: 'Click me' })).toBeInTheDocument();
  });

  it('calls onClick when clicked', () => {
    const handleClick = vi.fn();
    render(<Button onClick={handleClick}>Click me</Button>);
    
    fireEvent.click(screen.getByRole('button'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('applies variant classes correctly', () => {
    render(<Button variant="destructive">Delete</Button>);
    const button = screen.getByRole('button');
    expect(button).toHaveClass('bg-destructive');
  });
});
```

#### Form Component Test
```tsx
// __tests__/components/StudentForm.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { StudentForm } from '../components/StudentForm';

describe('StudentForm', () => {
  const mockOnSubmit = vi.fn();

  beforeEach(() => {
    mockOnSubmit.mockClear();
  });

  it('renders form fields', () => {
    render(<StudentForm onSubmit={mockOnSubmit} />);
    
    expect(screen.getByLabelText(/name/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/nim/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/program studi/i)).toBeInTheDocument();
  });

  it('submits form with correct data', async () => {
    const user = userEvent.setup();
    render(<StudentForm onSubmit={mockOnSubmit} />);
    
    await user.type(screen.getByLabelText(/name/i), 'John Doe');
    await user.type(screen.getByLabelText(/nim/i), '123456');
    await user.selectOptions(screen.getByLabelText(/program studi/i), 'informatika');
    
    await user.click(screen.getByRole('button', { name: /submit/i }));
    
    await waitFor(() => {
      expect(mockOnSubmit).toHaveBeenCalledWith({
        name: 'John Doe',
        nim: '123456',
        programStudi: 'informatika',
      });
    });
  });

  it('shows validation errors for invalid input', async () => {
    const user = userEvent.setup();
    render(<StudentForm onSubmit={mockOnSubmit} />);
    
    await user.click(screen.getByRole('button', { name: /submit/i }));
    
    expect(screen.getByText(/name is required/i)).toBeInTheDocument();
    expect(screen.getByText(/nim is required/i)).toBeInTheDocument();
  });
});
```

#### Hook Testing
```tsx
// __tests__/hooks/useStudents.test.ts
import { renderHook, waitFor } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { useStudents } from '../hooks/useStudents';
import { apiClient } from '../lib/api-client';

const createWrapper = () => {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false },
    },
  });
  
  return ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
};

describe('useStudents', () => {
  it('fetches students successfully', async () => {
    const mockStudents = [
      { id: '1', name: 'John Doe', nim: '123456' },
      { id: '2', name: 'Jane Smith', nim: '123457' },
    ];

    // Mock REST API response
    vi.mocked(apiClient.get).mockResolvedValue({
      data: {
        success: true,
        data: {
          students: mockStudents,
          pagination: {
            page: 1,
            limit: 10,
            total: 2,
            totalPages: 1,
          },
        },
      },
    } as any);

    const { result } = renderHook(() => useStudents(), {
      wrapper: createWrapper(),
    });

    await waitFor(() => {
      expect(result.current.data).toEqual(mockStudents);
    });
  });
});
```

## 🔗 Integration Testing

### Component Integration Tests
```tsx
// __tests__/integration/StudentList.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { StudentList } from '../components/StudentList';
import { apiClient } from '../lib/api-client';

const mockStudents = [
  { id: '1', name: 'John Doe', nim: '123456', programStudi: 'Informatika' },
  { id: '2', name: 'Jane Smith', nim: '123457', programStudi: 'Sistem Informasi' },
];

describe('StudentList Integration', () => {
  it('renders student list with data', async () => {
    // Mock API response
    vi.mocked(apiClient.get).mockResolvedValue({
      data: {
        success: true,
        data: {
          students: mockStudents,
          pagination: {
            page: 1,
            limit: 10,
            total: 2,
            totalPages: 1,
          },
        },
      },
    } as any);

    render(
      <QueryClientProvider client={new QueryClient()}>
        <StudentList />
      </QueryClientProvider>
    );

    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeInTheDocument();
      expect(screen.getByText('Jane Smith')).toBeInTheDocument();
    });
  });

  it('shows loading state', () => {
    vi.mocked(apiClient.get).mockImplementation(() => new Promise(() => {}));

    render(
      <QueryClientProvider client={new QueryClient()}>
        <StudentList />
      </QueryClientProvider>
    );

    expect(screen.getByText(/loading/i)).toBeInTheDocument();
  });

  it('shows error state', async () => {
    vi.mocked(apiClient.get).mockRejectedValue(new Error('Failed to fetch'));

    render(
      <QueryClientProvider client={new QueryClient()}>
        <StudentList />
      </QueryClientProvider>
    );

    await waitFor(() => {
      expect(screen.getByText(/error/i)).toBeInTheDocument();
    });
  });
});
```

## 🌐 E2E Testing with Playwright

### Setup
```bash
# Install Playwright
npm install -D @playwright/test

# Install browsers
npx playwright install

# Generate config
npx playwright init
```

### Configuration
```ts
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:5173',
    trace: 'on-first-retry',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:5173',
    reuseExistingServer: !process.env.CI,
  },
});
```

### E2E Test Examples
```ts
// e2e/students.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Students Page', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/students');
  });

  test('should display students list', async ({ page }) => {
    await expect(page.getByRole('heading', { name: 'Students' })).toBeVisible();
    await expect(page.getByRole('table')).toBeVisible();
  });

  test('should add new student', async ({ page }) => {
    // Click add button
    await page.getByRole('button', { name: 'Add Student' }).click();
    
    // Fill form
    await page.getByLabel('Name').fill('John Doe');
    await page.getByLabel('NIM').fill('123456');
    await page.getByLabel('Program Studi').selectOption('informatika');
    
    // Submit form
    await page.getByRole('button', { name: 'Submit' }).click();
    
    // Verify student was added
    await expect(page.getByText('John Doe')).toBeVisible();
    await expect(page.getByText('123456')).toBeVisible();
  });

  test('should edit student', async ({ page }) => {
    // Click edit button for first student
    await page.getByRole('button', { name: 'Edit' }).first().click();
    
    // Update name
    await page.getByLabel('Name').clear();
    await page.getByLabel('Name').fill('Updated Name');
    
    // Submit form
    await page.getByRole('button', { name: 'Update' }).click();
    
    // Verify update
    await expect(page.getByText('Updated Name')).toBeVisible();
  });

  test('should delete student', async ({ page }) => {
    const studentName = await page.getByRole('cell').first().textContent();
    
    // Click delete button
    await page.getByRole('button', { name: 'Delete' }).first().click();
    
    // Confirm deletion
    await page.getByRole('button', { name: 'Confirm' }).click();
    
    // Verify student was removed
    await expect(page.getByText(studentName!)).not.toBeVisible();
  });
});
```

## 🎯 Test Coverage

### Coverage Configuration
```ts
// vitest.config.ts
export default defineConfig({
  test: {
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'src/test/',
        '**/*.d.ts',
        '**/*.config.*',
      ],
    },
  },
});
```

### Coverage Targets
- **Statements**: 80%
- **Branches**: 75%
- **Functions**: 80%
- **Lines**: 80%

### Coverage Report
```bash
# Generate coverage report
npm run test:coverage

# View HTML report
open coverage/index.html
```

## 🚀 Running Tests

### Development
```bash
# Run all tests
npm run test

# Run tests in watch mode
npm run test:watch

# Run specific test file
npm run test Button.test.tsx

# Run tests with coverage
npm run test:coverage
```

### E2E Tests
```bash
# Run E2E tests
npm run test:e2e

# Run E2E tests in headed mode
npm run test:e2e:headed

# Run specific E2E test
npm run test:e2e students.spec.ts
```

### CI/CD Integration
```yaml
# .github/workflows/test.yml
name: Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run test:coverage
      - run: npm run test:e2e
```

## 📚 Best Practices

### Testing Guidelines
1. **Test Behavior, Not Implementation** - Focus on what the component does, not how it does it
2. **Use Semantic Queries** - Prefer `getByRole`, `getByLabelText` over `getByTestId`
3. **Test User Interactions** - Test from the user's perspective
4. **Keep Tests Simple** - One assertion per test when possible
5. **Use Descriptive Test Names** - Make test names clear and descriptive

### Mocking Guidelines
1. **Mock External Dependencies** - Mock API calls, timers, etc.
2. **Use MSW for API Mocking** - More realistic than manual mocking
3. **Reset Mocks Between Tests** - Use `beforeEach` to reset mocks
4. **Test Error States** - Don't just test happy path

### Performance Testing
```tsx
// Performance test example
test('should render large list efficiently', async () => {
  const largeList = Array.from({ length: 1000 }, (_, i) => ({
    id: i.toString(),
    name: `Student ${i}`,
    nim: `NIM${i}`,
  }));

  const startTime = performance.now();
  render(<StudentList students={largeList} />);
  const endTime = performance.now();

  expect(endTime - startTime).toBeLessThan(100); // Should render in < 100ms
});
```
