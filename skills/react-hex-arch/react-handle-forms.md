---
name: react-handle-forms
description: Handle forms with react-hook-form, zodResolver, and zod validation. Separate presentation logic from form logic using custom hooks. Use when creating forms, implementing validation, or separating form state management from UI components.
---

# React Handle Forms

## Quick start

```typescript
// 1. Define form schema with zod
// src/presentation/features/contracts/types.ts
import { z } from 'zod';

export const contractFormSchema = z.object({
  name: z.string().min(3, 'Name must be at least 3 characters'),
  email: z.string().email('Invalid email'),
  status: z.enum(['draft', 'active', 'terminated']),
});

export type ContractFormValues = z.infer<typeof contractFormSchema>;

// 2. Create custom hook for form logic
// src/presentation/features/contracts/hooks/useContractForm.ts
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { contractFormSchema, ContractFormValues } from '../types';

const defaultFormValues: Partial<ContractFormValues> = {
  status: 'draft',
};

export const useContractForm = ({
  defaultValues: externalDefaultValues,
  onSubmit,
}: {
  defaultValues?: Partial<ContractFormValues>;
  onSubmit: (data: ContractFormValues) => void;
}) => {
  const defaultValues = {
    ...defaultFormValues,
    ...externalDefaultValues,
  };

  const form = useForm<ContractFormValues>({
    resolver: zodResolver(contractFormSchema),
    defaultValues,
  });

  const handleSubmit = async (formData: ContractFormValues) => {
    await onSubmit(formData);
  };

  return {
    form,
    handleSubmit: form.handleSubmit(handleSubmit),
    isSubmitting: form.formState.isSubmitting,
  };
};

// 3. Create presentation component
// src/presentation/features/contracts/components/ContractForm.tsx
import { Form, FormField, FormItem, FormLabel, FormControl, FormMessage } from '@/presentation/components/ui/form';
import { Input } from '@/presentation/components/ui/input';
import { useContractForm } from '../hooks/useContractForm';
import { ContractFormValues } from '../types';

export const ContractForm = ({
  defaultValues,
  onSubmit,
}: {
  defaultValues?: Partial<ContractFormValues>;
  onSubmit: (data: ContractFormValues) => void;
}) => {
  const { form, handleSubmit, isSubmitting } = useContractForm({
    defaultValues,
    onSubmit,
  });

  return (
    <Form {...form}>
      <form onSubmit={handleSubmit} className="space-y-6">
        <FormField
          control={form.control}
          name="name"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Name</FormLabel>
              <FormControl>
                <Input {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
      </form>
    </Form>
  );
};
```

## Workflows

### Create form with separated logic and presentation

**Step 1: Define types and schema**

```typescript
// src/presentation/features/feature/types.ts
import { z } from 'zod';

export const featureFormSchema = z.object({
  name: z.string().min(3, 'Name must be at least 3 characters'),
  description: z.string().min(10, 'Description must be at least 10 characters'),
  status: z.enum(['draft', 'active', 'completed']),
  priority: z.enum(['low', 'medium', 'high']),
});

export type FeatureFormValues = z.infer<typeof featureFormSchema>;
```

**Step 2: Create custom hook**

```typescript
// src/presentation/features/feature/hooks/useFeatureForm.ts
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { featureFormSchema, FeatureFormValues } from '../types';

// Default values outside component to prevent recreation
const defaultFormValues: Partial<FeatureFormValues> = {
  status: 'draft',
  priority: 'medium',
};

export const useFeatureForm = ({
  defaultValues: externalDefaultValues,
  onSubmit,
  isViewMode = false,
}: {
  defaultValues?: Partial<FeatureFormValues>;
  onSubmit: (data: FeatureFormValues) => void;
  isViewMode?: boolean;
}) => {
  const defaultValues = {
    ...defaultFormValues,
    ...externalDefaultValues,
  };

  const form = useForm<FeatureFormValues>({
    resolver: zodResolver(featureFormSchema),
    defaultValues,
  });

  const handleSubmit = async (formData: FeatureFormValues) => {
    try {
      await onSubmit(formData);
    } catch (error) {
      console.error('Error submitting form:', error);
      throw error;
    }
  };

  return {
    form,
    handleSubmit: form.handleSubmit(handleSubmit) as (e?: React.BaseSyntheticEvent) => Promise<void>,
    isSubmitting: form.formState.isSubmitting,
    isViewMode,
  };
};

export default useFeatureForm;
```

**Step 3: Create presentation component**

```typescript
// src/presentation/features/feature/components/FeatureForm.tsx
import { Button } from '@/presentation/components/ui/button';
import { Input } from '@/presentation/components/ui/input';
import { Form, FormField, FormItem, FormLabel, FormControl, FormMessage } from '@/presentation/components/ui/form';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/presentation/components/ui/select';
import { useFeatureForm } from '../hooks/useFeatureForm';
import { FeatureFormValues } from '../types';

export const FeatureForm = ({
  defaultValues,
  onSubmit,
  isViewMode = false,
}: {
  defaultValues?: Partial<FeatureFormValues>;
  onSubmit: (data: FeatureFormValues) => void;
  isViewMode?: boolean;
}) => {
  const { form, handleSubmit, isSubmitting } = useFeatureForm({
    defaultValues,
    onSubmit,
    isViewMode,
  });

  return (
    <Form {...form}>
      <form onSubmit={handleSubmit} className="space-y-6">
        <FormField
          control={form.control}
          name="name"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Name</FormLabel>
              <FormControl>
                <Input {...field} disabled={isViewMode} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <FormField
          control={form.control}
          name="description"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Description</FormLabel>
              <FormControl>
                <Input {...field} disabled={isViewMode} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <FormField
          control={form.control}
          name="status"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Status</FormLabel>
              <Select onValueChange={field.onChange} value={field.value} disabled={isViewMode}>
                <FormControl>
                  <SelectTrigger>
                    <SelectValue placeholder="Select status" />
                  </SelectTrigger>
                </FormControl>
                <SelectContent>
                  <SelectItem value="draft">Draft</SelectItem>
                  <SelectItem value="active">Active</SelectItem>
                  <SelectItem value="completed">Completed</SelectItem>
                </SelectContent>
              </Select>
              <FormMessage />
            </FormItem>
          )}
        />

        <FormField
          control={form.control}
          name="priority"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Priority</FormLabel>
              <Select onValueChange={field.onChange} value={field.value} disabled={isViewMode}>
                <FormControl>
                  <SelectTrigger>
                    <SelectValue placeholder="Select priority" />
                  </SelectTrigger>
                </FormControl>
                <SelectContent>
                  <SelectItem value="low">Low</SelectItem>
                  <SelectItem value="medium">Medium</SelectItem>
                  <SelectItem value="high">High</SelectItem>
                </SelectContent>
              </Select>
              <FormMessage />
            </FormItem>
          )}
        />

        <Button type="submit" disabled={isSubmitting}>
          {isSubmitting ? 'Saving...' : 'Save'}
        </Button>
      </form>
    </Form>
  );
};
```

**Step 4: Use in page component**

```typescript
// src/presentation/pages/feature/FeatureFormPage.tsx
import { useAppDispatch } from '@/app/store/hooks';
import { createFeature, updateFeature } from '@/app/store/slices/featureSlice';
import { FeatureForm } from '../components/FeatureForm';
import type { FeatureFormValues } from '../types';

export const FeatureFormPage = () => {
  const dispatch = useAppDispatch();
  const isEditMode = !!initialData;

  const handleSubmit = async (formData: FeatureFormValues) => {
    if (isEditMode) {
      await dispatch(updateFeature({ id, ...formData })).unwrap();
    } else {
      await dispatch(createFeature(formData)).unwrap();
    }
  };

  return (
    <FeatureForm
      defaultValues={initialData}
      onSubmit={handleSubmit}
    />
  );
};
```

## Form Validation with Zod

### String validation

```typescript
z.string()
  .min(3, 'Must be at least 3 characters')
  .max(100, 'Must be less than 100 characters')
  .email('Invalid email format')
```

### Number validation

```typescript
z.coerce.number()
  .min(1, 'Must be at least 1')
  .max(100, 'Must be less than 100')
```

### Enum validation

```typescript
z.enum(['draft', 'active', 'completed'])
```

### Optional fields

```typescript
z.string().optional()
z.string().nullable()
```

### Object validation

```typescript
z.object({
  name: z.string(),
  address: z.object({
    street: z.string(),
    city: z.string(),
  }),
})
```

### Array validation

```typescript
z.array(z.string())
z.array(z.object({ id: z.number(), name: z.string() }))
```

## Form Field Types

### Text input

```typescript
<FormField
  control={form.control}
  name="fieldName"
  render={({ field }) => (
    <FormItem>
      <FormLabel>Field Name</FormLabel>
      <FormControl>
        <Input {...field} />
      </FormControl>
      <FormMessage />
    </FormItem>
  )}
/>
```

### Number input

```typescript
<FormField
  control={form.control}
  name="amount"
  render={({ field }) => (
    <FormItem>
      <FormLabel>Amount</FormLabel>
      <FormControl>
        <Input
          type="number"
          {...field}
          onChange={(e) => field.onChange(Number(e.target.value))}
        />
      </FormControl>
      <FormMessage />
    </FormItem>
  )}
/>
```

### Select dropdown

```typescript
<FormField
  control={form.control}
  name="status"
  render={({ field }) => (
    <FormItem>
      <FormLabel>Status</FormLabel>
      <Select onValueChange={field.onChange} value={field.value}>
        <FormControl>
          <SelectTrigger>
            <SelectValue placeholder="Select status" />
          </SelectTrigger>
        </FormControl>
        <SelectContent>
          <SelectItem value="draft">Draft</SelectItem>
          <SelectItem value="active">Active</SelectItem>
        </SelectContent>
      </Select>
      <FormMessage />
    </FormItem>
  )}
/>
```

### Textarea

```typescript
<FormField
  control={form.control}
  name="description"
  render={({ field }) => (
    <FormItem>
      <FormLabel>Description</FormLabel>
      <FormControl>
        <textarea
          className="flex min-h-[100px] w-full rounded-md border border-input bg-background px-3 py-2"
          {...field}
        />
      </FormControl>
      <FormMessage />
    </FormItem>
  )}
/>
```

## Prevent Enter Key Submit

```typescript
<form onSubmit={handleSubmit} onKeyDown={(e) => {
  if (e.key === 'Enter') {
    e.preventDefault();
  }
}}>
  {/* Form fields */}
</form>
```

## Form State Management

### Access form values

```typescript
const form = useForm<FormData>({ ... });
const values = form.watch();
const specificValue = form.watch('fieldName');
```

### Set form values programmatically

```typescript
form.setValue('fieldName', 'value');
form.setValue('fieldName', value, { shouldValidate: true });
```

### Reset form

```typescript
form.reset();
form.reset(defaultValues);
```

### Trigger validation

```typescript
form.trigger();
form.trigger('fieldName');
```

### Get form errors

```typescript
const errors = form.formState.errors;
const fieldError = form.formState.errors.fieldName;
```

## Pattern: Logic vs Presentation Separation

**Logic Layer (useContractForm.ts)**:
- Form configuration (resolver, defaultValues)
- Form submission handling
- Data transformation before submission
- Form state (isSubmitting, isViewMode)

**Presentation Layer (ContractForm.tsx)**:
- UI rendering
- FormField components
- Shadcn UI components
- Display logic (disabled states, conditional rendering)
