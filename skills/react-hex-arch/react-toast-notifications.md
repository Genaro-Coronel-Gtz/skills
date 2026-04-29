---
name: react-toast-notifications
description: Display toast notifications for user feedback using useToast hook. Show success, error, info, and warning messages with custom styling and auto-dismiss. Use when providing user feedback after actions, displaying alerts, or showing notifications.
---

# React Toast Notifications

## Quick start

```typescript
// Import useToast hook
import { useToast } from '@/presentation/hooks/use-toast';

export const MyComponent = () => {
  const { toast } = useToast();

  const handleAction = () => {
    toast({
      title: "¡Éxito!",
      description: "La operación se completó correctamente",
      variant: "default",
    });
  };

  return <button onClick={handleAction}>Acción</button>;
};
```

## Workflows

### Success notification

```typescript
const { toast } = useToast();

toast({
  title: "¡Éxito!",
  description: "El elemento se creó correctamente",
  variant: "default",
  className: "border-green-500 bg-green-50 text-green-700 dark:bg-green-950 dark:border-green-500 dark:text-green-300"
});
```

### Error notification

```typescript
const { toast } = useToast();

toast({
  title: "Error",
  description: "Ocurrió un error al procesar la solicitud",
  variant: "destructive",
});
```

### Info notification

```typescript
const { toast } = useToast();

toast({
  title: "Información",
  description: "El elemento está siendo procesado",
  variant: "default",
});
```

### Warning notification

```typescript
const { toast } = useToast();

toast({
  title: "Advertencia",
  description: "Esta acción no se puede deshacer",
  variant: "default",
  className: "border-yellow-500 bg-yellow-50 text-yellow-700"
});
```

### Toast with action

```typescript
const { toast } = useToast();

toast({
  title: "Confirmación requerida",
  description: "¿Deseas continuar con esta acción?",
  action: (
    <Button variant="outline" size="sm">
      Confirmar
    </Button>
  ),
});
```

### Dismiss toast manually

```typescript
const { toast, dismiss } = useToast();

const toastId = toast({
  title: "Notificación",
  description: "Mensaje importante",
});

// Dismiss programmatically
setTimeout(() => {
  dismiss(toastId.id);
}, 5000);
```

### Toast in async operations

```typescript
const handleSubmit = async (data: FormData) => {
  try {
    await dispatch(createItem(data)).unwrap();
    toast({
      title: "¡Éxito!",
      description: "Creado correctamente",
      variant: "default",
    });
  } catch (error: unknown) {
    const errorMessage = error instanceof Error ? error.message : 'Error inesperado';
    toast({
      title: "Error",
      description: errorMessage,
      variant: "destructive",
    });
  }
};
```

### Toast with custom duration

```typescript
const { toast } = useToast();

toast({
  title: "Notificación",
  description: "Este mensaje desaparecerá en 10 segundos",
  duration: 10000,
});
```

## Toast Variants

### Default (info/success)

```typescript
toast({
  title: "Información",
  description: "Mensaje informativo",
  variant: "default",
});
```

### Destructive (error)

```typescript
toast({
  title: "Error",
  description: "Mensaje de error",
  variant: "destructive",
});
```

## Custom Styling

### Green success style

```typescript
toast({
  title: "¡Éxito!",
  description: "Operación completada",
  variant: "default",
  className: "border-green-500 bg-green-50 text-green-700 dark:bg-green-950 dark:border-green-500 dark:text-green-300"
});
```

### Yellow warning style

```typescript
toast({
  title: "Advertencia",
  description: "Ten cuidado",
  variant: "default",
  className: "border-yellow-500 bg-yellow-50 text-yellow-700"
});
```

### Blue info style

```typescript
toast({
  title: "Información",
  description: "Detalles adicionales",
  variant: "default",
  className: "border-blue-500 bg-blue-50 text-blue-700"
});
```

## Use Cases

### After form submission

```typescript
const handleSubmit = async (formData: CreateDto) => {
  try {
    await dispatch(createItem(formData)).unwrap();
    toast({
      title: "¡Éxito!",
      description: "Creado correctamente",
      variant: "default",
    });
    navigate('/items');
  } catch (error) {
    toast({
      title: "Error",
      description: "No se pudo crear el elemento",
      variant: "destructive",
    });
  }
};
```

### After delete action

```typescript
const handleDelete = async (id: number) => {
  if (window.confirm('¿Estás seguro?')) {
    try {
      await dispatch(deleteItem(id.toString())).unwrap();
      toast({
        title: "¡Éxito!",
        description: "Eliminado correctamente",
        variant: "default",
      });
    } catch (error) {
      toast({
        title: "Error",
        description: "No se pudo eliminar",
        variant: "destructive",
      });
    }
  }
};
```

### After update action

```typescript
const handleUpdate = async (id: string, data: UpdateDto) => {
  try {
    await dispatch(updateItem({ id, ...data })).unwrap();
    toast({
      title: "¡Éxito!",
      description: "Actualizado correctamente",
      variant: "default",
    });
  } catch (error) {
    toast({
      title: "Error",
      description: "No se pudo actualizar",
      variant: "destructive",
    });
  }
};
```

### Error handling in useEffect

```typescript
useEffect(() => {
  dispatch(fetchItem(id)).unwrap().catch((error) => {
    toast({
      title: "Error",
      description: `Error al cargar: ${error.message}`,
      variant: "destructive",
    });
    navigate('/items');
  });
}, [dispatch, id, navigate]);
```

### Validation errors

```typescript
const handleSubmit = async (data: FormData) => {
  if (!data.name) {
    toast({
      title: "Error de validación",
      description: "El nombre es requerido",
      variant: "destructive",
    });
    return;
  }
  // Continue with submission
};
```

## Toaster Component

### Add Toaster to app

```typescript
// src/App.tsx
import { Toaster } from '@/presentation/components/ui/toaster';

export const App = () => {
  return (
    <>
      <AppRouter />
      <Toaster />
    </>
  );
};
```

## Toast Props

```typescript
interface ToastProps {
  id?: string;
  title?: React.ReactNode;
  description?: React.ReactNode;
  action?: ToastActionElement;
  variant?: 'default' | 'destructive';
  className?: string;
  duration?: number;
  onOpenChange?: (open: boolean) => void;
}
```

## Best Practices

### Always provide title

```typescript
toast({
  title: "¡Éxito!", // Always include
  description: "Operación completada",
});
```

### Use descriptive messages

```typescript
// Good
toast({
  title: "¡Éxito!",
  description: "La propiedad se ha creado correctamente",
});

// Avoid
toast({
  title: "OK",
});
```

### Match variant to context

```typescript
// Success/Info
toast({ variant: "default" });

// Error
toast({ variant: "destructive" });
```

### Dismiss on navigation

```typescript
useEffect(() => {
  return () => {
    // Clear toasts when component unmounts
    dismiss();
  };
}, []);
```
