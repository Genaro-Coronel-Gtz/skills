---
name: Rentix Design System
description: Modern property management UI with shadcn/ui components, Tailwind CSS, and CSS variables for theming. Clean, professional aesthetic with light/dark mode support.
version: "1.0.0"

colors:
  # Light mode (extracted from globals.css :root)
  background: "hsl(0 0% 100%)"
  foreground: "hsl(222.2 84% 4.9%)"
  card: "hsl(0 0% 100%)"
  card-foreground: "hsl(222.2 84% 4.9%)"
  popover: "hsl(0 0% 100%)"
  popover-foreground: "hsl(222.2 84% 4.9%)"
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

  # Dark mode (extracted from globals.css .dark)
  dark-background: "hsl(222.2 84% 4.9%)"
  dark-foreground: "hsl(210 40% 98%)"
  dark-card: "hsl(222.2 84% 4.9%)"
  dark-card-foreground: "hsl(210 40% 98%)"
  dark-popover: "hsl(222.2 84% 4.9%)"
  dark-popover-foreground: "hsl(210 40% 98%)"
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

typography:
  # Font family uses system fonts by default
  fontFamily: "system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif"
  
  # Base font sizes
  h1:
    fontSize: "2.25rem"
    fontWeight: "700"
    lineHeight: "2.5rem"
  h2:
    fontSize: "1.875rem"
    fontWeight: "600"
    lineHeight: "2.25rem"
  h3:
    fontSize: "1.5rem"
    fontWeight: "600"
    lineHeight: "2rem"
  h4:
    fontSize: "1.25rem"
    fontWeight: "600"
    lineHeight: "1.75rem"
  body-lg:
    fontSize: "1.125rem"
    fontWeight: "400"
    lineHeight: "1.75rem"
  body-md:
    fontSize: "1rem"
    fontWeight: "400"
    lineHeight: "1.5rem"
  body-sm:
    fontSize: "0.875rem"
    fontWeight: "400"
    lineHeight: "1.25rem"
  label:
    fontSize: "0.875rem"
    fontWeight: "500"
    lineHeight: "1.25rem"

rounded:
  sm: "calc(var(--radius) - 4px)"
  md: "calc(var(--radius) - 2px)"
  lg: "var(--radius)"
  xl: "0.75rem"
  full: "9999px"

spacing:
  xs: "0.25rem"
  sm: "0.5rem"
  md: "1rem"
  lg: "1.5rem"
  xl: "2rem"
  "2xl": "3rem"

components:
  button-primary:
    backgroundColor: "{colors.primary}"
    textColor: "{colors.primary-foreground}"
    rounded: "{rounded.md}"
    padding: "0.5rem 1rem"
    fontWeight: "500"
    hover:
      backgroundColor: "{colors.primary}"
      opacity: "0.9"
  
  button-secondary:
    backgroundColor: "{colors.secondary}"
    textColor: "{colors.secondary-foreground}"
    rounded: "{rounded.md}"
    padding: "0.5rem 1rem"
    fontWeight: "500"
    hover:
      backgroundColor: "{colors.secondary}"
      opacity: "0.8"
  
  button-destructive:
    backgroundColor: "{colors.destructive}"
    textColor: "{colors.destructive-foreground}"
    rounded: "{rounded.md}"
    padding: "0.5rem 1rem"
    fontWeight: "500"
    hover:
      backgroundColor: "{colors.destructive}"
      opacity: "0.9"
  
  button-outline:
    backgroundColor: "transparent"
    textColor: "{colors.foreground}"
    borderColor: "{colors.border}"
    rounded: "{rounded.md}"
    padding: "0.5rem 1rem"
    fontWeight: "500"
    hover:
      backgroundColor: "{colors.accent}"
      textColor: "{colors.accent-foreground}"
  
  button-ghost:
    backgroundColor: "transparent"
    textColor: "{colors.foreground}"
    rounded: "{rounded.md}"
    padding: "0.5rem 1rem"
    fontWeight: "500"
    hover:
      backgroundColor: "{colors.accent}"
      textColor: "{colors.accent-foreground}"
  
  input:
    backgroundColor: "{colors.background}"
    textColor: "{colors.foreground}"
    borderColor: "{colors.input}"
    rounded: "{rounded.md}"
    padding: "0.5rem 0.75rem"
    focus:
      borderColor: "{colors.ring}"
      ring: "2px solid {colors.ring}"
  
  card:
    backgroundColor: "{colors.card}"
    textColor: "{colors.card-foreground}"
    borderColor: "{colors.border}"
    rounded: "{rounded.lg}"
    padding: "1.5rem"
  
  dialog:
    backgroundColor: "{colors.background}"
    textColor: "{colors.foreground}"
    rounded: "{rounded.lg}"
    padding: "1.5rem"
    maxWidth: "32rem"
  
  table:
    backgroundColor: "{colors.card}"
    textColor: "{colors.card-foreground}"
    borderColor: "{colors.border}"
    rounded: "{rounded.md}"
    header:
      backgroundColor: "{colors.muted}"
      textColor: "{colors.muted-foreground}"
      fontWeight: "600"
    row:
      hover:
        backgroundColor: "{colors.muted}"
  
  badge:
    backgroundColor: "{colors.primary}"
    textColor: "{colors.primary-foreground}"
    rounded: "{rounded.full}"
    padding: "0.25rem 0.75rem"
    fontSize: "0.75rem"
    fontWeight: "500"
  
  alert-info:
    backgroundColor: "{colors.background}"
    borderColor: "{colors.primary}"
    textColor: "{colors.foreground}"
    rounded: "{rounded.md}"
    padding: "1rem"
  
  alert-warning:
    backgroundColor: "{colors.background}"
    borderColor: "#f59e0b"
    textColor: "{colors.foreground}"
    rounded: "{rounded.md}"
    padding: "1rem"
  
  alert-error:
    backgroundColor: "{colors.background}"
    borderColor: "{colors.destructive}"
    textColor: "{colors.foreground}"
    rounded: "{rounded.md}"
    padding: "1rem"
  
  toast:
    backgroundColor: "{colors.background}"
    textColor: "{colors.foreground}"
    borderColor: "{colors.border}"
    rounded: "{rounded.lg}"
    padding: "1rem"
    shadow: "0 4px 6px rgba(0, 0, 0, 0.1)"
  
  toast-destructive:
    backgroundColor: "{colors.destructive}"
    textColor: "{colors.destructive-foreground}"
    rounded: "{rounded.lg}"
    padding: "1rem"
    shadow: "0 4px 6px rgba(0, 0, 0, 0.1)"

---

## Overview

The Rentix Design System is a modern, clean UI built on shadcn/ui components with Tailwind CSS. It emphasizes professional aesthetics, accessibility, and a seamless light/dark mode experience. The system uses CSS variables for theming, making it easy to customize colors and styles across the application.

## Design Philosophy

- **Clean & Professional:** Minimal clutter with clear visual hierarchy
- **Accessibility First:** High contrast ratios and WCAG AA compliance
- **Responsive:** Mobile-first approach with breakpoints at 768px and 1024px
- **Themeable:** CSS variables enable easy color scheme customization
- **Component-Driven:** Reusable shadcn/ui components with consistent styling

## Colors

The color palette uses HSL values stored in CSS variables for easy theming. The system supports both light and dark modes.

### Light Mode
- **Primary (Blue):** `hsl(221.2 83.2% 53.3%)` - Main action color, CTAs, links
- **Secondary (Gray):** `hsl(210 40% 96.1%)` - Secondary actions, backgrounds
- **Muted (Gray):** `hsl(210 40% 96.1%)` - Subtle backgrounds, disabled states
- **Accent (Gray):** `hsl(210 40% 96.1%)` - Hover states, highlights
- **Destructive (Red):** `hsl(0 84.2% 60.2%)` - Delete actions, errors
- **Background (White):** `hsl(0 0% 100%)` - Main background
- **Foreground (Dark):** `hsl(222.2 84% 4.9%)` - Main text
- **Border (Light Gray):** `hsl(214.3 31.8% 91.4%)` - Borders, dividers
- **Card (White):** `hsl(0 0% 100%)` - Card backgrounds
- **Ring (Blue):** `hsl(221.2 83.2% 53.3%)` - Focus rings

### Dark Mode
- **Background (Dark):** `hsl(222.2 84% 4.9%)` - Main background
- **Foreground (Light):** `hsl(210 40% 98%)` - Main text
- **Primary (Light Blue):** `hsl(217.2 91.2% 59.8%)` - Main action color
- **Secondary (Dark Gray):** `hsl(217.2 32.6% 17.5%)` - Secondary backgrounds
- **Muted (Dark Gray):** `hsl(217.2 32.6% 17.5%)` - Subtle backgrounds
- **Destructive (Dark Red):** `hsl(0 62.8% 30.6%)` - Delete actions, errors
- **Border (Dark Gray):** `hsl(217.2 32.6% 17.5%)` - Borders, dividers

## Typography

The system uses system fonts for optimal performance and native feel. Font sizes follow a modular scale for consistency.

### Font Family
System fonts: `system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif`

### Type Scale
- **H1:** 2.25rem (36px) - Page titles
- **H2:** 1.875rem (30px) - Section headings
- **H3:** 1.5rem (24px) - Subsection headings
- **H4:** 1.25rem (20px) - Card titles
- **Body Large:** 1.125rem (18px) - Emphasized body text
- **Body Medium:** 1rem (16px) - Default body text
- **Body Small:** 0.875rem (14px) - Captions, metadata
- **Label:** 0.875rem (14px) - Form labels

## Spacing

Spacing follows a consistent scale based on Tailwind's default values:
- **XS:** 0.25rem (4px)
- **SM:** 0.5rem (8px)
- **MD:** 1rem (16px)
- **LG:** 1.5rem (24px)
- **XL:** 2rem (32px)
- **2XL:** 3rem (48px)

## Border Radius

Corner radius values use CSS variables for consistency:
- **SM:** `calc(var(--radius) - 4px)` - Small elements (badges, tags)
- **MD:** `calc(var(--radius) - 2px)` - Buttons, inputs
- **LG:** `var(--radius)` - Cards, dialogs (default: 0.5rem)
- **XL:** 0.75rem - Large cards
- **Full:** 9999px - Pills, avatars

## Components

### Buttons
- **Primary:** Main action buttons with primary color
- **Secondary:** Secondary actions with gray background
- **Destructive:** Delete/danger actions with red color
- **Outline:** Bordered buttons with transparent background
- **Ghost:** Minimal buttons with hover effect only

### Inputs
- Clean borders with focus rings
- Consistent padding and rounded corners
- Focus state uses ring color for accessibility

### Cards
- White background with subtle border
- Rounded corners for modern look
- Consistent padding for content

### Dialogs
- Centered modal with backdrop
- Maximum width of 32rem
- Rounded corners and padding

### Tables
- Clean headers with muted background
- Hover states on rows
- Consistent border styling

### Badges
- Pill-shaped with full rounded corners
- Primary color by default
- Small font size for labels

### Alerts
- Info, warning, and error variants
- Colored borders for semantic meaning
- Consistent padding and rounded corners

### Toasts
- Floating notifications
- Destructive variant for errors
- Shadow for depth

## Dark Mode

Dark mode is implemented using CSS variables with a `.dark` class on the root element. All color tokens have corresponding dark mode values. Toggle dark mode by adding/removing the `dark` class on the `<html>` element.

## Implementation

### CSS Variables
All colors are defined as HSL values in CSS variables under `:root` and `.dark` selectors in `globals.css`.

### Tailwind Configuration
The Tailwind config references these CSS variables in the `theme.extend.colors` section, allowing Tailwind utilities to use the design tokens.

### Component Usage
Components use Tailwind utility classes that reference the CSS variables, ensuring consistency across the application.

## Customization

To customize the design system:
1. Modify CSS variables in `globals.css`
2. Update color values in the Tailwind config if needed
3. Component styles will automatically reflect changes

## Accessibility

- All color combinations meet WCAG AA contrast ratios
- Focus states use visible ring indicators
- Semantic HTML elements are used throughout
- Keyboard navigation is supported for all interactive elements
