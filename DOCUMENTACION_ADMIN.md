# Documentación del Sistema de Administración

## Resumen
Este documento describe el sistema de administración implementado en la aplicación de taxímetro GPS. El sistema permite identificar usuarios administradores y mostrarles funcionalidades ocultas no disponibles para usuarios regulares.

## Sistema de Roles

### Usuario Admin
- **Email identificador:** `trabajoonline88@gmail.com`
- Cuando un usuario con este correo inicia sesión, automáticamente se le asigna el rol de administrador
- El sistema verifica el email del usuario autenticado en AuthWrapper y pasa esta información a App como prop

### Verificación de Admin
La verificación se realiza en dos lugares:

#### 1. AuthWrapper.tsx (línea 250-255)
```typescript
{React.isValidElement(children) && typeof children.type !== 'string'
  ? React.cloneElement(children as React.ReactElement<any>, { isAdmin: user?.email === 'trabajoonline88@gmail.com' })
  : children
}
```
- Verifica si el email del usuario coincide con `trabajoonline88@gmail.com`
- Usa `React.cloneElement` para pasar la prop `isAdmin` al componente App
- Esta verificación ocurre SOLO después que el usuario se autentica

#### 2. App.tsx (línea 248)
```typescript
function App({ isAdmin = false }: { isAdmin?: boolean }) {
```
- Recibe el prop `isAdmin` desde AuthWrapper
- Valor por defecto es `false` si no se proporciona
- Solo usa este prop para mostrar/ocultar funcionalidades admin

## Sección de Funciones Ocultas

### Ubicación en la Interfaz
- **Botón flotante** en la esquina inferior izquierda (debajo del botón de ganancias mensuales)
- **Visible solo para admin** - usuarios regulares no lo ven
- **Diseño distinguido:**
  - Fondo gris oscuro (`bg-gray-700`)
  - Borde rojo (`border-red-500`)
  - Ícono Shield (escudo) de lucide-react
  - Hover effect: `hover:bg-gray-600`

### Panel Admin (Modal)
Al hacer clic en el botón admin, se abre un modal con las siguientes funciones:

#### 1. Ver Estadísticas
- Estado de desarrollo
- Placeholder para: Estadísticas generales de usuarios y viajes

#### 2. Ver Todos los Viajes
- Estado de desarrollo
- Placeholder para: Historial completo de viajes de todos los conductores

#### 3. Configuración Avanzada
- Estado de desarrollo
- Placeholder para: Tarifas, zonas especiales, parámetros del sistema

## Estructura de Código

### Estados Necesarios (App.tsx línea 312)
```typescript
const [showAdminModal, setShowAdminModal] = useState(false);
```

### Botón Flotante Admin (App.tsx línea 860-869)
```typescript
{isAdmin && (
  <button
    onClick={() => setShowAdminModal(true)}
    className="fixed bottom-42 right-6 w-16 h-16 bg-gray-700 hover:bg-gray-600..."
    title="Panel de Administración"
  >
    <Shield className="w-8 h-8" />
  </button>
)}
```

### Modal Admin (App.tsx línea 1033-1082)
- Renderizado condicional: `{showAdminModal && isAdmin && (...)`
- Contiene 3 botones de funcionalidades
- Cada botón muestra un alert indicando que es funcionalidad en desarrollo

## Importaciones Requeridas

### En App.tsx
```typescript
// Shield agregado a los imports de lucide-react
import { Shield } from 'lucide-react';
```

## Seguridad

### Consideraciones Importantes
1. **Verificación en Frontend**:
   - Actualmente la verificación se hace en el frontend
   - Para producción, se RECOMIENDA implementar Custom Claims en Firebase Auth

2. **Custom Claims (Recomendado para Producción)**:
   ```javascript
   // En Cloud Functions o Admin SDK
   admin.auth().setCustomUserClaims(uid, { admin: true });
   ```

3. **Row Level Security en Firestore**:
   - Implementar reglas que verifiquen el rol de admin
   - Proteger colecciones sensibles

### No se Modifica la Lógica Existente
- El sistema admin es completamente aislado
- No afecta la funcionalidad de taxímetro
- Usuarios regulares no ven ningún cambio

## Expansión Futura

### Para Agregar Nuevas Funciones Admin

1. **Agregar botón en el modal admin:**
   ```typescript
   <button
     onClick={() => alert('Nueva funcionalidad')}
     className="w-full bg-[color]-600 hover:bg-[color]-500..."
   >
     Nombre de Función
   </button>
   ```

2. **Crear estado si es necesario:**
   ```typescript
   const [showNewAdminFeature, setShowNewAdminFeature] = useState(false);
   ```

3. **Crear modal para la nueva funcionalidad:**
   ```typescript
   {showNewAdminFeature && isAdmin && (
     <div className="fixed inset-0 bg-black bg-opacity-80...">
       {/* Contenido del modal */}
     </div>
   )}
   ```

### Agregar Múltiples Administradores

#### Opción A - Array de Emails (Simple)
```typescript
// En AuthWrapper.tsx
const ADMIN_EMAILS = [
  'trabajoonline88@gmail.com',
  'otro-admin@gmail.com'
];
const isAdmin = user?.email && ADMIN_EMAILS.includes(user.email);
```

#### Opción B - Firestore Collection (Recomendado)
```typescript
// Crear colección 'admins' en Firestore
// Verificar si el UID existe en esa colección
const isAdmin = await checkIfUserIsAdmin(user.uid);
```

#### Opción C - Custom Claims Firebase (Más Seguro)
```typescript
// En Cloud Functions
admin.auth().setCustomUserClaims(uid, { admin: true });

// En App
const isAdmin = (await user.getIdTokenResult()).claims.admin === true;
```

## Base de Datos

### Colecciones Existentes
- `usuarios`: Información básica de usuarios
- `gananciasDiarias`: Ganancias diarias por usuario
- `bitacora_ganancias`: Historial mensual de ganancias

### Colecciones Sugeridas para Admin
- `admin_logs`: Registro de acciones administrativas
- `admin_config`: Configuración global del sistema
- `tarifas_config`: Tarifas dinámicas
- `zonas_especiales`: Gestión de zonas

## Mantenimiento

### Cambiar Email de Admin
1. Ir a `AuthWrapper.tsx` línea 251
2. Cambiar: `user?.email === 'trabajoonline88@gmail.com'`
3. Por el nuevo email

### Cambiar Color/Estilo del Botón Admin
- Botón: `App.tsx` línea 864-865
- Modal: `App.tsx` línea 1036
- Modificar clases de Tailwind según necesario

### Deshabilitar Panel Admin Temporalmente
```typescript
// En AuthWrapper.tsx línea 251
const isAdmin = false; // Comentar la verificación normal
```

## Archivos Modificados

### 1. AuthWrapper.tsx
- **Línea 251**: Agregada verificación de email para admin
- **Línea 250-255**: Uso de React.cloneElement para pasar prop isAdmin

### 2. App.tsx
- **Línea 25**: Agregado import de Shield
- **Línea 248**: Función App ahora recibe prop isAdmin
- **Línea 312**: Estado showAdminModal
- **Línea 860-869**: Botón flotante admin
- **Línea 1033-1082**: Modal admin con 3 funcionalidades

### 3. DOCUMENTACION_ADMIN.md (nuevo)
- Este archivo de documentación

## Notas para Futuros Programadores

### Cosas Importantes a Saber
1. **El sistema de admin es independiente** - no modifica ninguna funcionalidad existente
2. **Solo se pasa un prop booleano** - no hay state management complejo
3. **Las funcionalidades están en desarrollo** - mostrar alertas temporales
4. **No hay cambios en la lógica del taxímetro** - completamente aislado
5. **El botón solo aparece para admin** - renderizado condicional con `{isAdmin && ...}`

### Seguridad
- **NO confiar 100% en la verificación frontend** para datos sensibles
- Implementar validación en backend/Firestore RLS
- Usar Custom Claims de Firebase para producción

### Cómo Debuggear
- Abrir DevTools → Console
- El prop `isAdmin` está disponible en el componente App
- Verificar en Network que se logueó con el email correcto

### Cambios Futuros Sugeridos
1. Implementar Custom Claims en Firebase
2. Crear colección de admin_logs en Firestore
3. Agregar funcionalidades reales en los 3 botones
4. Implementar RLS en Firestore para proteger datos admin

## Rutas y Tarifas - Viviendas

### Destinos Disponibles

#### 1. Pueblito/Pared
- **Base:** $50
- **Altura Tianguis:** $60
- **Altura Centro:** $80

#### 2. Agrícola Paredes
- **Base:** $100
- **Centro:** $130

#### 3. Tacamo
- **Base:** $250
- **Centro:** $300

#### 4. Estribo
- **Base:** $100
- **Centro:** $130

#### 5. Cielo Azul / Cerritos
- **Base:** $130
- **Centro:** $150

### Implementación Técnica

**Archivo:** `App.tsx`

**Estado:** `routeBaseFare` (línea 359)
```typescript
const [routeBaseFare, setRouteBaseFare] = useState<number | null>(null);
```

**Selector de Rutas:** (línea 2470-2541)
- Modal `showRutasModal` que permite seleccionar destino y opción
- Al seleccionar se ejecuta:
  - `setRouteBaseFare(opcion.costo)` - Establece el precio base especial
  - Se cierra el modal automáticamente después de seleccionar
  - El precio se usa como tarifa base para el viaje hasta que se finalice

**Reset al Finalizar Viaje (línea 702):**
```typescript
setRouteBaseFare(null);
```
- Se ejecuta automáticamente en `stopTrip()` (línea 702) para resetear a tarifa normal
- Ocurre cuando el usuario finaliza un viaje
- Junto con otros resets: Soriana, Extras, Paradas, Tipo de Tarifa, etc.
- Garantiza que el siguiente viaje inicie con estado limpio

### Agregar Nuevas Rutas o Destinos

#### Para agregar una nueva opción a un destino existente:

1. **Ubicar el destino en App.tsx** (línea 2480-2541)
2. **Agregar objeto a la lista:**
   ```typescript
   { nombre: 'Nueva Opción', costo: XXX }
   ```
3. **Asegurar que el costo sea positivo y en formato correcto**

#### Para agregar un nuevo destino:

1. **Agregar nombre a la lista de destinos** (línea 2479-2484)
2. **Crear condicional** similar a los existentes:
   ```typescript
   ) : selectedDestino === 'Nuevo Destino' ? (
     <div className="space-y-2">
       {[
         { nombre: 'Opción 1', costo: 100 },
         { nombre: 'Opción 2', costo: 150 }
       ].map((opcion) => (
         // ... botón con setRouteBaseFare
       ))}
     </div>
   ```
3. **Actualizar esta documentación**

## Reset de Datos al Finalizar Viaje

### Comportamiento Esperado
Cada vez que el conductor finaliza un viaje (botón "Finalizar Viaje"), TODOS los datos del viaje anterior deben reiniciarse para que el siguiente viaje comience en estado limpio.

### Variables que se Reinician en `stopTrip()` (App.tsx)
- `routeBaseFare` → `null` (tarifa base de ruta especial)
- `sorianaActive` → `false` (modo Soriana)
- Extras → `0`
- Paradas → estado inicial
- Tipo de tarifa → tarifa normal

### Regla General
**Siempre que se agregue una nueva variable de estado que afecte el cálculo de tarifas o el comportamiento del viaje, se DEBE incluir su reset dentro de la función `stopTrip()` en App.tsx.**

Ejemplo de patrón correcto:
```typescript
const stopTrip = () => {
  // ... logica de guardado
  setRouteBaseFare(null);      // reset tarifa de ruta
  setSorianaActive(false);     // reset modo Soriana
  setExtras(0);                // reset extras
  // ... cualquier nuevo estado debe resetearse aqui tambien
};
```

---

## Historial de Cambios

### Versión 1.2 (2026-04-21)
- Corregido: Agricola Paredes, Tacamo y Estribo no mostraban sus opciones de tarifa en el modal de rutas (caian en el bloque else generico)
- Se agregaron los bloques condicionales faltantes en App.tsx para esos tres destinos
- Agregada seccion "Reset de Datos al Finalizar Viaje" en esta documentacion

### Versión 1.1
- Agregadas rutas para Agricola Paredes, Tacamo, Estribo (solo en lista de destinos, faltaba el renderizado)
- Agregadas opciones para Cielo Azul / Cerritos
- Reset de routeBaseFare al finalizar viaje

### Versión 1.0
- Implementacion del sistema de admin
- Verificacion por email
- Panel admin con 3 funcionalidades placeholder
- Boton flotante exclusivo para admin
- Documentacion para programadores
- Sin cambios en logica existente

---

**Estado:** Sistema en desarrollo - Funcionalidades admin son placeholders

**Próximas mejoras:** Implementar funcionalidades reales cuando sea necesario

**Contacto:** Consultar este documento para dudas o cambios

