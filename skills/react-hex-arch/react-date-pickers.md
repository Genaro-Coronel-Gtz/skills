---
name: react-date-pickers
description: Handle date selection and formatting using date-fns with shadcn/ui Calendar and Popover components. Use when implementing date inputs, date ranges, or any date selection in forms.
---

# React Date Pickers

## Quick start

```typescript
import { useState } from 'react';
import { format } from 'date-fns';
import { Calendar } from '@/presentation/components/ui/calendar';
import { Popover, PopoverContent, PopoverTrigger } from '@/presentation/components/ui/popover';
import { Button } from '@/presentation/components/ui/button';
import { Calendar as CalendarIcon } from 'lucide-react';
import { cn } from '@/presentation/lib/utils';

export const DatePicker = ({ value, onChange }: { value: Date | null; onChange: (date: Date | null) => void }) => {
  const [open, setOpen] = useState(false);

  return (
    <Popover open={open} onOpenChange={setOpen}>
      <PopoverTrigger asChild>
        <Button
          variant="outline"
          className={cn('w-full justify-start text-left font-normal', !value && 'text-muted-foreground')}
        >
          <CalendarIcon className="mr-2 h-4 w-4" />
          {value ? format(value, 'PPP') : 'Seleccionar fecha'}
        </Button>
      </PopoverTrigger>
      <PopoverContent className="w-auto p-0" align="start">
        <Calendar
          mode="single"
          selected={value || undefined}
          onSelect={(date) => {
            onChange(date);
            setOpen(false);
          }}
          initialFocus
        />
      </PopoverContent>
    </Popover>
  );
};
```

## Workflows

### Date picker in form with react-hook-form

```typescript
// In custom hook
import { format, parseISO } from 'date-fns';

export const useContractForm = ({ defaultValues, onSubmit }: any) => {
  const [date, setDate] = useState<{
    startDate: Date | null;
    endDate: Date | null;
  }>(() => ({
    startDate: defaultValues?.startDate ? parseISO(defaultValues.startDate) : null,
    endDate: defaultValues?.endDate ? parseISO(defaultValues.endDate) : null,
  }));

  useEffect(() => {
    if (date.startDate) {
      form.setValue('startDate', format(date.startDate, 'yyyy-MM-dd'), { shouldValidate: true });
    }
    if (date.endDate) {
      form.setValue('endDate', format(date.endDate, 'yyyy-MM-dd'), { shouldValidate: true });
    }
  }, [date, form]);

  return { form, date, setDate, handleSubmit };
};

// In component
<FormField
  control={form.control}
  name="startDate"
  render={({ field }) => (
    <FormItem>
      <FormLabel>Fecha de inicio</FormLabel>
      <Popover>
        <PopoverTrigger asChild>
          <FormControl>
            <Button variant="outline">
              {date.startDate ? format(date.startDate, 'PPP') : 'Seleccionar'}
            </Button>
          </FormControl>
        </PopoverTrigger>
        <PopoverContent className="w-auto p-0">
          <Calendar
            mode="single"
            selected={date.startDate || undefined}
            onSelect={(date) => setDate({ ...date, startDate: date })}
          />
        </PopoverContent>
      </Popover>
    </FormItem>
  )}
/>
```

### Date range picker

```typescript
import { useState } from 'react';
import { format } from 'date-fns';
import { Calendar } from '@/presentation/components/ui/calendar';
import { Popover, PopoverContent, PopoverTrigger } from '@/presentation/components/ui/popover';

export const DateRangePicker = ({ value, onChange }: { value: { from?: Date; to?: Date }; onChange: (range: { from?: Date; to?: Date }) => void }) => {
  const [open, setOpen] = useState(false);

  return (
    <Popover open={open} onOpenChange={setOpen}>
      <PopoverTrigger asChild>
        <Button variant="outline" className="w-full justify-start">
          {value.from ? (
            value.to ? (
              `${format(value.from, 'PPP')} - ${format(value.to, 'PPP')}`
            ) : (
              format(value.from, 'PPP')
            )
          ) : (
            'Seleccionar rango'
          )}
        </Button>
      </PopoverTrigger>
      <PopoverContent className="w-auto p-0" align="start">
        <Calendar
          mode="range"
          selected={{ from: value.from, to: value.to }}
          onSelect={(range) => {
            onChange(range || { from: undefined, to: undefined });
          }}
          numberOfMonths={2}
        />
      </PopoverContent>
    </Popover>
  );
};
```

### Date formatting with date-fns

```typescript
import { format, parseISO, isValid } from 'date-fns';

// Format date for display
const displayDate = format(new Date(), 'PPP'); // "April 27, 2026"
const shortDate = format(new Date(), 'dd/MM/yyyy'); // "27/04/2026"
const isoDate = format(new Date(), 'yyyy-MM-dd'); // "2026-04-27"

// Parse ISO string to Date
const date = parseISO('2026-04-27T00:00:00Z');

// Validate date
if (isValid(date)) {
  // Date is valid
}

// Format for API
const apiDate = format(date, 'yyyy-MM-dd');
```

### Date validation with zod

```typescript
import { z } from 'zod';

const contractFormSchema = z.object({
  startDate: z.string().min(1, 'Fecha de inicio requerida'),
  endDate: z.string().optional(),
}).refine((data) => {
  if (data.startDate && data.endDate) {
    const start = new Date(data.startDate);
    const end = new Date(data.endDate);
    return start <= end;
  }
  return true;
}, {
  message: 'La fecha de fin debe ser posterior a la fecha de inicio',
});

type ContractFormValues = z.infer<typeof contractFormSchema>;
```

### Date field in form

```typescript
<FormField
  control={form.control}
  name="startDate"
  render={({ field }) => (
    <FormItem>
      <FormLabel>Fecha de inicio</FormLabel>
      <Popover>
        <PopoverTrigger asChild>
          <FormControl>
            <Button variant="outline" className="w-full justify-start text-left">
              <CalendarIcon className="mr-2 h-4 w-4" />
              {field.value ? format(parseISO(field.value), 'PPP') : 'Seleccionar fecha'}
            </Button>
          </FormControl>
        </PopoverTrigger>
        <PopoverContent className="w-auto p-0">
          <Calendar
            mode="single"
            selected={field.value ? parseISO(field.value) : undefined}
            onSelect={(date) => {
              field.onChange(format(date!, 'yyyy-MM-dd'));
            }}
          />
        </PopoverContent>
      </Popover>
      <FormMessage />
    </FormItem>
  )}
/>
```

### Date with time picker

```typescript
import { useState } from 'react';
import { format } from 'date-fns';

export const DateTimePicker = ({ value, onChange }: { value: Date | null; onChange: (date: Date | null) => void }) => {
  const [date, setDate] = useState<Date | null>(value);
  const [time, setTime] = useState(value ? format(value, 'HH:mm') : '00:00');

  const handleDateChange = (newDate: Date | null) => {
    if (newDate && time) {
      const [hours, minutes] = time.split(':');
      const combined = new Date(newDate);
      combined.setHours(parseInt(hours), parseInt(minutes));
      onChange(combined);
    }
    setDate(newDate);
  };

  const handleTimeChange = (newTime: string) => {
    setTime(newTime);
    if (date) {
      const [hours, minutes] = newTime.split(':');
      const combined = new Date(date);
      combined.setHours(parseInt(hours), parseInt(minutes));
      onChange(combined);
    }
  };

  return (
    <div className="flex gap-2">
      <DatePicker value={date} onChange={handleDateChange} />
      <input
        type="time"
        value={time}
        onChange={(e) => handleTimeChange(e.target.value)}
        className="border rounded px-2"
      />
    </div>
  );
};
```

## Date Formats

### Common formats

```typescript
format(date, 'PPP')        // "April 27, 2026"
format(date, 'dd/MM/yyyy')  // "27/04/2026"
format(date, 'yyyy-MM-dd')  // "2026-04-27"
format(date, 'MM/dd/yyyy')  // "04/27/2026"
format(date, 'HH:mm')       // "14:30"
format(date, 'hh:mm a')     // "02:30 PM"
```

### Locale formats

```typescript
import { es } from 'date-fns/locale';

format(date, 'PPP', { locale: es }); // "27 de abril de 2026"
```

## Date Utilities

### Get today's date

```typescript
const today = new Date();
const startOfDay = new Date(today.setHours(0, 0, 0, 0));
const endOfDay = new Date(today.setHours(23, 59, 59, 999));
```

### Add/subtract days

```typescript
import { addDays, subDays } from 'date-fns';

const tomorrow = addDays(new Date(), 1);
const yesterday = subDays(new Date(), 1);
```

### Date comparison

```typescript
import { isAfter, isBefore, isSameDay } from 'date-fns';

isAfter(date1, date2);
isBefore(date1, date2);
isSameDay(date1, date2);
```

## Best Practices

### Store dates as ISO strings in forms

```typescript
// Good
startDate: '2026-04-27'
endDate: '2026-05-27'

// Avoid
startDate: new Date()
endDate: Date object
```

### Format dates for display

```typescript
// Convert ISO to display
const displayDate = format(parseISO(isoDate), 'PPP');
```

### Validate dates before submission

```typescript
const isValidDate = (dateString: string) => {
  const date = new Date(dateString);
  return isValid(date);
};
```

### Handle null/undefined dates

```typescript
const displayDate = value ? format(parseISO(value), 'PPP') : 'No seleccionada';
```
