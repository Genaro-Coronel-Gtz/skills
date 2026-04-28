---
name: react-modals-dialogs
description: Create modals and dialogs using shadcn/ui Dialog components for confirmations, forms, and user interactions. Use when requiring user confirmation, displaying forms in modals, or showing additional information.
---

# React Modals Dialogs

## Quick start

```typescript
// src/presentation/components/dialogs/ConfirmDialog.tsx
import { AlertDialog, AlertDialogAction, AlertDialogCancel, AlertDialogContent, AlertDialogDescription, AlertDialogFooter, AlertDialogHeader, AlertDialogTitle } from '@/presentation/components/ui/alert-dialog';

interface ConfirmDialogProps {
  isOpen: boolean;
  onClose: () => void;
  onConfirm: () => void;
  title: string;
  description: string;
  confirmText?: string;
  cancelText?: string;
}

export const ConfirmDialog = ({
  isOpen,
  onClose,
  onConfirm,
  title,
  description,
  confirmText = 'Confirmar',
  cancelText = 'Cancelar',
}: ConfirmDialogProps) => {
  return (
    <AlertDialog open={isOpen} onOpenChange={onClose}>
      <AlertDialogContent>
        <AlertDialogHeader>
          <AlertDialogTitle>{title}</AlertDialogTitle>
          <AlertDialogDescription>{description}</AlertDialogDescription>
        </AlertDialogHeader>
        <AlertDialogFooter>
          <AlertDialogCancel onClick={onClose}>{cancelText}</AlertDialogCancel>
          <AlertDialogAction onClick={onConfirm}>{confirmText}</AlertDialogAction>
        </AlertDialogFooter>
      </AlertDialogContent>
    </AlertDialog>
  );
};
```

## Workflows

### Delete confirmation dialog

```typescript
// In component
import { ConfirmDialog } from '@/presentation/components/dialogs/ConfirmDialog';
import { useState } from 'react';

export const FeaturesPage = () => {
  const [deleteDialog, setDeleteDialog] = useState<{ isOpen: boolean; itemId: number | null }>({
    isOpen: false,
    itemId: null,
  });

  const handleDelete = async (id: number) => {
    setDeleteDialog({ isOpen: true, itemId: id });
  };

  const confirmDelete = async () => {
    if (deleteDialog.itemId) {
      await dispatch(deleteFeature(deleteDialog.itemId.toString())).unwrap();
      setDeleteDialog({ isOpen: false, itemId: null });
      toast({ title: "¡Éxito!", description: "Eliminado correctamente" });
    }
  };

  return (
    <div>
      <Button onClick={() => handleDelete(feature.id)}>Eliminar</Button>
      
      <ConfirmDialog
        isOpen={deleteDialog.isOpen}
        onClose={() => setDeleteDialog({ isOpen: false, itemId: null })}
        onConfirm={confirmDelete}
        title="¿Eliminar elemento?"
        description="Esta acción no se puede deshacer. ¿Estás seguro de que deseas eliminar este elemento?"
        confirmText="Eliminar"
        cancelText="Cancelar"
      />
    </div>
  );
};
```

### Form dialog

```typescript
// src/presentation/components/dialogs/FormDialog.tsx
import { Dialog, DialogContent, DialogDescription, DialogFooter, DialogHeader, DialogTitle } from '@/presentation/components/ui/dialog';

interface FormDialogProps {
  isOpen: boolean;
  onClose: () => void;
  title: string;
  description?: string;
  children: React.ReactNode;
  footer?: React.ReactNode;
}

export const FormDialog = ({
  isOpen,
  onClose,
  title,
  description,
  children,
  footer,
}: FormDialogProps) => {
  return (
    <Dialog open={isOpen} onOpenChange={onClose}>
      <DialogContent className="sm:max-w-[600px]">
        <DialogHeader>
          <DialogTitle>{title}</DialogTitle>
          {description && <DialogDescription>{description}</DialogDescription>}
        </DialogHeader>
        {children}
        {footer && <DialogFooter>{footer}</DialogFooter>}
      </DialogContent>
    </Dialog>
  );
};
```

### Use form dialog

```typescript
import { FormDialog } from '@/presentation/components/dialogs/FormDialog';
import { FeatureForm } from '@/presentation/features/feature/components/FeatureForm';

export const FeaturesPage = () => {
  const [formDialog, setFormDialog] = useState({ isOpen: false, mode: 'create' as 'create' | 'edit', itemId: null as number | null });

  const handleNew = () => {
    setFormDialog({ isOpen: true, mode: 'create', itemId: null });
  };

  const handleEdit = (id: number) => {
    setFormDialog({ isOpen: true, mode: 'edit', itemId: id });
  };

  const handleSubmit = async (data: CreateFeatureDto) => {
    if (formDialog.mode === 'create') {
      await dispatch(createFeature(data)).unwrap();
    } else {
      await dispatch(updateFeature({ id: formDialog.itemId!, ...data })).unwrap();
    }
    setFormDialog({ isOpen: false, mode: 'create', itemId: null });
  };

  return (
    <div>
      <Button onClick={handleNew}>Nuevo</Button>
      
      <FormDialog
        isOpen={formDialog.isOpen}
        onClose={() => setFormDialog({ isOpen: false, mode: 'create', itemId: null })}
        title={formDialog.mode === 'create' ? 'Nuevo Elemento' : 'Editar Elemento'}
        footer={<Button type="submit">Guardar</Button>}
      >
        <FeatureForm onSubmit={handleSubmit} />
      </FormDialog>
    </div>
  );
};
```

## Dialog Patterns

### Confirmation with destructive action

```typescript
<AlertDialogAction onClick={onConfirm} className="bg-red-600 hover:bg-red-700">
  Eliminar
</AlertDialogAction>
```

### Dialog with form submission

```typescript
<FormDialog
  isOpen={isOpen}
  onClose={onClose}
  title="Nuevo Elemento"
  footer={
    <div className="flex justify-end space-x-2">
      <Button variant="outline" onClick={onClose}>Cancelar</Button>
      <Button type="submit">Guardar</Button>
    </div>
  }
>
  <form onSubmit={handleSubmit}>
    <FeatureForm />
  </form>
</FormDialog>
```

### Full-width dialog

```typescript
<DialogContent className="sm:max-w-full">
  {/* Content */}
</DialogContent>
```

### Scrollable dialog

```typescript
<DialogContent className="max-h-[80vh] overflow-y-auto">
  {/* Content */}
</DialogContent>
```

## Best Practices

### Always provide clear title and description

```typescript
<AlertDialogHeader>
  <AlertDialogTitle>¿Eliminar elemento?</AlertDialogTitle>
  <AlertDialogDescription>
    Esta acción no se puede deshacer. ¿Estás seguro?
  </AlertDialogDescription>
</AlertDialogHeader>
```

### Use AlertDialog for confirmations

```typescript
// For confirmations (yes/no)
<AlertDialog>
  <AlertDialogContent>
    <AlertDialogHeader>
      <AlertDialogTitle>¿Confirmar?</AlertDialogTitle>
    </AlertDialogHeader>
    <AlertDialogFooter>
      <AlertDialogCancel>Cancelar</AlertDialogCancel>
      <AlertDialogAction>Confirmar</AlertDialogAction>
    </AlertDialogFooter>
  </AlertDialogContent>
</AlertDialog>
```

### Use Dialog for content/forms

```typescript
// For displaying content or forms
<Dialog>
  <DialogContent>
    <DialogHeader>
      <DialogTitle>Detalles</DialogTitle>
    </DialogHeader>
    {/* Content */}
  </DialogContent>
</Dialog>
```

### Close dialog on successful action

```typescript
const handleSubmit = async (data: FormData) => {
  await dispatch(createItem(data)).unwrap();
  setIsOpen(false);
  toast({ title: "¡Éxito!", description: "Creado correctamente" });
};
```
