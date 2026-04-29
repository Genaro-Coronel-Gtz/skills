---
name: design
description: Configure design system with Tailwind CSS, shadcn/ui components, CSS variables for theming, and DESIGN.md specification. Use when setting up a new project's visual identity, implementing dark mode, or creating consistent component styling.
---

## Use

**Caso 1: Configurar sistema de diseño desde cero**
Usa la skill design.md para configurar Tailwind CSS, crear globals.css con CSS variables, configurar tailwind.config.js, y crear DESIGN.md documentando el sistema de diseño que acabas de crear.

**Caso 2: Documentar sistema de diseño existente**
Usa la skill design.md para crear un archivo DESIGN.md basado en el sistema de diseño actual del proyecto. Lee src/presentation/styles/globals.css (fuente de la verdad), extrae los valores HSL de las CSS variables en los selectores :root y .dark, y genera el archivo DESIGN.md con la estructura YAML frontmatter y las secciones de documentación en markdown.

# Design System Configuration

## Quick start

```typescript
// 1. Install dependencies
npm install tailwindcss @tailwindcss-animate class-variance-authority clsx tailwind-merge

// 2. Initialize Tailwind
npx tailwindcss init -p

// 3. Configure tailwind.config.js
export default {
  darkMode: ['class'],
  content: ['./index.html', './src/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {
      colors: {
        border: 'hsl(var(--border))',
        input: 'hsl(var(--input))',
        ring: 'hsl(var(--ring))',
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',
        primary: {
          DEFAULT: 'hsl(var(--primary))',
          foreground: 'hsl(var(--primary-foreground))',
        },
        secondary: {
          DEFAULT: 'hsl(var(--secondary))',
          foreground: 'hsl(var(--secondary-foreground))',
        },
        destructive: {
          DEFAULT: 'hsl(var(--destructive))',
          foreground: 'hsl(var(--destructive-foreground))',
        },
        muted: {
          DEFAULT: 'hsl(var(--muted))',
          foreground: 'hsl(var(--muted-foreground))',
        },
        accent: {
          DEFAULT: 'hsl(var(--accent))',
          foreground: 'hsl(var(--accent-foreground))',
        },
        popover: {
          DEFAULT: 'hsl(var(--popover))',
          foreground: 'hsl(var(--popover-foreground))',
        },
        card: {
          DEFAULT: 'hsl(var(--card))',
          foreground: 'hsl(var(--card-foreground))',
        },
      },
      borderRadius: {
        lg: 'var(--radius)',
        md: 'calc(var(--radius) - 2px)',
        sm: 'calc(var(--radius) - 4px)',
      },
    },
  },
  plugins: [require('tailwindcss-animate')],
};
```

```css
/* 4. Create src/presentation/styles/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --card: 0 0% 100%;
    --card-foreground: 222.2 84% 4.9%;
    --popover: 0 0% 100%;
    --popover-foreground: 222.2 84% 4.9%;
    --primary: 221.2 83.2% 53.3%;
    --primary-foreground: 210 40% 98%;
    --secondary: 210 40% 96.1%;
    --secondary-foreground: 222.2 47.4% 11.2%;
    --muted: 210 40% 96.1%;
    --muted-foreground: 215.4 16.3% 46.9%;
    --accent: 210 40% 96.1%;
    --accent-foreground: 222.2 47.4% 11.2%;
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 210 40% 98%;
    --border: 214.3 31.8% 91.4%;
    --input: 214.3 31.8% 91.4%;
    --ring: 221.2 83.2% 53.3%;
    --radius: 0.5rem;
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    --card: 222.2 84% 4.9%;
    --card-foreground: 210 40% 98%;
    --popover: 222.2 84% 4.9%;
    --popover-foreground: 210 40% 98%;
    --primary: 217.2 91.2% 59.8%;
    --primary-foreground: 222.2 47.4% 11.2%;
    --secondary: 217.2 32.6% 17.5%;
    --secondary-foreground: 210 40% 98%;
    --muted: 217.2 32.6% 17.5%;
    --muted-foreground: 215 20.2% 65.1%;
    --accent: 217.2 32.6% 17.5%;
    --accent-foreground: 210 40% 98%;
    --destructive: 0 62.8% 30.6%;
    --destructive-foreground: 210 40% 98%;
    --border: 217.2 32.6% 17.5%;
    --input: 217.2 32.6% 17.5%;
    --ring: 224.3 76.3% 48%;
  }
}

@layer base {
  * {
    @apply border-border;
  }
  body {
    @apply bg-background text-foreground;
  }
}
```

```typescript
// 5. Create lib/utils.ts
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

## Workflows

### Configure color palette

**Case 1: New project - Define palette manually**

```yaml
# Define your color palette in DESIGN.md
colors:
  # Light mode
  background: "hsl(0 0% 100%)"
  foreground: "hsl(222.2 84% 4.9%)"
  primary: "hsl(221.2 83.2% 53.3%)"
  primary-foreground: "hsl(210 40% 98%)"
  secondary: "hsl(210 40% 96.1%)"
  secondary-foreground: "hsl(222.2 47.4% 11.2%)"
  muted: "hsl(210 40% 96.1%)"
  muted-foreground: "hsl(215.4 16.3% 46.9%)"
  accent: "hsl(210 40% 96.1%)"
  accent-foreground: "hsl(222.2 47.4% 11.2%)"
  destructive: "hsl(0 84.2% 60.2%)"
  destructive-foreground: "hsl(210 40% 98%)"
  border: "hsl(214.3 31.8% 91.4%)"
  input: "hsl(214.3 31.8% 91.4%)"
  ring: "hsl(221.2 83.2% 53.3%)"
  
  # Dark mode
  dark-background: "hsl(222.2 84% 4.9%)"
  dark-foreground: "hsl(210 40% 98%)"
  dark-primary: "hsl(217.2 91.2% 59.8%)"
  dark-primary-foreground: "hsl(222.2 47.4% 11.2%)"
  dark-secondary: "hsl(217.2 32.6% 17.5%)"
  dark-secondary-foreground: "hsl(210 40% 98%)"
  dark-muted: "hsl(217.2 32.6% 17.5%)"
  dark-muted-foreground: "hsl(215 20.2% 65.1%)"
  dark-accent: "hsl(217.2 32.6% 17.5%)"
  dark-accent-foreground: "hsl(210 40% 98%)"
  dark-destructive: "hsl(0 62.8% 30.6%)"
  dark-destructive-foreground: "hsl(210 40% 98%)"
  dark-border: "hsl(217.2 32.6% 17.5%)"
  dark-input: "hsl(217.2 32.6% 17.5%)"
  dark-ring: "hsl(224.3 76.3% 48%)"
```

**Case 2: Existing project - Extract palette from globals.css**

```typescript
// Read globals.css and extract HSL values from :root and .dark
// Example extraction:
:root {
  --background: 0 0% 100%;        // → background: "hsl(0 0% 100%)"
  --foreground: 222.2 84% 4.9%;  // → foreground: "hsl(222.2 84% 4.9%)"
  --primary: 221.2 83.2% 53.3%;  // → primary: "hsl(221.2 83.2% 53.3%)"
  // ... extract all variables
}

.dark {
  --background: 222.2 84% 4.9%;  // → dark-background: "hsl(222.2 84% 4.9%)"
  --foreground: 210 40% 98%;     // → dark-foreground: "hsl(210 40% 98%)"
  // ... extract all dark mode variables
}
```

**Generate globals.css from palette**

```css
@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --card: 0 0% 100%;
    --card-foreground: 222.2 84% 4.9%;
    --popover: 0 0% 100%;
    --popover-foreground: 222.2 84% 4.9%;
    --primary: 221.2 83.2% 53.3%;
    --primary-foreground: 210 40% 98%;
    --secondary: 210 40% 96.1%;
    --secondary-foreground: 222.2 47.4% 11.2%;
    --muted: 210 40% 96.1%;
    --muted-foreground: 215.4 16.3% 46.9%;
    --accent: 210 40% 96.1%;
    --accent-foreground: 222.2 47.4% 11.2%;
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 210 40% 98%;
    --border: 214.3 31.8% 91.4%;
    --input: 214.3 31.8% 91.4%;
    --ring: 221.2 83.2% 53.3%;
    --radius: 0.5rem;
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    --card: 222.2 84% 4.9%;
    --card-foreground: 210 40% 98%;
    --popover: 222.2 84% 4.9%;
    --popover-foreground: 210 40% 98%;
    --primary: 217.2 91.2% 59.8%;
    --primary-foreground: 222.2 47.4% 11.2%;
    --secondary: 217.2 32.6% 17.5%;
    --secondary-foreground: 210 40% 98%;
    --muted: 217.2 32.6% 17.5%;
    --muted-foreground: 215 20.2% 65.1%;
    --accent: 217.2 32.6% 17.5%;
    --accent-foreground: 210 40% 98%;
    --destructive: 0 62.8% 30.6%;
    --destructive-foreground: 210 40% 98%;
    --border: 217.2 32.6% 17.5%;
    --input: 217.2 32.6% 17.5%;
    --ring: 224.3 76.3% 48%;
  }
}
```

**Generate tailwind.config.js from palette**

```javascript
export default {
  darkMode: ['class'],
  content: ['./index.html', './src/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {
      colors: {
        border: 'hsl(var(--border))',
        input: 'hsl(var(--input))',
        ring: 'hsl(var(--ring))',
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',
        primary: {
          DEFAULT: 'hsl(var(--primary))',
          foreground: 'hsl(var(--primary-foreground))',
        },
        secondary: {
          DEFAULT: 'hsl(var(--secondary))',
          foreground: 'hsl(var(--secondary-foreground))',
        },
        destructive: {
          DEFAULT: 'hsl(var(--destructive))',
          foreground: 'hsl(var(--destructive-foreground))',
        },
        muted: {
          DEFAULT: 'hsl(var(--muted))',
          foreground: 'hsl(var(--muted-foreground))',
        },
        accent: {
          DEFAULT: 'hsl(var(--accent))',
          foreground: 'hsl(var(--accent-foreground))',
        },
        popover: {
          DEFAULT: 'hsl(var(--popover))',
          foreground: 'hsl(var(--popover-foreground))',
        },
        card: {
          DEFAULT: 'hsl(var(--card))',
          foreground: 'hsl(var(--card-foreground))',
        },
      },
      borderRadius: {
        lg: 'var(--radius)',
        md: 'calc(var(--radius) - 2px)',
        sm: 'calc(var(--radius) - 4px)',
      },
    },
  },
  plugins: [require('tailwindcss-animate')],
};
```

### Apply existing DESIGN.md to project

```typescript
// 1. Read DESIGN.md file
// The DESIGN.md file contains YAML frontmatter with design tokens

// 2. Extract color tokens from YAML
// Example colors from DESIGN.md:
colors:
  background: "hsl(0 0% 100%)"
  foreground: "hsl(222.2 84% 4.9%)"
  primary: "hsl(221.2 83.2% 53.3%)"
  # ... other colors

// 3. Apply colors to globals.css (SOURCE OF TRUTH)
// globals.css contains the actual HSL values
@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --primary: 221.2 83.2% 53.3%;
    // ... map all color tokens
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    --primary: 217.2 91.2% 59.8%;
    // ... map dark mode tokens
  }
}

// 4. Update tailwind.config.js to reference CSS variables
// tailwind.config.js only maps variable names, not values
export default {
  theme: {
    extend: {
      colors: {
        primary: 'hsl(var(--primary))',  // Reference to CSS variable
        // ... other color references
      }
    }
  }
}

// 5. Apply typography tokens
typography:
  h1:
    fontSize: "2.25rem"
    fontWeight: "700"

// Apply to Tailwind config:
theme: {
  extend: {
    fontSize: {
      '4xl': ['2.25rem', { lineHeight: '2.5rem', fontWeight: '700' }],
    }
  }
}

// 5. Apply spacing tokens
spacing:
  sm: "0.5rem"
  md: "1rem"
  lg: "1.5rem"

// Apply to Tailwind config:
theme: {
  extend: {
    spacing: {
      'sm': '0.5rem',
      'md': '1rem',
      'lg': '1.5rem',
    }
  }
}

// 6. Apply rounded tokens
rounded:
  sm: "calc(var(--radius) - 4px)"
  md: "calc(var(--radius) - 2px)"
  lg: "var(--radius)"

// Apply to Tailwind config:
theme: {
  extend: {
    borderRadius: {
      'sm': 'calc(var(--radius) - 4px)',
      'md': 'calc(var(--radius) - 2px)',
      'lg': 'var(--radius)',
    }
  }
}
```

### Create DESIGN.md from existing project

```yaml
---
name: Project Design System
description: Description of design system
version: "1.0.0"

colors:
  # Extract HSL values from globals.css (SOURCE OF TRUTH)
  # globals.css @layer base :root contains actual values
  background: "hsl(0 0% 100%)"
  foreground: "hsl(222.2 84% 4.9%)"
  primary: "hsl(221.2 83.2% 53.3%)"
  # ... all color tokens from :root and .dark selectors

typography:
  fontFamily: "system-ui, sans-serif"
  h1:
    fontSize: "2.25rem"
    fontWeight: "700"
  # ... all typography tokens

rounded:
  sm: "calc(var(--radius) - 4px)"
  md: "calc(var(--radius) - 2px)"
  lg: "var(--radius)"

spacing:
  sm: "0.5rem"
  md: "1rem"
  lg: "1.5rem"

components:
  button-primary:
    backgroundColor: "{colors.primary}"
    textColor: "{colors.primary-foreground}"
    rounded: "{rounded.md}"
  # ... component tokens
---

## Overview
Design system description...
```

### Install shadcn/ui

```bash
npx shadcn@latest init
```

```json
// components.json
{
  "$schema": "https://ui.shadcn.com/schema.json",
  "style": "default",
  "rsc": false,
  "tsx": true,
  "tailwind": {
    "config": "tailwind.config.js",
    "css": "src/presentation/styles/globals.css",
    "baseColor": "slate",
    "cssVariables": true,
    "prefix": ""
  },
  "aliases": {
    "components": "@/presentation/components",
    "utils": "@/shared/utils"
  }
}
```

### Add shadcn/ui components

```bash
npx shadcn@latest add button
npx shadcn@latest add input
npx shadcn@latest add card
npx shadcn@latest add dialog
npx shadcn@latest add table
npx shadcn@latest add badge
npx shadcn@latest add alert
npx shadcn@latest add toast
```

### Implement dark mode toggle

```typescript
// src/presentation/components/theme-toggle.tsx
import { useEffect } from 'react';
import { Moon, Sun } from 'lucide-react';
import { Button } from '@/presentation/components/ui/button';

export const ThemeToggle = () => {
  useEffect(() => {
    const isDark = localStorage.getItem('theme') === 'dark';
    document.documentElement.classList.toggle('dark', isDark);
  }, []);

  const toggleTheme = () => {
    const isDark = document.documentElement.classList.toggle('dark');
    localStorage.setItem('theme', isDark ? 'dark' : 'light');
  };

  return (
    <Button variant="ghost" size="icon" onClick={toggleTheme}>
      <Sun className="h-5 w-5 rotate-0 scale-100 transition-all dark:-rotate-90 dark:scale-0" />
      <Moon className="absolute h-5 w-5 rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100" />
    </Button>
  );
};
```

### Customize color palette

```css
/* Modify CSS variables in globals.css */
:root {
  /* Change primary color to green */
  --primary: 142 76% 36%;
  --primary-foreground: 355.7 100% 97.3%;
  
  /* Change radius */
  --radius: 0.75rem;
}
```

### Create DESIGN.md specification

```yaml
---
name: Your Design System
description: Description of your design system
version: "1.0.0"

colors:
  background: "hsl(0 0% 100%)"
  foreground: "hsl(222.2 84% 4.9%)"
  primary: "hsl(221.2 83.2% 53.3%)"
  # ... other colors

typography:
  fontFamily: "system-ui, sans-serif"
  h1:
    fontSize: "2.25rem"
    fontWeight: "700"

rounded:
  sm: "calc(var(--radius) - 4px)"
  md: "calc(var(--radius) - 2px)"
  lg: "var(--radius)"

spacing:
  sm: "0.5rem"
  md: "1rem"
  lg: "1.5rem"

components:
  button-primary:
    backgroundColor: "{colors.primary}"
    textColor: "{colors.primary-foreground}"
    rounded: "{rounded.md}"
---

## Overview
Design system description...
```

### Use CSS variables in components

```typescript
// Components automatically use CSS variables via Tailwind
<Button className="bg-primary text-primary-foreground">
  Click me
</Button>

// Custom component with CSS variables
<div className="bg-card text-card-foreground border-border rounded-lg p-4">
  Content
</div>
```

## Color System

### HSL color format

Colors use HSL (Hue, Saturation, Lightness) format for easy theming:
- **Hue:** 0-360 (color wheel position)
- **Saturation:** 0-100% (color intensity)
- **Lightness:** 0-100% (brightness)

### CSS variable naming

```css
--background      /* Main background */
--foreground      /* Main text color */
--primary         /* Primary action color */
--primary-foreground  /* Text on primary */
--secondary       /* Secondary color */
--muted          /* Subtle backgrounds */
--accent         /* Highlights */
--destructive    /* Error/danger */
--border         /* Borders */
--ring           /* Focus rings */
--radius         /* Border radius */
```

### Light vs dark mode

```css
:root {
  /* Light mode values */
  --background: 0 0% 100%;
  --foreground: 222.2 84% 4.9%;
}

.dark {
  /* Dark mode values */
  --background: 222.2 84% 4.9%;
  --foreground: 210 40% 98%;
}
```

## Typography

### Font families

```css
/* System fonts for performance */
font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
```

### Tailwind typography utilities

```typescript
// Use Tailwind's typography scale
<h1 className="text-4xl font-bold">Heading 1</h1>
<h2 className="text-3xl font-semibold">Heading 2</h2>
<p className="text-base">Body text</p>
<p className="text-sm">Small text</p>
```

## Spacing Scale

```typescript
// Tailwind's default spacing scale
p-1: 0.25rem (4px)
p-2: 0.5rem (8px)
p-4: 1rem (16px)
p-6: 1.5rem (24px)
p-8: 2rem (32px)
```

## Border Radius

```css
/* Using CSS variable for consistency */
--radius: 0.5rem;

/* Derived values */
rounded-sm: calc(var(--radius) - 4px)  /* 0.25rem */
rounded-md: calc(var(--radius) - 2px)  /* 0.375rem */
rounded-lg: var(--radius)               /* 0.5rem */
```

## Component Styling

### Button variants

```typescript
// Primary button
<Button className="bg-primary text-primary-foreground hover:bg-primary/90">
  Primary
</Button>

// Secondary button
<Button className="bg-secondary text-secondary-foreground hover:bg-secondary/80">
  Secondary
</Button>

// Destructive button
<Button className="bg-destructive text-destructive-foreground hover:bg-destructive/90">
  Delete
</Button>

// Outline button
<Button className="border border-input bg-background hover:bg-accent">
  Outline
</Button>

// Ghost button
<Button className="hover:bg-accent hover:text-accent-foreground">
  Ghost
</Button>
```

### Input styling

```typescript
<Input className="border-input bg-background ring-offset-background" />
```

### Card styling

```typescript
<Card className="border-border bg-card text-card-foreground">
  <CardHeader>
    <CardTitle>Title</CardTitle>
  </CardHeader>
  <CardContent>Content</CardContent>
</Card>
```

## Responsive Design

### Breakpoints

```typescript
// Tailwind default breakpoints
sm: 640px
md: 768px
lg: 1024px
xl: 1280px
2xl: 1536px
```

### Responsive utilities

```typescript
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3">
  {/* Responsive grid */}
</div>
```

## Accessibility

### Focus states

```typescript
// Focus ring automatically applied via CSS variables
<Button className="focus-visible:ring-2 focus-visible:ring-ring">
  Accessible button
</Button>
```

### Contrast ratios

Ensure color combinations meet WCAG AA standards:
- Normal text: 4.5:1 contrast ratio
- Large text: 3:1 contrast ratio
- UI components: 3:1 contrast ratio

## Best Practices

### Always use CSS variables

```typescript
// Good
<div className="bg-primary text-primary-foreground">

// Avoid
<div className="bg-blue-500 text-white">
```

### Use cn() utility for conditional classes

```typescript
import { cn } from '@/shared/utils';

<div className={cn('base-class', isActive && 'active-class', className)}>
  Content
</div>
```

### Test dark mode

```typescript
// Always test components in both light and dark modes
<div className="dark">
  <Component />
</div>
```

### Keep color palette minimal

```typescript
// Use semantic color names instead of arbitrary values
--primary
--secondary
--muted
--destructive
```

### Use shadcn/ui components as base

```typescript
// Extend shadcn/ui components instead of building from scratch
import { Button } from '@/presentation/components/ui/button';

<Button className="custom-class">Custom Button</Button>
```
