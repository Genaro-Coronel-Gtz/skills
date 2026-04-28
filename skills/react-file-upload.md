---
name: react-file-upload
description: Implement file upload with react-dropzone, progress tracking, and Redux integration. Handle multiple files, file validation, and upload to API. Use when implementing media uploads, document management, or any file-based features.
---

# React File Upload

## Quick start

```typescript
// src/presentation/components/media/MediaUploader.tsx
import { useState, useRef, ChangeEvent } from 'react';
import { useDropzone } from 'react-dropzone';
import { useAppDispatch } from '@/app/store/hooks';
import { createMedia } from '@/app/store/slices/mediaSlice';
import { Button } from '@/presentation/components/ui/button';
import { Progress } from '@/presentation/components/ui/progress';

interface MediaUploaderProps {
  isOpen: boolean;
  onClose: () => void;
  onSuccess: (file: File) => void;
  propertyId?: number;
  unitId?: number;
}

export const MediaUploader = ({
  isOpen,
  onClose,
  onSuccess,
  propertyId,
  unitId,
}: MediaUploaderProps) => {
  const dispatch = useAppDispatch();
  const [files, setFiles] = useState<File[]>([]);
  const [isUploading, setIsUploading] = useState(false);
  const [uploadProgress, setUploadProgress] = useState(0);
  const [error, setError] = useState<string | null>(null);

  const { getRootProps, getInputProps, isDragActive } = useDropzone({
    onDrop: (acceptedFiles) => {
      setFiles(prev => [...prev, ...acceptedFiles]);
    },
    accept: {
      'image/*': ['.jpg', '.jpeg', '.png', '.gif', '.webp'],
      'video/*': ['.mp4', '.webm', '.mov'],
      'application/pdf': ['.pdf'],
    },
    maxSize: 10 * 1024 * 1024, // 10MB
    multiple: true,
  });

  const handleFileUpload = async () => {
    if (files.length === 0) return;

    setIsUploading(true);
    setError(null);
    setUploadProgress(0);

    try {
      const formData = new FormData();
      files.forEach((file) => {
        formData.append('files[]', file);
      });

      if (propertyId) {
        formData.append('property_id', propertyId.toString());
      } else if (unitId) {
        formData.append('unit_id', unitId.toString());
      }

      await dispatch(createMedia(formData));
      setUploadProgress(100);

      if (files.length > 0) {
        onSuccess(files[0]);
      }

      setTimeout(() => {
        setFiles([]);
        setIsUploading(false);
        onClose();
      }, 500);
    } catch (err) {
      setError('Error al subir los archivos');
      setIsUploading(false);
    }
  };

  if (!isOpen) return null;

  return (
    <div className="fixed inset-0 z-50 flex items-center justify-center bg-black/50 p-4">
      <div className="w-full max-w-2xl rounded-lg bg-white shadow-xl">
        <div className="p-6">
          <div {...getRootProps()} className="border-2 border-dashed rounded-lg p-8 text-center cursor-pointer">
            <input {...getInputProps()} disabled={isUploading} />
            <p>Arrastra archivos aquí o haz clic para seleccionar</p>
          </div>

          {files.length > 0 && (
            <div className="mt-6 space-y-2">
              {files.map((file, index) => (
                <div key={index} className="flex items-center justify-between border p-3 rounded">
                  <span>{file.name}</span>
                  <button onClick={() => setFiles(prev => prev.filter((_, i) => i !== index))}>
                    Eliminar
                  </button>
                </div>
              ))}
            </div>
          )}

          {isUploading && (
            <div className="mt-6">
              <Progress value={uploadProgress} />
            </div>
          )}
        </div>

        <div className="flex justify-end space-x-3 border-t p-4">
          <Button variant="outline" onClick={onClose} disabled={isUploading}>
            Cancelar
          </Button>
          <Button onClick={handleFileUpload} disabled={files.length === 0 || isUploading}>
            {isUploading ? 'Subiendo...' : 'Subir archivos'}
          </Button>
        </div>
      </div>
    </div>
  );
};
```

## Workflows

### Configure Redux slice for media

```typescript
// src/app/store/slices/mediaSlice.ts
export const createMedia = createAsyncThunk(
  'media/create',
  async (formData: FormData, { rejectWithValue }) => {
    try {
      const useCase = container.get<CreateMediaUseCase>(TYPES.CreateMediaUseCase);
      return await useCase.execute(formData);
    } catch (error: any) {
      return rejectWithValue(error.response?.data?.message || 'Error al subir');
    }
  }
);
```

### Configure file types and size limits

```typescript
const { getRootProps, getInputProps } = useDropzone({
  accept: {
    'image/*': ['.jpg', '.jpeg', '.png', '.gif', '.webp'],
    'video/*': ['.mp4', '.webm', '.mov'],
    'audio/*': ['.mp3', '.wav', '.ogg'],
    'application/pdf': ['.pdf'],
    'application/msword': ['.doc'],
    'application/vnd.openxmlformats-officedocument.wordprocessingml.document': ['.docx'],
    'application/vnd.ms-excel': ['.xls'],
    'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet': ['.xlsx'],
  },
  maxSize: 10 * 1024 * 1024, // 10MB
  multiple: true,
  onDropRejected: (fileRejections) => {
    const errors = fileRejections.flatMap(({ errors }) => 
      errors.map(error => {
        if (error.code === 'file-too-large') {
          return 'El archivo es demasiado grande. Tamaño máximo: 10MB';
        } else if (error.code === 'file-invalid-type') {
          return 'Tipo de archivo no soportado';
        }
        return error.message;
      })
    );
    setError(errors.join(', '));
  },
});
```

### Handle file removal

```typescript
const removeFile = (index: number) => {
  setFiles(prev => prev.filter((_, i) => i !== index));
};
```

### Format file size for display

```typescript
const formatFileSize = (bytes: number): string => {
  if (bytes === 0) return '0 Bytes';
  const k = 1024;
  const sizes = ['Bytes', 'KB', 'MB', 'GB'];
  const i = Math.floor(Math.log(bytes) / Math.log(k));
  return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
};
```

### Display file icon based on type

```typescript
const getFileIcon = (file: File) => {
  if (file.type.startsWith('image/')) {
    return <ImageIcon className="h-5 w-5 text-blue-500" />;
  } else if (file.type.startsWith('video/')) {
    return <Video className="h-5 w-5 text-red-500" />;
  } else if (file.type.startsWith('audio/')) {
    return <Music className="h-5 w-5 text-purple-500" />;
  } else {
    return <FileIcon className="h-5 w-5 text-gray-500" />;
  }
};
```

### Upload with progress simulation

```typescript
const handleFileUpload = async () => {
  setIsUploading(true);
  setUploadProgress(0);

  const progressInterval = setInterval(() => {
    setUploadProgress((prev) => {
      const newProgress = prev + 10;
      if (newProgress >= 90) clearInterval(progressInterval);
      return newProgress;
    });
  }, 300);

  try {
    await dispatch(createMedia(formData));
    clearInterval(progressInterval);
    setUploadProgress(100);
  } catch (err) {
    clearInterval(progressInterval);
    setError('Error al subir');
  }
};
```

### Associate upload with entity

```typescript
const formData = new FormData();
files.forEach((file) => {
  formData.append('files[]', file);
});

if (propertyId) {
  formData.append('property_id', propertyId.toString());
} else if (unitId) {
  formData.append('unit_id', unitId.toString());
}
```

### Use in component

```typescript
import { MediaUploader } from '@/presentation/components/media/MediaUploader';

export const PropertyFormPage = () => {
  const [isUploaderOpen, setIsUploaderOpen] = useState(false);

  const handleUploadSuccess = (file: File) => {
    toast({ title: "¡Éxito!", description: "Archivo subido correctamente" });
  };

  return (
    <div>
      <Button onClick={() => setIsUploaderOpen(true)}>Subir archivo</Button>
      
      <MediaUploader
        isOpen={isUploaderOpen}
        onClose={() => setIsUploaderOpen(false)}
        onSuccess={handleUploadSuccess}
        propertyId={propertyId}
      />
    </div>
  );
};
```

## File Validation

### Validate file type

```typescript
const acceptedTypes = ['image/jpeg', 'image/png', 'application/pdf'];
if (!acceptedTypes.includes(file.type)) {
  setError('Tipo de archivo no soportado');
  return;
}
```

### Validate file size

```typescript
const maxSize = 10 * 1024 * 1024; // 10MB
if (file.size > maxSize) {
  setError('El archivo es demasiado grande');
  return;
}
```

### Validate multiple files

```typescript
const maxFiles = 5;
if (files.length >= maxFiles) {
  setError('Máximo 5 archivos permitidos');
  return;
}
```

## Error Handling

### Display upload errors

```typescript
{error && (
  <Alert variant="destructive" className="mb-4">
    <AlertCircle className="h-4 w-4" />
    <AlertTitle>Error</AlertTitle>
    <AlertDescription>{error}</AlertDescription>
  </Alert>
)}
```

### Handle API errors

```typescript
try {
  await dispatch(createMedia(formData));
} catch (err) {
  setError('Error al subir los archivos. Por favor, inténtalo de nuevo.');
  setIsUploading(false);
}
```

## Best Practices

### Always disable upload during processing

```typescript
<input {...getInputProps()} disabled={isUploading} />
<Button disabled={files.length === 0 || isUploading}>Subir</Button>
```

### Show file preview for images

```typescript
{file.type.startsWith('image/') && (
  <img src={URL.createObjectURL(file)} alt={file.name} className="w-16 h-16 object-cover" />
)}
```

### Clean up object URLs

```typescript
useEffect(() => {
  return () => {
    files.forEach(file => URL.revokeObjectURL(URL.createObjectURL(file)));
  };
}, [files]);
```

### Limit file count

```typescript
const MAX_FILES = 5;
if (files.length >= MAX_FILES) {
  setError(`Máximo ${MAX_FILES} archivos permitidos`);
  return;
}
```
