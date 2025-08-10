
# Frontend Development Guide

## 🛠️ Setup Development Environment

### Prerequisites
- Node.js 18+
- npm atau yarn
- Git

### Initial Setup
```bash
# Clone repository
git clone <repository-url>
cd siakad

# Install dependencies
npm install

# Setup frontend
cd apps/frontend
npm install
```

### Environment Configuration
```bash
# Copy environment file
cp .env.example .env.local

# Configure environment variables
VITE_API_URL=http://localhost:3001/api/v1
VITE_APP_NAME=SIAKAD
```

---

## 🏗️ Project Structure
```
apps/frontend/
├── src/
│   ├── components/           # Reusable components
│   │   ├── ui/              # shadcn/ui components
│   │   ├── forms/           # Form components
│   │   └── layout/          # Layout components
│   ├── pages/               # Page components
│   │   ├── Students/        # Student pages
|   |   |   ├── components/
|   |   |   ├── StudentPage.tsx
│   │   ├── Courses/         # Course pages
|   |   |   ├── components/
|   |   |   ├── CoursePage.tsx
│   │   └── Dashboard/       # Dashboard pages
|   |   |   ├── components/
|   |   |   ├── DashboardPage.tsx
│   ├── router/              # TanStack Router config
│   ├── lib/                 # Utility functions
│   ├── types/               # TypeScript types
│   ├── hooks/               # Custom React hooks
│   ├── store/               # Zustand stores
│   └── styles/              # Global styles
├── public/                  # Static assets
└── index.html               # Entry point
```

---

## 🎨 Component Architecture

### Component Guidelines
- **Single Responsibility**: Setiap komponen memiliki satu tanggung jawab.
- **Composition over Inheritance**: Gunakan composition pattern.
- **Props Interface**: Definisikan interface untuk props.

### Example Component Structure
```tsx
// components/StudentCard.tsx
interface StudentCardProps {
  student: Student;
  onEdit?: (id: string) => void;
  onDelete?: (id: string) => void;
}

export const StudentCard: React.FC<StudentCardProps> = ({
  student,
  onEdit,
  onDelete
}) => {
  return (
    <Card>
      <CardHeader>
        <CardTitle>{student.name}</CardTitle>
      </CardHeader>
      <CardContent>
        <p>NIM: {student.nim}</p>
        <p>Program Studi: {student.programStudi}</p>
      </CardContent>
      <CardFooter>
        <Button onClick={() => onEdit?.(student.id)}>Edit</Button>
        <Button variant="destructive" onClick={() => onDelete?.(student.id)}>
          Delete
        </Button>
      </CardFooter>
    </Card>
  );
};
```

---

## 🔄 State Management

### TanStack Query (React Query) with REST API
```tsx
// hooks/useStudents.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { apiClient } from '../lib/api-client';

export const useStudents = () => {
  return useQuery({
    queryKey: ['students'],
    queryFn: () => apiClient.get('/students').then(res => res.data.data),
  });
};

export const useCreateStudent = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: CreateStudentInput) => 
      apiClient.post('/students', data).then(res => res.data.data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['students'] });
    },
  });
};

export const useUpdateStudent = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: UpdateStudentInput }) =>
      apiClient.put(`/students/${id}`, data).then(res => res.data.data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['students'] });
    },
  });
};

export const useDeleteStudent = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (id: string) =>
      apiClient.delete(`/students/${id}`).then(res => res.data.data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['students'] });
    },
  });
};
```

### API Client Setup
```tsx
// lib/api-client.ts
import axios from 'axios';

export const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor untuk menambahkan auth token
apiClient.interceptors.request.use((config) => {
  const token = localStorage.getItem('accessToken');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Response interceptor untuk handle refresh token
apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;

    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;

      try {
        const refreshToken = localStorage.getItem('refreshToken');
        const response = await axios.post(`${import.meta.env.VITE_API_URL}/auth/refresh`, {
          refreshToken,
        });

        const { accessToken } = response.data.data;
        localStorage.setItem('accessToken', accessToken);

        originalRequest.headers.Authorization = `Bearer ${accessToken}`;
        return apiClient(originalRequest);
      } catch (refreshError) {
        // Redirect to login if refresh fails
        localStorage.removeItem('accessToken');
        localStorage.removeItem('refreshToken');
        window.location.href = '/login';
        return Promise.reject(refreshError);
      }
    }

    return Promise.reject(error);
  }
);
```

### Local State Management with Zustand
```tsx
// store/useStudentStore.ts
import { create } from 'zustand';

interface StudentState {
  selectedStudentId: string | null;
  setSelectedStudentId: (id: string | null) => void;
}

export const useStudentStore = create<StudentState>((set) => ({
  selectedStudentId: null,
  setSelectedStudentId: (id) => set({ selectedStudentId: id }),
}));
```

---

## 📝 Form Handling & Validation

### React Hook Form Example
```tsx
// components/forms/StudentForm.tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const studentSchema = z.object({
  name: z.string().min(1, 'Nama wajib diisi'),
  nim: z.string().min(1, 'NIM wajib diisi'),
  programStudi: z.string().min(1, 'Program studi wajib diisi'),
});

type StudentFormValues = z.infer<typeof studentSchema>;

export const StudentForm = ({ onSubmit }: { onSubmit: (data: StudentFormValues) => void }) => {
  const { register, handleSubmit, formState: { errors } } = useForm<StudentFormValues>({
    resolver: zodResolver(studentSchema),
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        {...register("name")}
        placeholder="Nama"
      />
      {errors.name && <span>{errors.name.message}</span>}

      <input
        {...register("nim")}
        placeholder="NIM"
      />
      {errors.nim && <span>{errors.nim.message}</span>}

      <input
        {...register("programStudi")}
        placeholder="Program Studi"
      />
      {errors.programStudi && <span>{errors.programStudi.message}</span>}

      <button type="submit">Simpan</button>
    </form>
  );
};
```

---

## 🧪 Testing Setup

### Unit Testing
```bash
# Install testing dependencies
npm install -D vitest @testing-library/react @testing-library/jest-dom
```

---

## 🚀 Build & Development

### Development Commands
```bash
# Start development server
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview

# Type checking
npm run type-check

# Linting
npm run lint
```

---

## 📚 Resources
- [React Documentation](https://react.dev/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [TanStack Query](https://tanstack.com/query/latest)
- [Zustand](https://docs.pmnd.rs/zustand/getting-started/introduction)
- [React Hook Form](https://react-hook-form.com/)
- [shadcn/ui Documentation](https://ui.shadcn.com/)
- [Tailwind CSS](https://tailwindcss.com/docs)
- [Axios Documentation](https://axios-http.com/docs/intro)
