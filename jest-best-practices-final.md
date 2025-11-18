# Jest Best Practices for React Applications

## Table of Contents
1. [Naming Conventions](#naming-conventions)
2. [Project Structure](#project-structure)
3. [Testing Components](#testing-components)
4. [Testing Custom Hooks](#testing-custom-hooks)
5. [Mock Functions and Data](#mock-functions-and-data)
6. [Async Testing](#async-testing)
7. [Snapshot Testing](#snapshot-testing)
8. [Coverage and Performance](#coverage-and-performance)

---

## Naming Conventions

### Test File Naming
- Use `.test.js` or `.spec.js` suffix for test files
- Mirror the component/file name: `Button.jsx` → `Button.test.jsx`
- Place test files adjacent to source files or in `__tests__` folder

### Test Suite and Case Naming
```javascript
// ✅ Good: Descriptive and readable
describe('LoginForm', () => {
  describe('when user submits valid credentials', () => {
    it('should call onSubmit with email and password', () => {});
    it('should display success message', () => {});
  });

  describe('when user submits invalid credentials', () => {
    it('should display error message', () => {});
    it('should not call onSubmit', () => {});
  });
});

// ❌ Bad: Vague and unclear
describe('form tests', () => {
  it('works', () => {});
  it('test 2', () => {});
});
```

### Naming Patterns
- **Components**: `ComponentName` (e.g., `UserProfile`, `NavigationBar`)
- **Hooks**: `useHookName` (e.g., `useFetchData`, `useAuth`)
- **Utils**: `functionName` (e.g., `formatDate`, `validateEmail`)

---

## Project Structure

```
src/
├── components/
│   ├── Button/
│   │   ├── Button.jsx
│   │   ├── Button.test.jsx
│   │   └── Button.module.css
│   └── LoginForm/
│       ├── LoginForm.jsx
│       └── LoginForm.test.jsx
├── hooks/
│   ├── useFetch.js
│   └── useFetch.test.js
├── utils/
│   ├── helpers.js
│   └── helpers.test.js
└── __mocks__/
    ├── axios.js
    └── mockData.js
```

---

## Testing Components

### Basic Component Testing

```javascript
import { render, screen, fireEvent } from '@testing-library/react';
import '@testing-library/jest-dom';
import Button from './Button';

describe('Button Component', () => {
  it('should render with correct text', () => {
    render(<Button>Click Me</Button>);
    
    const button = screen.getByRole('button', { name: /click me/i });
    expect(button).toBeInTheDocument();
  });

  it('should call onClick handler when clicked', () => {
    const mockOnClick = jest.fn();
    render(<Button onClick={mockOnClick}>Click Me</Button>);
    
    const button = screen.getByRole('button');
    fireEvent.click(button);
    
    expect(mockOnClick).toHaveBeenCalledTimes(1);
  });

  it('should be disabled when disabled prop is true', () => {
    render(<Button disabled>Click Me</Button>);
    
    const button = screen.getByRole('button');
    expect(button).toBeDisabled();
  });

  it('should apply custom className', () => {
    const { container } = render(
      <Button className="custom-class">Click Me</Button>
    );
    
    const button = container.querySelector('.custom-class');
    expect(button).toBeInTheDocument();
  });
});
```

### Testing Component with User Interactions

```javascript
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import LoginForm from './LoginForm';

describe('LoginForm Component', () => {
  const mockOnSubmit = jest.fn();

  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('should update input values when user types', async () => {
    render(<LoginForm onSubmit={mockOnSubmit} />);
    
    const emailInput = screen.getByLabelText(/email/i);
    const passwordInput = screen.getByLabelText(/password/i);
    
    await userEvent.type(emailInput, 'test@example.com');
    await userEvent.type(passwordInput, 'password123');
    
    expect(emailInput).toHaveValue('test@example.com');
    expect(passwordInput).toHaveValue('password123');
  });

  it('should call onSubmit with form data when submitted', async () => {
    render(<LoginForm onSubmit={mockOnSubmit} />);
    
    const emailInput = screen.getByLabelText(/email/i);
    const passwordInput = screen.getByLabelText(/password/i);
    const submitButton = screen.getByRole('button', { name: /submit/i });
    
    await userEvent.type(emailInput, 'test@example.com');
    await userEvent.type(passwordInput, 'password123');
    await userEvent.click(submitButton);
    
    expect(mockOnSubmit).toHaveBeenCalledWith({
      email: 'test@example.com',
      password: 'password123'
    });
  });

  it('should display validation error for invalid email', async () => {
    render(<LoginForm onSubmit={mockOnSubmit} />);
    
    const emailInput = screen.getByLabelText(/email/i);
    const submitButton = screen.getRole('button', { name: /submit/i });
    
    await userEvent.type(emailInput, 'invalid-email');
    await userEvent.click(submitButton);
    
    const errorMessage = await screen.findByText(/please enter a valid email/i);
    expect(errorMessage).toBeInTheDocument();
    expect(mockOnSubmit).not.toHaveBeenCalled();
  });
});
```

### Testing Components with Props

```javascript
import { render, screen } from '@testing-library/react';
import UserCard from './UserCard';

describe('UserCard Component', () => {
  const mockUser = {
    id: 1,
    name: 'John Doe',
    email: 'john@example.com',
    role: 'admin'
  };

  it('should render user information correctly', () => {
    render(<UserCard user={mockUser} />);
    
    expect(screen.getByText('John Doe')).toBeInTheDocument();
    expect(screen.getByText('john@example.com')).toBeInTheDocument();
    expect(screen.getByText(/admin/i)).toBeInTheDocument();
  });

  it('should render loading state when user is null', () => {
    render(<UserCard user={null} />);
    
    expect(screen.getByText(/loading/i)).toBeInTheDocument();
  });

  it('should call onDelete with user id when delete button clicked', () => {
    const mockOnDelete = jest.fn();
    render(<UserCard user={mockUser} onDelete={mockOnDelete} />);
    
    const deleteButton = screen.getByRole('button', { name: /delete/i });
    fireEvent.click(deleteButton);
    
    expect(mockOnDelete).toHaveBeenCalledWith(mockUser.id);
  });
});
```

---

## Testing Custom Hooks

### Using renderHook from @testing-library/react

```javascript
import { renderHook, waitFor } from '@testing-library/react';
import { act } from 'react';
import useFetch from './useFetch';

// Mock fetch API
global.fetch = jest.fn();

describe('useFetch Hook', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('should fetch data successfully', async () => {
    const mockData = { users: [{ id: 1, name: 'John' }] };
    
    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => mockData
    });

    const { result } = renderHook(() => useFetch('/api/users'));

    expect(result.current.loading).toBe(true);
    expect(result.current.data).toBe(null);

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    expect(result.current.data).toEqual(mockData);
    expect(result.current.error).toBe(null);
    expect(fetch).toHaveBeenCalledWith('/api/users');
  });

  it('should handle fetch errors', async () => {
    const mockError = new Error('Network error');
    fetch.mockRejectedValueOnce(mockError);

    const { result } = renderHook(() => useFetch('/api/users'));

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    expect(result.current.error).toEqual(mockError);
    expect(result.current.data).toBe(null);
  });

  it('should refetch data when refetch is called', async () => {
    const mockData1 = { count: 1 };
    const mockData2 = { count: 2 };

    fetch
      .mockResolvedValueOnce({ ok: true, json: async () => mockData1 })
      .mockResolvedValueOnce({ ok: true, json: async () => mockData2 });

    const { result } = renderHook(() => useFetch('/api/data'));

    await waitFor(() => {
      expect(result.current.data).toEqual(mockData1);
    });

    act(() => {
      result.current.refetch();
    });

    await waitFor(() => {
      expect(result.current.data).toEqual(mockData2);
    });

    expect(fetch).toHaveBeenCalledTimes(2);
  });
});
```

### Testing Hook with State Updates

```javascript
import { renderHook, act } from '@testing-library/react';
import useCounter from './useCounter';

describe('useCounter Hook', () => {
  it('should initialize with default value', () => {
    const { result } = renderHook(() => useCounter());
    
    expect(result.current.count).toBe(0);
  });

  it('should initialize with custom value', () => {
    const { result } = renderHook(() => useCounter(10));
    
    expect(result.current.count).toBe(10);
  });

  it('should increment count', () => {
    const { result } = renderHook(() => useCounter());
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });

  it('should decrement count', () => {
    const { result } = renderHook(() => useCounter(5));
    
    act(() => {
      result.current.decrement();
    });
    
    expect(result.current.count).toBe(4);
  });

  it('should reset count to initial value', () => {
    const { result } = renderHook(() => useCounter(5));
    
    act(() => {
      result.current.increment();
      result.current.increment();
    });
    
    expect(result.current.count).toBe(7);
    
    act(() => {
      result.current.reset();
    });
    
    expect(result.current.count).toBe(5);
  });
});
```

---

## Mock Functions and Data

### Creating Mock Functions

```javascript
describe('Mock Functions Examples', () => {
  it('should track function calls', () => {
    const mockFn = jest.fn();
    
    mockFn('hello', 123);
    mockFn('world');
    
    expect(mockFn).toHaveBeenCalledTimes(2);
    expect(mockFn).toHaveBeenCalledWith('hello', 123);
    expect(mockFn).toHaveBeenLastCalledWith('world');
  });

  it('should return mocked values', () => {
    const mockFn = jest.fn()
      .mockReturnValueOnce('first')
      .mockReturnValueOnce('second')
      .mockReturnValue('default');
    
    expect(mockFn()).toBe('first');
    expect(mockFn()).toBe('second');
    expect(mockFn()).toBe('default');
    expect(mockFn()).toBe('default');
  });

  it('should implement custom logic', () => {
    const mockFn = jest.fn((x, y) => x + y);
    
    const result = mockFn(2, 3);
    
    expect(result).toBe(5);
    expect(mockFn).toHaveBeenCalledWith(2, 3);
  });
});
```

### Mocking Modules

```javascript
// __mocks__/axios.js
export default {
  get: jest.fn(),
  post: jest.fn(),
  put: jest.fn(),
  delete: jest.fn()
};

// UserService.test.js
import axios from 'axios';
import { getUsers, createUser } from './UserService';

jest.mock('axios');

describe('UserService', () => {
  afterEach(() => {
    jest.clearAllMocks();
  });

  describe('getUsers', () => {
    it('should fetch users successfully', async () => {
      const mockUsers = [
        { id: 1, name: 'John' },
        { id: 2, name: 'Jane' }
      ];
      
      axios.get.mockResolvedValue({ data: mockUsers });
      
      const users = await getUsers();
      
      expect(axios.get).toHaveBeenCalledWith('/api/users');
      expect(users).toEqual(mockUsers);
    });

    it('should handle errors when fetching users', async () => {
      const mockError = new Error('Network error');
      axios.get.mockRejectedValue(mockError);
      
      await expect(getUsers()).rejects.toThrow('Network error');
      expect(axios.get).toHaveBeenCalledWith('/api/users');
    });
  });

  describe('createUser', () => {
    it('should create user successfully', async () => {
      const newUser = { name: 'John Doe', email: 'john@example.com' };
      const createdUser = { id: 1, ...newUser };
      
      axios.post.mockResolvedValue({ data: createdUser });
      
      const result = await createUser(newUser);
      
      expect(axios.post).toHaveBeenCalledWith('/api/users', newUser);
      expect(result).toEqual(createdUser);
    });
  });
});
```

### Mocking Context and Providers

```javascript
import { render, screen } from '@testing-library/react';
import { AuthContext } from './AuthContext';
import Dashboard from './Dashboard';

describe('Dashboard Component', () => {
  const mockAuthContextValue = {
    user: { id: 1, name: 'John Doe', role: 'admin' },
    isAuthenticated: true,
    login: jest.fn(),
    logout: jest.fn()
  };

  const renderWithAuth = (component, authValue = mockAuthContextValue) => {
    return render(
      <AuthContext.Provider value={authValue}>
        {component}
      </AuthContext.Provider>
    );
  };

  it('should display user name when authenticated', () => {
    renderWithAuth(<Dashboard />);
    
    expect(screen.getByText(/welcome, john doe/i)).toBeInTheDocument();
  });

  it('should display login prompt when not authenticated', () => {
    const unauthenticatedValue = {
      ...mockAuthContextValue,
      user: null,
      isAuthenticated: false
    };
    
    renderWithAuth(<Dashboard />, unauthenticatedValue);
    
    expect(screen.getByText(/please log in/i)).toBeInTheDocument();
  });

  it('should call logout when logout button clicked', () => {
    renderWithAuth(<Dashboard />);
    
    const logoutButton = screen.getByRole('button', { name: /logout/i });
    fireEvent.click(logoutButton);
    
    expect(mockAuthContextValue.logout).toHaveBeenCalledTimes(1);
  });
});
```

---

## Async Testing

### Testing Async Components

```javascript
import { render, screen, waitFor } from '@testing-library/react';
import UserList from './UserList';
import * as api from './api';

jest.mock('./api');

describe('UserList Component', () => {
  const mockUsers = [
    { id: 1, name: 'John Doe' },
    { id: 2, name: 'Jane Smith' }
  ];

  it('should display loading state initially', () => {
    api.fetchUsers.mockReturnValue(new Promise(() => {})); // Never resolves
    
    render(<UserList />);
    
    expect(screen.getByText(/loading/i)).toBeInTheDocument();
  });

  it('should display users after loading', async () => {
    api.fetchUsers.mockResolvedValue(mockUsers);
    
    render(<UserList />);
    
    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeInTheDocument();
    });
    
    expect(screen.getByText('Jane Smith')).toBeInTheDocument();
    expect(screen.queryByText(/loading/i)).not.toBeInTheDocument();
  });

  it('should display error message on fetch failure', async () => {
    api.fetchUsers.mockRejectedValue(new Error('Failed to fetch'));
    
    render(<UserList />);
    
    const errorMessage = await screen.findByText(/failed to fetch/i);
    expect(errorMessage).toBeInTheDocument();
  });

  it('should retry fetching users when retry button clicked', async () => {
    api.fetchUsers
      .mockRejectedValueOnce(new Error('Failed'))
      .mockResolvedValueOnce(mockUsers);
    
    render(<UserList />);
    
    await screen.findByText(/failed/i);
    
    const retryButton = screen.getByRole('button', { name: /retry/i });
    fireEvent.click(retryButton);
    
    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeInTheDocument();
    });
    
    expect(api.fetchUsers).toHaveBeenCalledTimes(2);
  });
});
```

### Testing with Timers

```javascript
import { render, screen, act } from '@testing-library/react';
import Toast from './Toast';

jest.useFakeTimers();

describe('Toast Component', () => {
  it('should auto-hide after specified duration', () => {
    const mockOnClose = jest.fn();
    
    render(
      <Toast message="Success!" duration={3000} onClose={mockOnClose} />
    );
    
    expect(screen.getByText('Success!')).toBeInTheDocument();
    
    act(() => {
      jest.advanceTimersByTime(3000);
    });
    
    expect(mockOnClose).toHaveBeenCalledTimes(1);
  });

  it('should not auto-hide if duration is not provided', () => {
    const mockOnClose = jest.fn();
    
    render(<Toast message="Stay visible" onClose={mockOnClose} />);
    
    act(() => {
      jest.advanceTimersByTime(5000);
    });
    
    expect(mockOnClose).not.toHaveBeenCalled();
    expect(screen.getByText('Stay visible')).toBeInTheDocument();
  });
});

afterAll(() => {
  jest.useRealTimers();
});
```

---

## Snapshot Testing

### Basic Snapshot Testing

```javascript
import { render } from '@testing-library/react';
import Button from './Button';

describe('Button Snapshot Tests', () => {
  it('should match snapshot for primary button', () => {
    const { container } = render(<Button variant="primary">Click Me</Button>);
    expect(container.firstChild).toMatchSnapshot();
  });

  it('should match snapshot for disabled button', () => {
    const { container } = render(<Button disabled>Disabled</Button>);
    expect(container.firstChild).toMatchSnapshot();
  });

  it('should match snapshot for button with icon', () => {
    const { container } = render(
      <Button icon={<span>🚀</span>}>Launch</Button>
    );
    expect(container.firstChild).toMatchSnapshot();
  });
});
```

### Inline Snapshots

```javascript
import { render } from '@testing-library/react';
import Badge from './Badge';

describe('Badge Inline Snapshots', () => {
  it('should render success badge correctly', () => {
    const { container } = render(<Badge status="success">Active</Badge>);
    
    expect(container.firstChild).toMatchInlineSnapshot(`
      <span
        class="badge badge-success"
      >
        Active
      </span>
    `);
  });
});
```

---

## Coverage and Performance

### Coverage Configuration (jest.config.js)

```javascript
module.exports = {
  collectCoverageFrom: [
    'src/**/*.{js,jsx}',
    '!src/index.js',
    '!src/**/*.test.{js,jsx}',
    '!src/**/*.stories.{js,jsx}'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  },
  coverageReporters: ['text', 'lcov', 'html']
};
```

### Performance Testing

```javascript
import { render } from '@testing-library/react';
import { performance } from 'perf_hooks';
import LargeList from './LargeList';

describe('LargeList Performance', () => {
  it('should render large list within acceptable time', () => {
    const items = Array.from({ length: 1000 }, (_, i) => ({
      id: i,
      name: `Item ${i}`
    }));

    const startTime = performance.now();
    render(<LargeList items={items} />);
    const endTime = performance.now();

    const renderTime = endTime - startTime;
    
    // Expect render to complete within 100ms
    expect(renderTime).toBeLessThan(100);
  });
});
```

### Best Practices Summary

1. **Keep tests isolated** - Each test should be independent
2. **Use descriptive names** - Make test intent clear
3. **Follow AAA pattern** - Arrange, Act, Assert
4. **Mock external dependencies** - Keep tests fast and reliable
5. **Test user behavior** - Focus on what users see and do
6. **Avoid implementation details** - Test the interface, not internals
7. **Use data-testid sparingly** - Prefer accessible queries
8. **Clean up after tests** - Use beforeEach/afterEach for cleanup
9. **Aim for meaningful coverage** - Quality over quantity
10. **Keep tests maintainable** - Refactor test code like production code