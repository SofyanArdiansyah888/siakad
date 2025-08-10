# Integrasi shadcn/ui ke SIAKAD Frontend

## Mengapa Menggunakan shadcn/ui?

### 1. **Konsistensi UI/UX**
- Komponen yang sudah didesain dengan baik dan konsisten
- Mengikuti design system yang modern dan accessible
- Mengurangi waktu development untuk styling

### 2. **Customizable**
- Bukan library yang rigid, bisa dikustomisasi sesuai kebutuhan
- Menggunakan Tailwind CSS untuk styling yang fleksibel
- Bisa dimodifikasi tanpa breaking changes

### 3. **Accessibility**
- Dibangun di atas Radix UI primitives
- Memenuhi standar accessibility (WCAG)
- Keyboard navigation dan screen reader support

### 4. **TypeScript Support**
- Full TypeScript support dengan type safety
- Auto-completion dan IntelliSense yang baik
- Mengurangi runtime errors

### 5. **Performance**
- Bundle size yang kecil
- Tree-shaking friendly
- Hanya import komponen yang digunakan

## Komponen yang Diintegrasikan

### Core Components
- **Button** - Tombol dengan berbagai variant
- **Input** - Input field dengan styling konsisten
- **Label** - Label untuk form elements
- **Card** - Container dengan header, content, footer
- **Badge** - Status indicators dan labels
- **Select** - Dropdown select dengan search
- **Dialog** - Modal dialog dengan overlay

### Dependencies
```json
{
  "@radix-ui/react-slot": "^1.0.2",
  "@radix-ui/react-dialog": "^1.0.5",
  "@radix-ui/react-select": "^2.0.0",
  "@radix-ui/react-label": "^2.0.2",
  "class-variance-authority": "^0.7.0",
  "clsx": "^2.0.0",
  "tailwind-merge": "^2.0.0",
  "lucide-react": "^0.294.0"
}
```

## Struktur File

```
src/
├── components/
│   ├── ui/                    # shadcn/ui components
│   │   ├── button.tsx
│   │   ├── input.tsx
│   │   ├── label.tsx
│   │   ├── card.tsx
│   │   ├── badge.tsx
│   │   ├── select.tsx
│   │   ├── dialog.tsx
│   │   └── index.ts
│   ├── examples/              # Contoh penggunaan
│   │   ├── ShadcnExample.tsx
│   │   └── index.ts
│   ├── StudentForm.tsx        # Updated dengan shadcn/ui
│   └── StudentList.tsx        # Updated dengan shadcn/ui
├── lib/
│   └── utils.ts               # Utility functions
└── index.css                  # CSS variables untuk tema
```

## Penggunaan Komponen

### Button
```tsx
import { Button } from './ui/button'

// Default button
<Button>Click me</Button>

// Custom styling
<Button className="bg-gradient-to-r from-blue-600 to-purple-600 text-white">
  Custom Button
</Button>
```

### Input dengan Label
```tsx
import { Input } from './ui/input'
import { Label } from './ui/label'

<div className="space-y-2">
  <Label htmlFor="email">Email</Label>
  <Input id="email" type="email" placeholder="Enter email" />
</div>
```

### Card Layout
```tsx
import { Card, CardHeader, CardContent, CardFooter } from './ui/card'

<Card>
  <CardHeader>
    <h2>Card Title</h2>
  </CardHeader>
  <CardContent>
    <p>Card content goes here</p>
  </CardContent>
  <CardFooter>
    <Button>Action</Button>
  </CardFooter>
</Card>
```

### Select Dropdown
```tsx
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from './ui/select'

<Select value={value} onValueChange={setValue}>
  <SelectTrigger>
    <SelectValue placeholder="Select option" />
  </SelectTrigger>
  <SelectContent>
    <SelectItem value="option1">Option 1</SelectItem>
    <SelectItem value="option2">Option 2</SelectItem>
  </SelectContent>
</Select>
```

### Dialog/Modal
```tsx
import { Dialog, DialogContent, DialogHeader, DialogTitle } from './ui/dialog'

<Dialog open={isOpen} onOpenChange={setIsOpen}>
  <DialogContent>
    <DialogHeader>
      <DialogTitle>Modal Title</DialogTitle>
    </DialogHeader>
    <div>Modal content</div>
  </DialogContent>
</Dialog>
```

## Customization

### Tema Warna
CSS variables di `src/index.css`:
```css
:root {
  --background: 0 0% 100%;
  --foreground: 222.2 84% 4.9%;
  --primary: 221.2 83.2% 53.3%;
  --primary-foreground: 210 40% 98%;
  /* ... more variables */
}
```

### Custom Variants
Menggunakan `class-variance-authority`:
```tsx
const buttonVariants = cva(
  "inline-flex items-center justify-center rounded-md text-sm font-medium",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground",
        custom: "bg-gradient-to-r from-green-500 to-emerald-500 text-white",
      },
    },
  }
)
```

## Best Practices

### 1. **Konsistensi**
- Gunakan komponen shadcn/ui untuk semua UI elements
- Ikuti pola yang sudah ditetapkan
- Jangan mix dengan styling custom yang tidak perlu

### 2. **Accessibility**
- Selalu gunakan proper labels
- Implement keyboard navigation
- Test dengan screen readers

### 3. **Performance**
- Import hanya komponen yang dibutuhkan
- Gunakan dynamic imports untuk komponen besar
- Optimize bundle size

### 4. **Maintenance**
- Update dependencies secara regular
- Test komponen setelah update
- Dokumentasikan custom variants

## Migration dari Custom Components

### Sebelum (Custom Styling)
```tsx
<button className="btn-primary flex items-center space-x-2 shadow-lg">
  <FiSave className="h-4 w-4" />
  <span>Save</span>
</button>
```

### Sesudah (shadcn/ui)
```tsx
<Button className="flex items-center space-x-2 shadow-lg bg-gradient-to-r from-blue-600 to-purple-600 text-white">
  <FiSave className="h-4 w-4" />
  <span>Save</span>
</Button>
```

## Keuntungan Implementasi

1. **Development Speed** - Komponen siap pakai
2. **Consistency** - UI yang konsisten di seluruh aplikasi
3. **Maintainability** - Mudah di-maintain dan update
4. **Accessibility** - Built-in accessibility features
5. **Customization** - Fleksibel untuk kebutuhan custom
6. **Type Safety** - Full TypeScript support

## Next Steps

1. **Tambah Komponen Lain** - Accordion, Tabs, Toast, dll
2. **Theme System** - Implement dark mode toggle
3. **Animation** - Tambah micro-interactions
4. **Testing** - Unit tests untuk komponen
5. **Documentation** - Storybook untuk komponen
