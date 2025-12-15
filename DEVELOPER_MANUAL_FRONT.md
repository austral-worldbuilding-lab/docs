# Lab3 Frontend - Technical Documentation

This document provides comprehensive technical documentation for the Lab3 Worldbuilding Platform frontend application. It is intended for developers who need to understand, maintain, or extend the codebase.

## Table of Contents

1. [Product Overview](#1-product-overview)
2. [Technology Stack](#2-technology-stack)
3. [Project Architecture](#3-project-architecture)
4. [Components](#4-components)
5. [State Management](#5-state-management)
6. [Authentication System](#6-authentication-system)
7. [Services & API Layer](#7-services--api-layer)
8. [Routing System](#8-routing-system)
9. [Styling & UI](#9-styling--ui)
10. [Development Guide](#10-development-guide)
11. [Key Dependencies](#11-key-dependencies)

---

## 1. Product Overview

### What is Austral World Building Lab Platform?

Austral World Building Lab Platform is a **worldbuilding platform** designed to help teams collaboratively create and visualize complex worlds, scenarios, and narratives. It's particularly useful for:

- Strategic planning and scenario exploration
- Creative worldbuilding projects
- Collaborative brainstorming sessions
- Visual organization of complex information

### Core Concepts

#### Mandala

The **Mandala** is the central visualization tool in AWBL. It's a radial, multi-layered diagram that organizes information in concentric levels and sectors. Key characteristics:

- **Levels**: Concentric rings representing different layers of detail or abstraction
- **Sectors**: Radial divisions representing different categories or themes
- **Interactive canvas**: Built with Konva for smooth panning, zooming, and manipulation

#### Postits

**Postits** are the primary content units in a Mandala. They are:

- Positioned within specific levels and sectors
- Color-coded for visual categorization
- Hierarchical (postits can have child postits)
- Editable with rich text content

#### Dimensions

**Dimensions** are different views or perspectives of the same Mandala content. They allow users to analyze the worldbuilding from different angles (e.g., political, economic, cultural).

#### Characters

**Characters** are entities that can be associated with postits and tracked across the worldbuilding narrative.

### Organizational Hierarchy

```
Organization
└── Projects
    └── Mandalas
        ├── Postits
        ├── Characters
        └── Dimensions
```

- **Organizations**: Top-level containers for teams/companies
- **Projects**: Workspaces within organizations for specific initiatives
- **Mandalas**: The visual worldbuilding canvases within projects

---

## 2. Technology Stack

### Core Framework

| Technology | Version | Purpose                   |
| ---------- | ------- | ------------------------- |
| React      | 19.0.0  | UI framework              |
| TypeScript | 5.7.2   | Type-safe JavaScript      |
| Vite       | 6.3.1   | Build tool and dev server |

### State Management

| Technology           | Version | Purpose                                 |
| -------------------- | ------- | --------------------------------------- |
| TanStack React Query | 5.81.2  | Server state management and caching     |
| React Context API    | -       | Global client state (auth, permissions) |

### UI & Styling

| Technology               | Version | Purpose                         |
| ------------------------ | ------- | ------------------------------- |
| Tailwind CSS             | 4.1.5   | Utility-first CSS framework     |
| Radix UI                 | Various | Accessible component primitives |
| Lucide React             | 0.509.0 | Icon library                    |
| Class Variance Authority | 0.7.1   | Component variant management    |

### Canvas & Visualization

| Technology           | Version | Purpose                      |
| -------------------- | ------- | ---------------------------- |
| Konva                | 9.3.20  | 2D canvas library            |
| React Konva          | 19.0.3  | React bindings for Konva     |
| D3                   | 7.9.0   | Data visualization utilities |
| React Zoom Pan Pinch | 3.7.0   | Canvas navigation            |

### Backend Integration

| Technology | Version | Purpose                            |
| ---------- | ------- | ---------------------------------- |
| Firebase   | 11.6.1  | Authentication & Realtime Database |
| Axios      | 1.9.0   | HTTP client                        |

### Other

| Technology       | Version | Purpose                     |
| ---------------- | ------- | --------------------------- |
| React Router DOM | 7.6.0   | Client-side routing         |
| PostHog          | 1.261.7 | Analytics (production only) |
| jsPDF            | 3.0.2   | PDF generation              |
| Vitest           | 1.3.1   | Testing framework           |

---

## 3. Project Architecture

### Directory Structure

```
src/
├── assets/              # Static assets (images, fonts, SVGs)
├── components/          # React components
│   ├── ui/              # Base UI components (shadcn/ui style)
│   ├── auth/            # Authentication components
│   ├── common/          # Shared/reusable components
│   ├── layout/          # Layout components (Header, AppLayout)
│   ├── mandala/         # Mandala-specific components
│   │   ├── postits/     # Postit components
│   │   ├── characters/  # Character management
│   │   ├── sidebar/     # Mandala sidebar
│   │   ├── filters/     # Filter components
│   │   └── mandala-list/# Mandala listing
│   ├── organization/    # Organization management
│   ├── project/         # Project components
│   ├── home/            # Home page components
│   ├── files/           # File management
│   └── download/        # Download functionality
├── config/              # Configuration files
│   └── firebase.ts      # Firebase initialization
├── constants/           # Application constants
├── context/             # React Context providers
│   ├── AuthContext.tsx
│   ├── PermissionsContext.tsx
│   ├── PostItAnimationContext.tsx
│   └── ProjectBreadcrumbContext.tsx
├── hooks/               # Custom React hooks (60+ hooks)
├── lib/                 # Utilities and configurations
│   ├── axios.ts         # Axios instance configuration
│   └── utils.ts         # Utility functions
├── pages/               # Page components (route endpoints)
├── services/            # API service layer (20+ services)
├── styles/              # Global styles
│   └── index.css        # Main CSS file (Tailwind)
├── types/               # TypeScript type definitions
├── utils/               # Utility functions
├── main.tsx             # Application entry point
└── routes.tsx           # Route configuration
```

### Design Patterns

#### 1. Component-Based Architecture

Components are organized by domain/feature:

```
components/
├── ui/          # Generic, reusable UI primitives
├── mandala/     # Feature-specific (Mandala domain)
└── project/     # Feature-specific (Project domain)
```

#### 2. Service Layer Pattern

All API communication is centralized in the `services/` directory:

```typescript
// Services handle API calls
const data = await getMandalas(projectId);

// Hooks consume services and manage state
const { mandalas, isLoading } = useGetMandalas(projectId);
```

#### 3. Custom Hooks Pattern

Business logic is encapsulated in custom hooks:

```typescript
// Hook combines service calls with React Query
export function useFiles(scope: FileScope, id: string) {
  const { data, isLoading } = useQuery({
    queryKey: ["files", scope, id],
    queryFn: () => getFiles(scope, id),
  });

  return { files: data, isLoading };
}
```

#### 4. Context for Global State

Authentication and permissions use React Context:

```typescript
// Contexts provide global state
<AuthProvider>
  <PermissionsProvider>
    <App />
  </PermissionsProvider>
</AuthProvider>
```

### Data Flow

```
User Action
    ↓
Component (UI)
    ↓
Custom Hook (useFiles, useMandala, etc.)
    ↓
React Query (caching, refetching)
    ↓
Service Layer (API calls)
    ↓
Axios (HTTP) / Firebase (Realtime)
    ↓
Backend API
```

---

## 4. Components

### UI Components (`src/components/ui/`)

Base components following the shadcn/ui pattern. Built on Radix UI primitives with Tailwind CSS styling.

| Component     | File                | Description                                   |
| ------------- | ------------------- | --------------------------------------------- |
| Button        | `button.tsx`        | Button with variants (filled, outline, ghost) |
| Dialog        | `dialog.tsx`        | Modal dialog                                  |
| Card          | `card.tsx`          | Card container                                |
| Input         | `input.tsx`         | Text input                                    |
| Select        | `select.tsx`        | Dropdown select                               |
| Tabs          | `tabs.tsx`          | Tabbed interface                              |
| Tooltip       | `tooltip.tsx`       | Hover tooltips                                |
| Accordion     | `accordion.tsx`     | Collapsible sections                          |
| Alert Dialog  | `alert-dialog.tsx`  | Confirmation dialogs                          |
| Dropdown Menu | `dropdown-menu.tsx` | Context menus                                 |
| Badge         | `badge.tsx`         | Status badges                                 |
| Checkbox      | `checkbox.tsx`      | Checkbox input                                |
| Textarea      | `textarea.tsx`      | Multi-line text input                         |
| Scroll Area   | `scroll-area.tsx`   | Custom scrollable area                        |

### Mandala Components (`src/components/mandala/`)

Core components for the Mandala visualization:

| Component           | File                      | Description                               |
| ------------------- | ------------------------- | ----------------------------------------- |
| MandalaContainer    | `MandalaContainer.tsx`    | Main mandala wrapper and state management |
| MandalaCanvas       | `MandalaCanvas.tsx`       | Konva stage and layer setup               |
| KonvaContainer      | `KonvaContainer.tsx`      | Konva rendering logic                     |
| MultiKonvaContainer | `MultiKonvaContainer.tsx` | Multiple mandala comparison view          |
| MandalaSVG          | `MandalaSVG.tsx`          | SVG export functionality                  |
| Mandala             | `Mandala.tsx`             | Mandala shape rendering                   |

### Postit Components (`src/components/mandala/postits/`)

| Component    | File               | Description                 |
| ------------ | ------------------ | --------------------------- |
| Postit       | `Postit.tsx`       | Individual postit rendering |
| PostitEditor | `PostitEditor.tsx` | Inline postit editing       |
| PostitDialog | `PostitDialog.tsx` | Postit detail modal         |
| PostitMenu   | `PostitMenu.tsx`   | Context menu for postits    |

### Layout Components (`src/components/layout/`)

| Component        | File                   | Description             |
| ---------------- | ---------------------- | ----------------------- |
| AppLayout        | `AppLayout.tsx`        | Main application layout |
| Header           | `Header.tsx`           | Application header      |
| NotificationBell | `NotificationBell.tsx` | Notification indicator  |

---

## 5. Authentication System

### Architecture

```
Firebase Auth (Identity Provider)
        ↓
    JWT Token
        ↓
Axios Interceptor (adds token to requests)
        ↓
Backend API (validates token)
```

### Firebase Configuration

```typescript
// src/config/firebase.ts
const firebaseConfig = {
  apiKey: import.meta.env.VITE_FIREBASE_API_KEY,
  authDomain: import.meta.env.VITE_FIREBASE_AUTH_DOMAIN,
  projectId: import.meta.env.VITE_FIREBASE_PROJECT_ID,
  storageBucket: import.meta.env.VITE_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: import.meta.env.VITE_FIREBASE_MESSAGING_SENDER_ID,
  appId: import.meta.env.VITE_FIREBASE_APP_ID,
  databaseURL: import.meta.env.VITE_FIREBASE_DATABASE_URL,
};

const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);
export const db = getFirestore(app);
export const database = getDatabase(app);
```

### Authentication Hook

```typescript
// src/hooks/useAuth.ts
export const useAuth = () => {
  const [user, setUser] = useState<FirebaseUser | null>(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    const unsubscribe = onAuthStateChanged(auth, (currentUser) => {
      setUser(currentUser);
      setIsLoading(false);
    });
    return () => unsubscribe();
  }, []);

  const login = async (email: string, password: string): Promise<boolean> => {
    await setPersistence(auth, browserLocalPersistence);
    const result = await signInWithEmailAndPassword(auth, email, password);
    setUser(result.user);
    return true;
  };

  const register = async (
    email: string,
    password: string,
    fullName: string
  ) => {
    const result = await createUserWithEmailAndPassword(auth, email, password);
    // Also create user in backend
    await createUser({
      firebaseUid: result.user.uid,
      email: result.user.email!,
      fullName,
    });
    return true;
  };

  const logout = async () => {
    await signOut(auth);
    setUser(null);
  };

  return { user, login, register, logout, isAuth: () => !!user, isLoading };
};
```

### Protected Routes

```typescript
// src/components/auth/ProtectedRoute.tsx
const ProtectedRoute = ({ children }: ProtectedRouteProps) => {
  const { isAuth, isLoading } = useAuthContext();

  if (isLoading) return null;

  if (!isAuth()) {
    // Save intended destination
    sessionStorage.setItem(RETURN_TO_KEY, location.pathname);
    return <Navigate to="/login" replace />;
  }

  return <>{children}</>;
};
```

### Automatic Token Injection

```typescript
// src/lib/axios.ts
axiosInstance.interceptors.request.use(async (config) => {
  const user = auth.currentUser;
  if (user) {
    const token = await user.getIdToken();
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});
```

---

## 6. Services & API Layer

### Axios Configuration

```typescript
// src/lib/axios.ts
const axiosInstance = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  headers: { "Content-Type": "application/json" },
});

// Response interceptor - handle 401 errors
axiosInstance.interceptors.response.use(
  (response) => response,
  async (error) => {
    if (error.response?.status === 401) {
      await signOut(auth);
      window.location.href = "/login";
    }
    return Promise.reject(error);
  }
);

// Generic typed methods
export async function get<T>(url: string): Promise<AxiosResponse<T>> {
  return axiosInstance.get<T>(url);
}

export async function post<T, U>(
  url: string,
  data: U
): Promise<AxiosResponse<T>> {
  return axiosInstance.post<T>(url, data);
}
```

### Service Structure

Services are organized by domain:

| Service             | File                     | Purpose                 |
| ------------------- | ------------------------ | ----------------------- |
| organizationService | `organizationService.ts` | CRUD for organizations  |
| projectService      | `projectService.ts`      | CRUD for projects       |
| mandalaService      | `mandalaService.ts`      | Mandala operations      |
| filesService        | `filesService.ts`        | File upload/download    |
| userService         | `userService.ts`         | User management         |
| invitationService   | `invitationService.ts`   | Project invitations     |
| solutionService     | `solutionService.ts`     | Solution management     |
| notificationService | `notificationService.ts` | User notifications      |
| imageService        | `imageService.ts`        | Image operations        |
| presence            | `presence.ts`            | Real-time user presence |
| analytics           | `analytics.ts`           | PostHog integration     |

---

## 7. Routing System

### URL Patterns

| Pattern                                                                                    | Page                        | Description               |
| ------------------------------------------------------------------------------------------ | --------------------------- | ------------------------- |
| `/login`                                                                                   | LoginPage                   | User login                |
| `/register`                                                                                | RegisterPage                | User registration         |
| `/invite/:token`                                                                           | InviteTokenPage             | Accept project invitation |
| `/organization-invite/:token`                                                              | OrganizationInviteTokenPage | Accept org invitation     |
| `/app/organization/`                                                                       | OrganizationListPage        | List user's organizations |
| `/app/organization/:orgId/projects`                                                        | OrganizationPage            | List org's projects       |
| `/app/organization/:orgId/projects/:projectId`                                             | ProjectPage                 | Project dashboard         |
| `/app/organization/:orgId/projects/:projectId/timeline`                                    | TimelinePage                | Project timeline          |
| `/app/organization/:orgId/projects/:projectId/solutions`                                   | SolutionsPage               | Project solutions         |
| `/app/organization/:orgId/projects/:projectId/mandala/:mandalaId`                          | MandalaPage                 | Mandala view              |
| `/app/organization/:orgId/projects/:projectId/mandala/:mandalaId/dimension/:dimensionName` | DimensionPage               | Dimension view            |

---

## 8. Styling & UI

### Global Styles

```css
/* src/styles/index.css */
@import "tailwindcss";
@import "tw-animate-css";

@custom-variant dark (&:is(.dark *));

@theme {
  --color-primary: #172187;
  --color-secondary: #dba5e5;
  --color-accent: #f59e0b;
  --color-danger: #dc2626;
}

@layer base {
  * {
    @apply border-border outline-ring/50;
  }
}
```

### Component Variants with CVA

```typescript
// src/components/ui/button.tsx
import { cva, type VariantProps } from "class-variance-authority";

const buttonVariants = cva(
  "inline-flex items-center justify-center gap-2 whitespace-nowrap rounded-md font-medium",
  {
    variants: {
      variant: {
        filled: "",
        outline: "border",
        ghost: "bg-transparent",
      },
      size: {
        sm: "h-8 px-3 text-sm",
        md: "h-10 px-4 text-base",
        lg: "h-12 px-6 text-lg",
      },
      color: {
        primary: "bg-primary text-white hover:bg-primary/90",
        secondary: "bg-secondary text-primary hover:bg-secondary/90",
        danger: "bg-danger text-white hover:bg-danger/90",
      },
    },
    defaultVariants: {
      variant: "filled",
      size: "md",
      color: "primary",
    },
  }
);

interface ButtonProps extends VariantProps<typeof buttonVariants> {
  // ...
}
```

---

## 9. Development Guide

### Creating a New Component

1. **Create the component file**:

```typescript
// src/components/feature/MyComponent.tsx
import { cn } from "@/lib/utils";

interface MyComponentProps {
  title: string;
  className?: string;
}

export function MyComponent({ title, className }: MyComponentProps) {
  return (
    <div className={cn("p-4 rounded-lg bg-white", className)}>
      <h2 className="text-lg font-semibold">{title}</h2>
    </div>
  );
}
```

2. **Export from index** (if using barrel exports):

```typescript
// src/components/feature/index.ts
export { MyComponent } from "./MyComponent";
```

### Creating a New Hook

1. **Create the hook file**:

```typescript
// src/hooks/useMyFeature.ts
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { getMyData, updateMyData } from "@/services/myService";

export const myFeatureKeys = {
  all: ["myFeature"] as const,
  byId: (id: string) => [...myFeatureKeys.all, id] as const,
};

export function useMyFeature(id: string) {
  const queryClient = useQueryClient();

  const { data, isLoading, error } = useQuery({
    queryKey: myFeatureKeys.byId(id),
    queryFn: () => getMyData(id),
    enabled: !!id,
  });

  const updateMutation = useMutation({
    mutationFn: (newData: UpdateData) => updateMyData(id, newData),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: myFeatureKeys.byId(id) });
    },
  });

  return {
    data,
    isLoading,
    error,
    update: updateMutation.mutateAsync,
    isUpdating: updateMutation.isPending,
  };
}
```

### Creating a New Service

1. **Create the service file**:

```typescript
// src/services/myService.ts
import axiosInstance from "@/lib/axios";
import { MyData, CreateMyData, UpdateMyData } from "@/types/myTypes";

export const getMyData = async (id: string): Promise<MyData> => {
  const response = await axiosInstance.get<{ data: MyData }>(
    `/my-endpoint/${id}`
  );
  return response.data.data;
};

export const createMyData = async (data: CreateMyData): Promise<MyData> => {
  const response = await axiosInstance.post<{ data: MyData }>(
    "/my-endpoint",
    data
  );
  return response.data.data;
};

export const updateMyData = async (
  id: string,
  data: UpdateMyData
): Promise<MyData> => {
  const response = await axiosInstance.patch<{ data: MyData }>(
    `/my-endpoint/${id}`,
    data
  );
  return response.data.data;
};

export const deleteMyData = async (id: string): Promise<void> => {
  await axiosInstance.delete(`/my-endpoint/${id}`);
};
```

### Adding a New Page

1. **Create the page component**:

```typescript
// src/pages/app/MyPage.tsx
import { useParams } from "react-router-dom";
import { AppLayout } from "@/components/layout/AppLayout";
import { useMyFeature } from "@/hooks/useMyFeature";

export default function MyPage() {
  const { id } = useParams<{ id: string }>();
  const { data, isLoading } = useMyFeature(id!);

  if (isLoading) return <div>Loading...</div>;

  return (
    <AppLayout>
      <div className="p-6">
        <h1 className="text-2xl font-bold">{data?.title}</h1>
      </div>
    </AppLayout>
  );
}
```

2. **Add the route**:

```typescript
// src/routes.tsx
<Route path="my-page/:id" element={<MyPage />} />
```

### Code Conventions

- **File naming**: PascalCase for components (`MyComponent.tsx`), camelCase for utilities (`myUtil.ts`)
- **Component naming**: PascalCase (`MyComponent`)
- **Hook naming**: camelCase with `use` prefix (`useMyHook`)
- **Service naming**: camelCase (`myService.ts`)
- **Type naming**: PascalCase (`MyType`)
- **Interfaces**: Prefer `interface` over `type` for object shapes
- **Props**: Suffix with `Props` (`MyComponentProps`)

### Path Aliases

The project uses the `@/` alias for imports:

```typescript
// Instead of relative imports
import { Button } from "../../../components/ui/button";

// Use alias
import { Button } from "@/components/ui/button";
```

---

## 10. Key Dependencies

### React Konva

Used for the Mandala canvas visualization
**Documentation**: [konvajs.org](https://konvajs.org/)

### TanStack React Query

Server state management
**Documentation**: [tanstack.com/query](https://tanstack.com/query)

### Radix UI

Accessible component primitives
**Documentation**: [radix-ui.com](https://www.radix-ui.com/)

### Firebase

Authentication and real-time database
**Documentation**: [firebase.google.com/docs](https://firebase.google.com/docs)

### Tailwind CSS

Utility-first CSS:
**Documentation**: [tailwindcss.com](https://tailwindcss.com/)

### Lucide React

Icon library
**Documentation**: [lucide.dev](https://lucide.dev/)

---

## Quick Reference

### Common Commands

```bash
npm run dev          # Start development server
npm run build        # Build for production
npm run lint         # Run ESLint
npm run test         # Run tests
```

### Key Files

| File                          | Purpose                 |
| ----------------------------- | ----------------------- |
| `src/main.tsx`                | Application entry point |
| `src/routes.tsx`              | Route configuration     |
| `src/lib/axios.ts`            | HTTP client setup       |
| `src/config/firebase.ts`      | Firebase configuration  |
| `src/context/AuthContext.tsx` | Authentication provider |
| `src/styles/index.css`        | Global styles           |

### Environment Variables

See `.env.template` for all required environment variables.
