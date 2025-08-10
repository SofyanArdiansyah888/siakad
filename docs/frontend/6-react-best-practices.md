# React Best Practices

## 🎯 Overview

Dokumentasi best practices untuk pengembangan React di SIAKAD Frontend, mencakup patterns, conventions, dan guidelines untuk maintainable dan scalable code.

## 📋 Table of Contents

1. [Component Architecture](#component-architecture)
2. [State Management](#state-management)
3. [Performance Optimization](#performance-optimization)
4. [TypeScript Integration](#typescript-integration)
5. [Code Organization](#code-organization)
6. [Testing Patterns](#testing-patterns)
7. [Accessibility](#accessibility)
8. [Security](#security)
9. [Error Handling](#error-handling)
10. [Documentation](#documentation)
11. [Naming Conventions](#naming-conventions)

## 🏗️ Component Architecture

### Component Design Principles

#### 1. Single Responsibility Principle
```tsx
// ❌ Bad: Component doing too many things
const StudentDashboard = () => {
  const [students, setStudents] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  // Fetching, filtering, sorting, rendering all in one component
  return (
    <div>
      {/* 100+ lines of JSX */}
    </div>
  );
};

// ✅ Good: Separated concerns
const StudentDashboard = () => {
  return (
    <div>
      <StudentFilters />
      <StudentList />
      <StudentStats />
    </div>
  );
};
```

#### 2. Composition over Inheritance
```tsx
// ❌ Bad: Inheritance-like pattern
const BaseCard = ({ children, ...props }) => (
  <div className="base-card" {...props}>
    {children}
  </div>
);

const StudentCard = ({ student }) => (
  <BaseCard>
    <h3>{student.name}</h3>
    <p>{student.nim}</p>
  </BaseCard>
);

// ✅ Good: Composition pattern
const Card = ({ children, variant = "default", ...props }) => (
  <div className={`card card--${variant}`} {...props}>
    {children}
  </div>
);

const StudentCard = ({ student }) => (
  <Card variant="student">
    <CardHeader>
      <CardTitle>{student.name}</CardTitle>
    </CardHeader>
    <CardContent>
      <p>NIM: {student.nim}</p>
    </CardContent>
  </Card>
);
```

#### 3. Props Interface Design
```tsx
// ✅ Good: Clear and specific interfaces
interface StudentCardProps {
  student: Student;
  onEdit?: (id: string) => void;
  onDelete?: (id: string) => void;
  variant?: 'default' | 'compact' | 'detailed';
  showActions?: boolean;
  className?: string;
}

// ✅ Good: Default props with destructuring
const StudentCard: React.FC<StudentCardProps> = ({
  student,
  onEdit,
  onDelete,
  variant = 'default',
  showActions = true,
  className,
}) => {
  // Component implementation
};
```

### Component Patterns

#### 1. Container/Presentational Pattern
```tsx
// Container Component (Logic)
const StudentListContainer = () => {
  const { data: students, isLoading, error } = useStudents();
  const { mutate: deleteStudent } = useDeleteStudent();
  
  const handleDelete = (id: string) => {
    deleteStudent(id);
  };
  
  return (
    <StudentList
      students={students}
      isLoading={isLoading}
      error={error}
      onDelete={handleDelete}
    />
  );
};

// Presentational Component (UI)
interface StudentListProps {
  students: Student[];
  isLoading: boolean;
  error: Error | null;
  onDelete: (id: string) => void;
}

const StudentList: React.FC<StudentListProps> = ({
  students,
  isLoading,
  error,
  onDelete,
}) => {
  if (isLoading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;
  
  return (
    <div className="student-list">
      {students.map(student => (
        <StudentCard
          key={student.id}
          student={student}
          onDelete={onDelete}
        />
      ))}
    </div>
  );
};
```

#### 2. Render Props Pattern
```tsx
// ✅ Good: Flexible component with render props
interface DataFetcherProps<T> {
  queryKey: string[];
  queryFn: () => Promise<T>;
  children: (data: T, isLoading: boolean, error: Error | null) => React.ReactNode;
}

const DataFetcher = <T,>({ queryKey, queryFn, children }: DataFetcherProps<T>) => {
  const { data, isLoading, error } = useQuery({
    queryKey,
    queryFn,
  });
  
  return <>{children(data, isLoading, error)}</>;
};

// Usage
<DataFetcher
  queryKey={['students']}
  queryFn={fetchStudents}
>
  {(students, isLoading, error) => (
    <StudentList
      students={students}
      isLoading={isLoading}
      error={error}
    />
  )}
</DataFetcher>
```

#### 3. Compound Components Pattern
```tsx
// ✅ Good: Compound components for complex UI
interface TableProps {
  children: React.ReactNode;
  className?: string;
}

const Table = ({ children, className }: TableProps) => (
  <table className={cn("table", className)}>
    {children}
  </table>
);

Table.Header = ({ children }: { children: React.ReactNode }) => (
  <thead>{children}</thead>
);

Table.Body = ({ children }: { children: React.ReactNode }) => (
  <tbody>{children}</tbody>
);

Table.Row = ({ children }: { children: React.ReactNode }) => (
  <tr>{children}</tr>
);

// Usage
<Table>
  <Table.Header>
    <tr>
      <th>Name</th>
      <th>NIM</th>
      <th>Actions</th>
    </tr>
  </Table.Header>
  <Table.Body>
    {students.map(student => (
      <Table.Row key={student.id}>
        <td>{student.name}</td>
        <td>{student.nim}</td>
        <td>
          <ActionButtons student={student} />
        </td>
      </Table.Row>
    ))}
  </Table.Body>
</Table>
```

## 🔄 State Management

### Local State Guidelines

#### 1. useState Best Practices
```tsx
// ❌ Bad: Multiple related states
const [name, setName] = useState('');
const [email, setEmail] = useState('');
const [phone, setPhone] = useState('');

// ✅ Good: Grouped related state
const [formData, setFormData] = useState({
  name: '',
  email: '',
  phone: '',
});

const updateFormData = (field: keyof typeof formData, value: string) => {
  setFormData(prev => ({ ...prev, [field]: value }));
};

// ✅ Good: Use reducer for complex state
const [state, dispatch] = useReducer(formReducer, initialState);
```

#### 2. Custom Hooks for State Logic
```tsx
// ✅ Good: Extract state logic into custom hooks
const useForm = <T extends Record<string, any>>(initialState: T) => {
  const [state, setState] = useState<T>(initialState);
  const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});
  
  const updateField = (field: keyof T, value: T[keyof T]) => {
    setState(prev => ({ ...prev, [field]: value }));
    // Clear error when field is updated
    if (errors[field]) {
      setErrors(prev => ({ ...prev, [field]: undefined }));
    }
  };
  
  const validate = (validationSchema: ValidationSchema<T>) => {
    const newErrors = validationSchema.validate(state);
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };
  
  const reset = () => {
    setState(initialState);
    setErrors({});
  };
  
  return {
    state,
    errors,
    updateField,
    validate,
    reset,
  };
};

// Usage
const StudentForm = () => {
  const { state, errors, updateField, validate, reset } = useForm({
    name: '',
    nim: '',
    email: '',
  });
  
  const handleSubmit = () => {
    if (validate(studentValidationSchema)) {
      // Submit form
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <Input
        value={state.name}
        onChange={(e) => updateField('name', e.target.value)}
        error={errors.name}
      />
      {/* Other fields */}
    </form>
  );
};
```

### Global State Management

#### 1. Context API Best Practices
```tsx
// ✅ Good: Split contexts by domain
interface AuthContextType {
  user: User | null;
  login: (credentials: LoginCredentials) => Promise<void>;
  logout: () => void;
  isLoading: boolean;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(false);
  
  const login = async (credentials: LoginCredentials) => {
    setIsLoading(true);
    try {
      const userData = await authService.login(credentials);
      setUser(userData);
    } finally {
      setIsLoading(false);
    }
  };
  
  const logout = () => {
    setUser(null);
    authService.logout();
  };
  
  return (
    <AuthContext.Provider value={{ user, login, logout, isLoading }}>
      {children}
    </AuthContext.Provider>
  );
};

// Custom hook for using context
export const useAuth = () => {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};
```

#### 2. TanStack Query Best Practices
```tsx
// ✅ Good: Centralized query keys
export const queryKeys = {
  students: ['students'] as const,
  student: (id: string) => ['students', id] as const,
  courses: ['courses'] as const,
  course: (id: string) => ['courses', id] as const,
} as const;

// ✅ Good: Custom hooks for queries
export const useStudents = (filters?: StudentFilters) => {
  return useQuery({
    queryKey: [...queryKeys.students, filters],
    queryFn: () => studentService.getStudents(filters),
    staleTime: 5 * 60 * 1000, // 5 minutes
    gcTime: 10 * 60 * 1000, // 10 minutes
  });
};

export const useStudent = (id: string) => {
  return useQuery({
    queryKey: queryKeys.student(id),
    queryFn: () => studentService.getStudent(id),
    enabled: !!id,
  });
};

// ✅ Good: Optimistic updates
export const useUpdateStudent = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: (data: UpdateStudentData) => studentService.updateStudent(data),
    onMutate: async (newStudent) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: queryKeys.student(newStudent.id) });
      
      // Snapshot previous value
      const previousStudent = queryClient.getQueryData(queryKeys.student(newStudent.id));
      
      // Optimistically update
      queryClient.setQueryData(queryKeys.student(newStudent.id), newStudent);
      
      return { previousStudent };
    },
    onError: (err, newStudent, context) => {
      // Rollback on error
      queryClient.setQueryData(queryKeys.student(newStudent.id), context?.previousStudent);
    },
    onSettled: (data, error, variables) => {
      // Always refetch after error or success
      queryClient.invalidateQueries({ queryKey: queryKeys.student(variables.id) });
    },
  });
};
```

## ⚡ Performance Optimization

### React.memo and useMemo

#### 1. When to Use React.memo
```tsx
// ✅ Good: Memoize expensive components
const StudentCard = React.memo<StudentCardProps>(({ student, onEdit, onDelete }) => {
  return (
    <Card>
      <CardHeader>
        <CardTitle>{student.name}</CardTitle>
      </CardHeader>
      <CardContent>
        <p>NIM: {student.nim}</p>
      </CardContent>
      <CardFooter>
        <Button onClick={() => onEdit(student.id)}>Edit</Button>
        <Button onClick={() => onDelete(student.id)}>Delete</Button>
      </CardFooter>
    </Card>
  );
});

// ✅ Good: Memoize callbacks
const StudentList = ({ students }: { students: Student[] }) => {
  const handleEdit = useCallback((id: string) => {
    // Edit logic
  }, []);
  
  const handleDelete = useCallback((id: string) => {
    // Delete logic
  }, []);
  
  return (
    <div>
      {students.map(student => (
        <StudentCard
          key={student.id}
          student={student}
          onEdit={handleEdit}
          onDelete={handleDelete}
        />
      ))}
    </div>
  );
};
```

#### 2. useMemo for Expensive Calculations
```tsx
// ✅ Good: Memoize expensive calculations
const StudentDashboard = ({ students }: { students: Student[] }) => {
  const stats = useMemo(() => {
    return {
      total: students.length,
      active: students.filter(s => s.status === 'active').length,
      averageGPA: students.reduce((sum, s) => sum + s.gpa, 0) / students.length,
    };
  }, [students]);
  
  const sortedStudents = useMemo(() => {
    return [...students].sort((a, b) => a.name.localeCompare(b.name));
  }, [students]);
  
  return (
    <div>
      <StatsDisplay stats={stats} />
      <StudentList students={sortedStudents} />
    </div>
  );
};
```

### Code Splitting and Lazy Loading

#### 1. Route-based Code Splitting
```tsx
// ✅ Good: Lazy load routes
const StudentsPage = lazy(() => import('./pages/Students'));
const CoursesPage = lazy(() => import('./pages/Courses'));
const ReportsPage = lazy(() => import('./pages/Reports'));

const AppRouter = () => {
  return (
    <Routes>
      <Route
        path="/students"
        element={
          <Suspense fallback={<PageLoader />}>
            <StudentsPage />
          </Suspense>
        }
      />
      <Route
        path="/courses"
        element={
          <Suspense fallback={<PageLoader />}>
            <CoursesPage />
          </Suspense>
        }
      />
    </Routes>
  );
};
```

#### 2. Component-based Code Splitting
```tsx
// ✅ Good: Lazy load heavy components
const HeavyChart = lazy(() => import('./components/HeavyChart'));

const Dashboard = () => {
  const [showChart, setShowChart] = useState(false);
  
  return (
    <div>
      <Button onClick={() => setShowChart(true)}>
        Show Chart
      </Button>
      
      {showChart && (
        <Suspense fallback={<ChartLoader />}>
          <HeavyChart />
        </Suspense>
      )}
    </div>
  );
};
```

## 🔷 TypeScript Integration

### Type Safety Best Practices

#### 1. Strict Type Definitions
```tsx
// ✅ Good: Strict interfaces
interface Student {
  id: string;
  name: string;
  nim: string;
  email: string;
  programStudi: string;
  status: 'active' | 'inactive' | 'graduated';
  gpa: number;
  createdAt: Date;
  updatedAt: Date;
}

// ✅ Good: Generic types for reusable components
interface DataTableProps<T> {
  data: T[];
  columns: Column<T>[];
  onRowClick?: (item: T) => void;
  loading?: boolean;
  error?: Error | null;
}

const DataTable = <T extends Record<string, any>>({
  data,
  columns,
  onRowClick,
  loading,
  error,
}: DataTableProps<T>) => {
  // Implementation
};
```

#### 2. Utility Types
```tsx
// ✅ Good: Use utility types
type CreateStudentInput = Omit<Student, 'id' | 'createdAt' | 'updatedAt'>;
type UpdateStudentInput = Partial<CreateStudentInput> & { id: string };
type StudentFilters = Pick<Student, 'status' | 'programStudi'>;

// ✅ Good: Discriminated unions
type ApiResponse<T> = 
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };

const useApiData = <T>(queryFn: () => Promise<T>): ApiResponse<T> => {
  const { data, isLoading, error } = useQuery({
    queryKey: ['api-data'],
    queryFn,
  });
  
  if (isLoading) return { status: 'loading' };
  if (error) return { status: 'error', error };
  return { status: 'success', data: data! };
};
```

## 📁 Code Organization

### File Structure Guidelines

```
src/
├── components/
│   ├── ui/                    # Reusable UI components
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   ├── Button.test.tsx
│   │   │   └── index.ts
│   │   └── index.ts
│   ├── forms/                 # Form components
│   │   ├── StudentForm/
│   │   │   ├── StudentForm.tsx
│   │   │   ├── StudentForm.test.tsx
│   │   │   ├── validation.ts
│   │   │   └── index.ts
│   │   └── index.ts
│   └── layout/                # Layout components
│       ├── Sidebar/
│       ├── Header/
│       └── Footer/
├── pages/                     # Page components
│   ├── Students/
│   │   ├── StudentsPage.tsx
│   │   ├── components/
│   │   │   ├── StudentList.tsx
│   │   │   └── StudentFilters.tsx
│   │   └── index.ts
│   └── index.ts
├── hooks/                     # Custom hooks
│   ├── useStudents.ts
│   ├── useForm.ts
│   └── index.ts
├── services/                  # API services
│   ├── studentService.ts
│   ├── courseService.ts
│   └── index.ts
├── types/                     # TypeScript types
│   ├── student.ts
│   ├── course.ts
│   └── index.ts
├── utils/                     # Utility functions
│   ├── validation.ts
│   ├── formatting.ts
│   └── index.ts
└── constants/                 # Constants
    ├── api.ts
    ├── routes.ts
    └── index.ts
```

## 🏷️ Naming Conventions

### 1. File and Folder Naming

#### 1.1 Component Files
```tsx
// ✅ Good: Component file naming conventions
// React Components: PascalCase
StudentCard.tsx
StudentList.tsx
Dashboard.tsx
UserProfile.tsx
CourseEnrollment.tsx

// Page Components: PascalCase with 'Page' suffix
StudentsPage.tsx
CoursesPage.tsx
LoginPage.tsx
DashboardPage.tsx
ProfilePage.tsx

// Layout Components: PascalCase with 'Layout' suffix
MainLayout.tsx
SidebarLayout.tsx
AuthLayout.tsx
DashboardLayout.tsx

// Feature Components: PascalCase with feature prefix
StudentCard.tsx
StudentForm.tsx
StudentList.tsx
StudentFilters.tsx
StudentStats.tsx

// Modal/Dialog Components: PascalCase with 'Modal' or 'Dialog' suffix
StudentModal.tsx
ConfirmDialog.tsx
EditStudentModal.tsx
DeleteConfirmationDialog.tsx
```

#### 1.2 Utility and Service Files
```tsx
// ✅ Good: Utility file naming conventions
// Utility functions: camelCase
formatDate.ts
validationUtils.ts
apiHelpers.ts
stringUtils.ts
numberUtils.ts
dateUtils.ts
arrayUtils.ts
objectUtils.ts

// Constants: camelCase
constants.ts
apiEndpoints.ts
routes.ts
config.ts
appConfig.ts
themeConfig.ts

// Types: camelCase
types.ts
studentTypes.ts
courseTypes.ts
apiTypes.ts
formTypes.ts
validationTypes.ts

// Hooks: camelCase with 'use' prefix
useStudents.ts
useForm.ts
useAuth.ts
useLocalStorage.ts
useDebounce.ts
usePrevious.ts
useClickOutside.ts
useWindowSize.ts

// Services: camelCase with 'Service' suffix
studentService.ts
courseService.ts
authService.ts
apiService.ts
fileService.ts
notificationService.ts

// API related: camelCase
apiClient.ts
apiConfig.ts
apiMiddleware.ts
apiInterceptors.ts
```

#### 1.3 Test Files
```tsx
// ✅ Good: Test file naming conventions
// Unit tests: same name as source + .test
StudentCard.test.tsx
useStudents.test.ts
studentService.test.ts
formatDate.test.ts
validationUtils.test.ts

// Integration tests: .integration.test
studentApi.integration.test.ts
authFlow.integration.test.ts
formSubmission.integration.test.ts

// E2E tests: .e2e.test
studentManagement.e2e.test.ts
loginFlow.e2e.test.ts
courseEnrollment.e2e.test.ts

// Test utilities: camelCase with 'test' prefix
testUtils.ts
testHelpers.ts
testData.ts
mockData.ts
testSetup.ts
```

#### 1.4 Configuration Files
```tsx
// ✅ Good: Configuration file naming conventions
// Build tools: kebab-case
vite.config.ts
webpack.config.js
rollup.config.js
babel.config.js
postcss.config.js

// Package managers: kebab-case
package.json
package-lock.json
yarn.lock
pnpm-lock.yaml

// TypeScript: kebab-case
tsconfig.json
tsconfig.node.json
tsconfig.app.json
tsconfig.test.json

// Linting and formatting: kebab-case
.eslintrc.js
.prettierrc
.stylelintrc
.editorconfig

// Environment: kebab-case
.env
.env.local
.env.development
.env.staging
.env.production
.env.test
```

#### 1.5 Asset Files
```tsx
// ✅ Good: Asset file naming conventions
// Images: kebab-case
hero-image.png
student-avatar.jpg
course-banner.svg
logo-white.png
icon-arrow-right.svg

// Icons: kebab-case with 'icon' prefix
icon-home.svg
icon-user.svg
icon-settings.svg
icon-edit.svg
icon-delete.svg

// Styles: kebab-case
main.css
global-styles.css
component-styles.css
theme-dark.css
theme-light.css

// Fonts: kebab-case
roboto-regular.woff2
roboto-bold.woff2
inter-regular.woff2
inter-medium.woff2
```

### 2. Folder Structure Naming

#### 2.1 Main Application Folders
```tsx
// ✅ Good: Main folder naming conventions
src/
├── components/           # camelCase - Reusable UI components
├── pages/               # lowercase - Page components
├── hooks/               # lowercase - Custom React hooks
├── services/            # lowercase - API and external services
├── types/               # lowercase - TypeScript type definitions
├── utils/               # lowercase - Utility functions
├── constants/           # lowercase - Application constants
├── assets/              # lowercase - Static assets
├── styles/              # lowercase - Global styles
├── config/              # lowercase - Configuration files
├── store/               # lowercase - State management
├── router/              # lowercase - Routing configuration
├── middleware/          # lowercase - Custom middleware
├── guards/              # lowercase - Route guards
├── interceptors/        # lowercase - HTTP interceptors
└── __tests__/           # lowercase with underscores - Test files
```

#### 2.2 Component Organization Folders
```tsx
// ✅ Good: Component folder naming conventions
src/components/
├── ui/                  # lowercase - Base UI components
│   ├── Button/
│   ├── Input/
│   ├── Modal/
│   ├── Table/
│   └── Card/
├── forms/               # lowercase - Form components
│   ├── StudentForm/
│   ├── CourseForm/
│   ├── LoginForm/
│   └── SearchForm/
├── layout/              # lowercase - Layout components
│   ├── Header/
│   ├── Sidebar/
│   ├── Footer/
│   └── Navigation/
├── features/            # lowercase - Feature-specific components
│   ├── students/
│   ├── courses/
│   ├── auth/
│   └── dashboard/
└── shared/              # lowercase - Shared components
    ├── Loading/
    ├── ErrorBoundary/
    ├── ConfirmDialog/
    └── Pagination/
```

#### 2.3 Page Organization Folders
```tsx
// ✅ Good: Page folder naming conventions
src/pages/
├── students/            # lowercase - Student-related pages
│   ├── StudentsPage.tsx
│   ├── StudentDetailPage.tsx
│   ├── StudentCreatePage.tsx
│   ├── StudentEditPage.tsx
│   └── components/      # Page-specific components
│       ├── StudentFilters.tsx
│       ├── StudentStats.tsx
│       └── StudentActions.tsx
├── courses/             # lowercase - Course-related pages
│   ├── CoursesPage.tsx
│   ├── CourseDetailPage.tsx
│   ├── CourseCreatePage.tsx
│   └── components/
├── auth/                # lowercase - Authentication pages
│   ├── LoginPage.tsx
│   ├── RegisterPage.tsx
│   ├── ForgotPasswordPage.tsx
│   └── components/
├── dashboard/           # lowercase - Dashboard pages
│   ├── DashboardPage.tsx
│   ├── AnalyticsPage.tsx
│   └── components/
└── settings/            # lowercase - Settings pages
    ├── ProfilePage.tsx
    ├── PreferencesPage.tsx
    └── components/
```

#### 2.4 Feature-based Organization
```tsx
// ✅ Good: Feature-based folder naming conventions
src/features/
├── students/            # lowercase - Student feature
│   ├── components/
│   │   ├── StudentCard.tsx
│   │   ├── StudentList.tsx
│   │   ├── StudentForm.tsx
│   │   └── StudentFilters.tsx
│   ├── hooks/
│   │   ├── useStudents.ts
│   │   ├── useStudentForm.ts
│   │   └── useStudentFilters.ts
│   ├── services/
│   │   └── studentService.ts
│   ├── types/
│   │   └── studentTypes.ts
│   ├── utils/
│   │   └── studentUtils.ts
│   └── pages/
│       ├── StudentsPage.tsx
│       ├── StudentDetailPage.tsx
│       └── StudentCreatePage.tsx
├── courses/             # lowercase - Course feature
│   ├── components/
│   ├── hooks/
│   ├── services/
│   ├── types/
│   ├── utils/
│   └── pages/
├── auth/                # lowercase - Authentication feature
│   ├── components/
│   ├── hooks/
│   ├── services/
│   ├── types/
│   ├── utils/
│   └── pages/
└── dashboard/           # lowercase - Dashboard feature
    ├── components/
    ├── hooks/
    ├── services/
    ├── types/
    ├── utils/
    └── pages/
```

#### 2.5 Asset Organization Folders
```tsx
// ✅ Good: Asset folder naming conventions
src/assets/
├── images/              # lowercase - Image files
│   ├── logos/           # lowercase - Logo images
│   │   ├── logo-primary.png
│   │   ├── logo-white.png
│   │   └── logo-dark.png
│   ├── icons/           # lowercase - Icon images
│   │   ├── ui/          # lowercase - UI icons
│   │   │   ├── home.svg
│   │   │   ├── user.svg
│   │   │   └── settings.svg
│   │   ├── social/      # lowercase - Social media icons
│   │   │   ├── facebook.svg
│   │   │   ├── twitter.svg
│   │   │   └── linkedin.svg
│   │   └── flags/       # lowercase - Country flags
│   │       ├── id.svg
│   │       ├── us.svg
│   │       └── gb.svg
│   ├── backgrounds/     # lowercase - Background images
│   │   ├── hero-bg.jpg
│   │   ├── pattern-bg.svg
│   │   └── gradient-bg.png
│   └── placeholders/    # lowercase - Placeholder images
│       ├── avatar-placeholder.png
│       ├── image-placeholder.jpg
│       └── card-placeholder.svg
├── fonts/               # lowercase - Font files
│   ├── roboto/
│   │   ├── roboto-regular.woff2
│   │   ├── roboto-medium.woff2
│   │   └── roboto-bold.woff2
│   └── inter/
│       ├── inter-regular.woff2
│       ├── inter-medium.woff2
│       └── inter-bold.woff2
├── styles/              # lowercase - Style files
│   ├── global.css
│   ├── variables.css
│   ├── animations.css
│   └── themes/
│       ├── light.css
│       └── dark.css
└── data/                # lowercase - Static data files
    ├── countries.json
    ├── universities.json
    ├── courses.json
    └── mock-data.json
```

#### 2.6 Configuration Organization Folders
```tsx
// ✅ Good: Configuration folder naming conventions
src/config/
├── api/                 # lowercase - API configuration
│   ├── endpoints.ts
│   ├── interceptors.ts
│   ├── middleware.ts
│   └── types.ts
├── auth/                # lowercase - Authentication configuration
│   ├── guards.ts
│   ├── strategies.ts
│   ├── permissions.ts
│   └── types.ts
├── theme/               # lowercase - Theme configuration
│   ├── colors.ts
│   ├── typography.ts
│   ├── spacing.ts
│   └── breakpoints.ts
├── validation/          # lowercase - Validation configuration
│   ├── schemas.ts
│   ├── rules.ts
│   ├── messages.ts
│   └── types.ts
└── constants/           # lowercase - Application constants
    ├── app.ts
    ├── routes.ts
    ├── status.ts
    └── messages.ts
```

#### 2.7 Test Organization Folders
```tsx
// ✅ Good: Test folder naming conventions
src/__tests__/
├── components/          # lowercase - Component tests
│   ├── ui/
│   │   ├── Button.test.tsx
│   │   ├── Input.test.tsx
│   │   └── Modal.test.tsx
│   ├── forms/
│   │   ├── StudentForm.test.tsx
│   │   └── LoginForm.test.tsx
│   └── layout/
│       ├── Header.test.tsx
│       └── Sidebar.test.tsx
├── hooks/               # lowercase - Hook tests
│   ├── useStudents.test.ts
│   ├── useForm.test.ts
│   └── useAuth.test.ts
├── services/            # lowercase - Service tests
│   ├── studentService.test.ts
│   ├── authService.test.ts
│   └── apiService.test.ts
├── utils/               # lowercase - Utility tests
│   ├── formatDate.test.ts
│   ├── validationUtils.test.ts
│   └── apiHelpers.test.ts
├── integration/         # lowercase - Integration tests
│   ├── studentApi.integration.test.ts
│   ├── authFlow.integration.test.ts
│   └── formSubmission.integration.test.ts
├── e2e/                 # lowercase - E2E tests
│   ├── studentManagement.e2e.test.ts
│   ├── loginFlow.e2e.test.ts
│   └── courseEnrollment.e2e.test.ts
├── fixtures/            # lowercase - Test data
│   ├── students.json
│   ├── courses.json
│   └── users.json
├── mocks/               # lowercase - Mock files
│   ├── apiMocks.ts
│   ├── componentMocks.ts
│   └── serviceMocks.ts
└── setup/               # lowercase - Test setup
    ├── testUtils.ts
    ├── testHelpers.ts
    └── testSetup.ts
```

#### 2.8 Documentation Organization Folders
```tsx
// ✅ Good: Documentation folder naming conventions
docs/
├── components/          # lowercase - Component documentation
│   ├── Button.md
│   ├── Input.md
│   ├── Modal.md
│   └── StudentCard.md
├── hooks/               # lowercase - Hook documentation
│   ├── useStudents.md
│   ├── useForm.md
│   └── useAuth.md
├── services/            # lowercase - Service documentation
│   ├── studentService.md
│   ├── authService.md
│   └── apiService.md
├── api/                 # lowercase - API documentation
│   ├── endpoints.md
│   ├── authentication.md
│   ├── errors.md
│   └── examples.md
├── guides/              # lowercase - Development guides
│   ├── getting-started.md
│   ├── component-guidelines.md
│   ├── testing-guide.md
│   └── deployment-guide.md
├── examples/            # lowercase - Code examples
│   ├── forms/
│   ├── tables/
│   ├── modals/
│   └── charts/
└── assets/              # lowercase - Documentation assets
    ├── images/
    ├── diagrams/
    └── screenshots/
```

### 3. Function and Variable Naming

```tsx
// ✅ Good: Function naming conventions
// Components: PascalCase
const StudentCard = () => {};
const StudentListContainer = () => {};

// Hooks: camelCase with 'use' prefix
const useStudents = () => {};
const useStudentForm = () => {};
const useLocalStorage = () => {};

// Event handlers: camelCase with 'handle' prefix
const handleSubmit = () => {};
const handleClick = () => {};
const handleInputChange = () => {};

// Async functions: camelCase with descriptive names
const fetchStudents = async () => {};
const createStudent = async () => {};
const updateStudentData = async () => {};

// Utility functions: camelCase with descriptive names
const formatDate = (date: Date) => {};
const validateEmail = (email: string) => {};
const calculateGPA = (grades: number[]) => {};

// Boolean functions: camelCase with 'is', 'has', 'can' prefix
const isValidEmail = (email: string) => {};
const hasPermission = (permission: string) => {};
const canEditStudent = (student: Student) => {};

// Variables: camelCase
const studentName = 'John Doe';
const isEditing = false;
const selectedStudents = [];
const apiEndpoint = '/api/students';
```

### 4. Constants and Configuration Naming

```tsx
// ✅ Good: Constants naming conventions
// API endpoints: UPPER_SNAKE_CASE
const API_ENDPOINTS = {
  STUDENTS: '/api/students',
  COURSES: '/api/courses',
  AUTH: '/api/auth',
};

// Environment variables: UPPER_SNAKE_CASE
const API_BASE_URL = process.env.VITE_API_URL;
const APP_NAME = process.env.VITE_APP_NAME;

// Configuration objects: UPPER_SNAKE_CASE
const APP_CONFIG = {
  MAX_FILE_SIZE: 5 * 1024 * 1024, // 5MB
  SUPPORTED_FORMATS: ['jpg', 'png', 'pdf'],
  PAGINATION_LIMIT: 20,
};

// Route constants: UPPER_SNAKE_CASE
const ROUTES = {
  HOME: '/',
  STUDENTS: '/students',
  COURSES: '/courses',
  LOGIN: '/login',
};

// Status constants: UPPER_SNAKE_CASE
const STUDENT_STATUS = {
  ACTIVE: 'active',
  INACTIVE: 'inactive',
  GRADUATED: 'graduated',
} as const;
```

### 5. Type and Interface Naming

```tsx
// ✅ Good: Type naming conventions
// Interfaces: PascalCase with descriptive names
interface StudentCardProps {
  student: Student;
  onEdit?: (id: string) => void;
  onDelete?: (id: string) => void;
}

interface ApiResponse<T> {
  data: T;
  message: string;
  success: boolean;
}

// Types: PascalCase with descriptive names
type StudentStatus = 'active' | 'inactive' | 'graduated';
type ApiError = {
  code: string;
  message: string;
  details?: Record<string, any>;
};

// Generic types: PascalCase with T, K, V parameters
type ApiResponse<T> = {
  data: T;
  status: 'success' | 'error';
};

type FormField<T> = {
  value: T;
  error?: string;
  touched: boolean;
};
```

### 6. CSS Class Naming

```tsx
// ✅ Good: CSS class naming conventions
// BEM methodology or kebab-case
const StudentCard = () => {
  return (
    <div className="student-card">
      <div className="student-card__header">
        <h3 className="student-card__title">John Doe</h3>
      </div>
      <div className="student-card__content">
        <p className="student-card__nim">NIM: 123456</p>
      </div>
      <div className="student-card__actions">
        <button className="student-card__edit-btn">Edit</button>
        <button className="student-card__delete-btn">Delete</button>
      </div>
    </div>
  );
};

// Or using Tailwind CSS classes
const StudentCard = () => {
  return (
    <div className="bg-white rounded-lg shadow-md p-4">
      <div className="flex items-center justify-between mb-2">
        <h3 className="text-lg font-semibold text-gray-900">John Doe</h3>
      </div>
      <div className="space-y-1">
        <p className="text-sm text-gray-600">NIM: 123456</p>
      </div>
      <div className="flex gap-2 mt-4">
        <button className="btn btn-primary">Edit</button>
        <button className="btn btn-danger">Delete</button>
      </div>
    </div>
  );
};
```

### 7. Import/Export Naming

```tsx
// ✅ Good: Import/Export naming conventions
// Named exports: PascalCase for components, camelCase for others
export const StudentCard = () => {};
export const useStudents = () => {};
export const studentService = {};

// Default exports: PascalCase for components
export default StudentCard;

// Index files: re-export with clear names
// components/index.ts
export { StudentCard } from './StudentCard';
export { StudentList } from './StudentList';
export { StudentForm } from './StudentForm';

// Import with aliases when needed
import { StudentCard as StudentCardComponent } from './components';
import { useStudents as useStudentData } from './hooks';
```

### 8. Database and API Naming

```tsx
// ✅ Good: Database naming conventions
// Tables: snake_case
const students = 'students';
const student_courses = 'student_courses';
const course_enrollments = 'course_enrollments';

// Columns: snake_case
const studentData = {
  id: '1',
  first_name: 'John',
  last_name: 'Doe',
  student_id: '123456',
  email_address: 'john@example.com',
  created_at: '2024-01-01',
  updated_at: '2024-01-01',
};

// API endpoints: kebab-case
const API_ENDPOINTS = {
  GET_STUDENTS: '/api/students',
  GET_STUDENT_BY_ID: '/api/students/:id',
  CREATE_STUDENT: '/api/students',
  UPDATE_STUDENT: '/api/students/:id',
  DELETE_STUDENT: '/api/students/:id',
  GET_STUDENT_COURSES: '/api/students/:id/courses',
};
```

### 9. Error and Exception Naming

```tsx
// ✅ Good: Error naming conventions
// Error classes: PascalCase with 'Error' suffix
class ValidationError extends Error {
  constructor(message: string, public field: string) {
    super(message);
    this.name = 'ValidationError';
  }
}

class ApiError extends Error {
  constructor(message: string, public statusCode: number) {
    super(message);
    this.name = 'ApiError';
  }
}

// Error constants: UPPER_SNAKE_CASE
const ERROR_MESSAGES = {
  REQUIRED_FIELD: 'This field is required',
  INVALID_EMAIL: 'Please enter a valid email address',
  NETWORK_ERROR: 'Network error occurred',
  UNAUTHORIZED: 'You are not authorized to perform this action',
};

// Error handling functions: camelCase
const handleApiError = (error: ApiError) => {};
const validateFormData = (data: FormData) => {};
const showErrorMessage = (message: string) => {};
```

### 10. Testing Naming

```tsx
// ✅ Good: Testing naming conventions
// Test files: same name as source + .test
describe('StudentCard', () => {
  // Test descriptions: descriptive and clear
  it('renders student information correctly', () => {});
  it('calls onEdit when edit button is clicked', () => {});
  it('displays loading state when data is loading', () => {});
  it('shows error message when API call fails', () => {});
});

// Test utilities: camelCase with 'test' prefix
const createTestStudent = (overrides = {}) => ({
  id: '1',
  name: 'Test Student',
  nim: '123456',
  ...overrides,
});

const renderWithProviders = (component: React.ReactElement) => {};
const mockApiResponse = (data: any) => {};
```

### 11. Documentation Naming

```tsx
// ✅ Good: Documentation naming conventions
// JSDoc comments: descriptive and clear
/**
 * StudentCard - Displays student information in a card format
 * 
 * @component
 * @param {StudentCardProps} props - Component props
 * @param {Student} props.student - Student data to display
 * @param {Function} props.onEdit - Edit callback function
 * @param {Function} props.onDelete - Delete callback function
 * @returns {JSX.Element} Student card component
 * 
 * @example
 * ```tsx
 * <StudentCard
 *   student={studentData}
 *   onEdit={(id) => handleEdit(id)}
 *   onDelete={(id) => handleDelete(id)}
 * />
 * ```
 */

// README files: README.md
// Component documentation: ComponentName.md
// API documentation: api.md
// Setup documentation: setup.md
```

### 12. Environment and Configuration Naming

```tsx
// ✅ Good: Environment naming conventions
// Environment files: .env.{environment}
.env.development
.env.staging
.env.production
.env.local

// Environment variables: UPPER_SNAKE_CASE
VITE_API_URL=http://localhost:3001
VITE_APP_NAME=SIAKAD
VITE_ENVIRONMENT=development
VITE_DEBUG_MODE=true

// Configuration files: kebab-case
vite.config.ts
tailwind.config.js
tsconfig.json
package.json
```

### 13. Git and Version Control Naming

```tsx
// ✅ Good: Git naming conventions
// Branches: kebab-case with prefixes
feature/add-student-management
bugfix/fix-login-validation
hotfix/critical-api-fix
release/v1.2.0

// Commits: conventional commits
feat: add student management functionality
fix: resolve login validation issue
docs: update API documentation
test: add unit tests for StudentCard component
refactor: improve error handling in API calls

// Tags: semantic versioning
v1.0.0
v1.1.0
v1.1.1
```

### 14. Package and Dependency Naming

```tsx
// ✅ Good: Package naming conventions
// Package name: kebab-case
"@siakad/ui-components"
"@siakad/api-client"
"@siakad/utils"

// Script names: kebab-case
"build:production"
"test:coverage"
"lint:fix"
"type-check"

// Dependency categories
"dependencies": {
  "react": "^18.0.0",
  "react-dom": "^18.0.0"
},
"devDependencies": {
  "@types/react": "^18.0.0",
  "typescript": "^5.0.0"
},
"peerDependencies": {
  "react": ">=16.8.0"
}
```

## 🧪 Testing Patterns

### Component Testing Best Practices

#### 1. Test Structure
```tsx
// ✅ Good: Organized test structure
describe('StudentCard', () => {
  const defaultProps: StudentCardProps = {
    student: {
      id: '1',
      name: 'John Doe',
      nim: '123456',
      email: 'john@example.com',
      programStudi: 'Informatika',
      status: 'active',
      gpa: 3.5,
      createdAt: new Date(),
      updatedAt: new Date(),
    },
  };
  
  describe('rendering', () => {
    it('renders student information correctly', () => {
      render(<StudentCard {...defaultProps} />);
      
      expect(screen.getByText('John Doe')).toBeInTheDocument();
      expect(screen.getByText('NIM: 123456')).toBeInTheDocument();
      expect(screen.getByText('Informatika')).toBeInTheDocument();
    });
    
    it('shows correct status badge', () => {
      render(<StudentCard {...defaultProps} />);
      
      const statusBadge = screen.getByText('Active');
      expect(statusBadge).toHaveClass('badge--active');
    });
  });
  
  describe('interactions', () => {
    it('calls onEdit when edit button is clicked', () => {
      const onEdit = jest.fn();
      render(<StudentCard {...defaultProps} onEdit={onEdit} />);
      
      fireEvent.click(screen.getByRole('button', { name: /edit/i }));
      expect(onEdit).toHaveBeenCalledWith('1');
    });
    
    it('calls onDelete when delete button is clicked', () => {
      const onDelete = jest.fn();
      render(<StudentCard {...defaultProps} onDelete={onDelete} />);
      
      fireEvent.click(screen.getByRole('button', { name: /delete/i }));
      expect(onDelete).toHaveBeenCalledWith('1');
    });
  });
  
  describe('edge cases', () => {
    it('handles missing optional props', () => {
      render(<StudentCard {...defaultProps} onEdit={undefined} onDelete={undefined} />);
      
      expect(screen.getByText('John Doe')).toBeInTheDocument();
    });
    
    it('displays loading state when student data is loading', () => {
      render(<StudentCard {...defaultProps} isLoading={true} />);
      
      expect(screen.getByTestId('loading-skeleton')).toBeInTheDocument();
    });
  });
});
```

#### 2. Custom Render Function
```tsx
// ✅ Good: Custom render with providers
const AllTheProviders = ({ children }: { children: React.ReactNode }) => {
  return (
    <QueryClientProvider client={queryClient}>
      <AuthProvider>
        <ThemeProvider>
          {children}
        </ThemeProvider>
      </AuthProvider>
    </QueryClientProvider>
  );
};

const customRender = (ui: React.ReactElement, options = {}) =>
  render(ui, { wrapper: AllTheProviders, ...options });

// Usage in tests
test('renders student list', () => {
  customRender(<StudentList />);
  expect(screen.getByText('Students')).toBeInTheDocument();
});
```

## ♿ Accessibility

### Accessibility Best Practices

#### 1. Semantic HTML
```tsx
// ✅ Good: Use semantic HTML elements
const StudentList = ({ students }: { students: Student[] }) => {
  return (
    <section aria-labelledby="students-heading">
      <h2 id="students-heading">Students</h2>
      <ul role="list">
        {students.map(student => (
          <li key={student.id}>
            <StudentCard student={student} />
          </li>
        ))}
      </ul>
    </section>
  );
};

// ✅ Good: Proper form labels
const StudentForm = () => {
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="name">Full Name</label>
        <input
          id="name"
          name="name"
          type="text"
          required
          aria-describedby="name-help"
        />
        <p id="name-help">Enter the student's full name as it appears on official documents</p>
      </div>
    </form>
  );
};
```

#### 2. Keyboard Navigation
```tsx
// ✅ Good: Keyboard accessible components
const StudentCard = ({ student, onEdit, onDelete }: StudentCardProps) => {
  const handleKeyDown = (event: React.KeyboardEvent) => {
    if (event.key === 'Enter' || event.key === ' ') {
      event.preventDefault();
      onEdit?.(student.id);
    }
  };
  
  return (
    <div
      role="button"
      tabIndex={0}
      onKeyDown={handleKeyDown}
      onClick={() => onEdit?.(student.id)}
      aria-label={`Edit student ${student.name}`}
    >
      <h3>{student.name}</h3>
      <p>NIM: {student.nim}</p>
    </div>
  );
};
```

## 🔒 Security

### Security Best Practices

#### 1. Input Sanitization
```tsx
// ✅ Good: Sanitize user inputs
import DOMPurify from 'dompurify';

const StudentComment = ({ comment }: { comment: string }) => {
  const sanitizedComment = DOMPurify.sanitize(comment);
  
  return (
    <div
      dangerouslySetInnerHTML={{ __html: sanitizedComment }}
      className="comment-content"
    />
  );
};

// ✅ Good: Validate inputs
const useFormValidation = <T extends Record<string, any>>(schema: ValidationSchema<T>) => {
  const validate = (data: T): ValidationErrors<T> => {
    return schema.validate(data);
  };
  
  return { validate };
};
```

#### 2. XSS Prevention
```tsx
// ❌ Bad: Direct innerHTML usage
const UserContent = ({ content }: { content: string }) => (
  <div dangerouslySetInnerHTML={{ __html: content }} />
);

// ✅ Good: Safe content rendering
const UserContent = ({ content }: { content: string }) => (
  <div>{content}</div>
);

// ✅ Good: Use React's built-in XSS protection
const UserContent = ({ content }: { content: string }) => {
  // React automatically escapes content
  return <div>{content}</div>;
};
```

## 🚨 Error Handling

### Error Boundary Pattern
```tsx
// ✅ Good: Error boundary component
class ErrorBoundary extends React.Component<
  { children: React.ReactNode; fallback?: React.ComponentType<{ error: Error }> },
  { hasError: boolean; error: Error | null }
> {
  constructor(props: any) {
    super(props);
    this.state = { hasError: false, error: null };
  }
  
  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }
  
  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
    // Log to error reporting service
    errorReportingService.captureException(error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      const FallbackComponent = this.props.fallback || DefaultErrorFallback;
      return <FallbackComponent error={this.state.error!} />;
    }
    
    return this.props.children;
  }
}

// Usage
<ErrorBoundary fallback={CustomErrorFallback}>
  <StudentDashboard />
</ErrorBoundary>
```

### Async Error Handling
```tsx
// ✅ Good: Handle async errors gracefully
const useAsyncOperation = <T,>(
  operation: () => Promise<T>
) => {
  const [state, setState] = useState<{
    data: T | null;
    loading: boolean;
    error: Error | null;
  }>({
    data: null,
    loading: false,
    error: null,
  });
  
  const execute = useCallback(async () => {
    setState(prev => ({ ...prev, loading: true, error: null }));
    
    try {
      const result = await operation();
      setState({ data: result, loading: false, error: null });
    } catch (error) {
      setState({ 
        data: null, 
        loading: false, 
        error: error instanceof Error ? error : new Error('Unknown error') 
      });
    }
  }, [operation]);
  
  return { ...state, execute };
};
```

## 📚 Documentation

### Component Documentation
```tsx
/**
 * StudentCard - Displays student information in a card format
 * 
 * @component
 * @example
 * ```tsx
 * <StudentCard
 *   student={studentData}
 *   onEdit={(id) => handleEdit(id)}
 *   onDelete={(id) => handleDelete(id)}
 * />
 * ```
 */
interface StudentCardProps {
  /** Student data to display */
  student: Student;
  /** Callback function when edit button is clicked */
  onEdit?: (id: string) => void;
  /** Callback function when delete button is clicked */
  onDelete?: (id: string) => void;
  /** Visual variant of the card */
  variant?: 'default' | 'compact' | 'detailed';
  /** Whether to show action buttons */
  showActions?: boolean;
  /** Additional CSS classes */
  className?: string;
}

const StudentCard: React.FC<StudentCardProps> = ({
  student,
  onEdit,
  onDelete,
  variant = 'default',
  showActions = true,
  className,
}) => {
  // Component implementation
};
```

### README Documentation
```markdown
# StudentCard Component

A reusable card component for displaying student information.

## Features

- Displays student name, NIM, and program studi
- Supports different visual variants
- Optional action buttons (edit/delete)
- Fully accessible with keyboard navigation
- Responsive design

## Usage

```tsx
import { StudentCard } from '@/components/StudentCard';

<StudentCard
  student={studentData}
  onEdit={(id) => handleEdit(id)}
  onDelete={(id) => handleDelete(id)}
  variant="detailed"
  showActions={true}
/>
```

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| student | Student | - | Student data to display |
| onEdit | (id: string) => void | - | Edit callback function |
| onDelete | (id: string) => void | - | Delete callback function |
| variant | 'default' \| 'compact' \| 'detailed' | 'default' | Visual variant |
| showActions | boolean | true | Show action buttons |
| className | string | - | Additional CSS classes |

## Accessibility

- Uses semantic HTML elements
- Supports keyboard navigation
- Includes proper ARIA labels
- Screen reader friendly
```

## 📋 Checklist

### Before Committing Code

- [ ] Components follow single responsibility principle
- [ ] Props interfaces are properly defined
- [ ] TypeScript types are strict and accurate
- [ ] Components are memoized when appropriate
- [ ] Custom hooks are used for complex logic
- [ ] Error boundaries are in place
- [ ] Accessibility features are implemented
- [ ] Tests cover main functionality and edge cases
- [ ] Code is properly documented
- [ ] Performance is optimized
- [ ] Security best practices are followed
- [ ] Naming conventions are followed

### Code Review Checklist

- [ ] Code follows established patterns
- [ ] No prop drilling issues
- [ ] State management is appropriate
- [ ] Error handling is comprehensive
- [ ] Accessibility requirements are met
- [ ] Performance considerations are addressed
- [ ] Security vulnerabilities are avoided
- [ ] Tests are meaningful and comprehensive
- [ ] Documentation is clear and complete
- [ ] Naming conventions are consistent
