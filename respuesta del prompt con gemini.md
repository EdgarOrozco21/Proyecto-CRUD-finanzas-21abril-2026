¡Excelente iniciativa! Como arquitecto de software, he diseñado un plan de trabajo que integra el desarrollo de **Flutter** con la metodología de orquestación de agentes de **Antigravity**. 

Este enfoque no solo construye el CRUD, sino que enseña a los estudiantes cómo delegar tareas a agentes especializados dentro de un flujo de trabajo automatizado.

---

## 1. Metodología de Trabajo: Flujo de Agentes (Antigravity)

Para que el proyecto sea funcional bajo este paradigma, definiremos los siguientes roles y flujos.

### Estructura de Agentes y Roles
| Agente | Rol | Skill (Habilidad) |
| :--- | :--- | :--- |
| **Architect Agent** | Definir estructura | Generación de `pubspec.yaml` y carpetas. |
| **Firebase Agent** | Configurar Backend | Enlace con Firestore y Auth. |
| **Data Agent** | Modelado | Creación de clases `Gasto`, `Ingreso` y `Usuario`. |
| **Logic Agent** | Implementar CRUD | Métodos `add`, `get`, `update`, `delete`. |

### Flujo de Trabajo
1.  **Inicio:** El usuario solicita la creación de la carpeta `crudfinanzas`.
2.  **Configuración:** El Architect Agent genera la estructura de archivos.
3.  **Vinculación:** El Firebase Agent asegura que el `google-services.json` esté en su lugar.
4.  **Generación:** El Data Agent construye los modelos Dart.
5.  **Despliegue:** El Logic Agent conecta la UI con Firestore.

---

## 2. Configuración Inicial y Dependencias

### Creación del Proyecto
```bash
flutter create crudfinanzas
cd crudfinanzas
```

### Archivo `pubspec.yaml`
Añade estas líneas bajo la sección `dependencies`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^2.27.0
  cloud_firestore: ^4.15.8
  firebase_auth: ^4.17.8
```
*Ejecuta `flutter pub get` después de guardar.*

---

## 3. Estructura de Carpetas Sugerida
```text
lib/
├── agents/            # Lógica de Antigravity (opcional para simulación)
├── models/            # gasto.dart, ingreso.dart, usuario.dart
├── services/          # firebase_service.dart (CRUD)
├── screens/           # UI (Login, Dashboard, Formularios)
└── main.dart          # Punto de entrada e inicialización de Firebase
```

---

## 4. Modelado de Datos (Data Agent)

### Archivo: `lib/models/gasto.dart`
```dart
class Gasto {
  String? id;
  double monto;
  String categoria;
  String descripcion;
  DateTime fecha;

  Gasto({this.id, required this.monto, required this.categoria, required this.descripcion, required this.fecha});

  Map<String, dynamic> toMap() {
    return {
      'monto': monto,
      'categoria': categoria,
      'descripcion': descripcion,
      'fecha': fecha.toIso8601String(),
    };
  }

  factory Gasto.fromMap(String id, Map<String, dynamic> map) {
    return Gasto(
      id: id,
      monto: map['monto']?.toDouble() ?? 0.0,
      categoria: map['categoria'] ?? '',
      descripcion: map['descripcion'] ?? '',
      fecha: DateTime.parse(map['fecha']),
    );
  }
}
```

### Archivo: `lib/models/usuario.dart`
```dart
class Usuario {
  String uid;
  String nombre;
  String correo;
  DateTime fechaRegistro;

  Usuario({required this.uid, required this.nombre, required this.correo, required this.fechaRegistro});

  Map<String, dynamic> toMap() {
    return {
      'nombre': nombre,
      'correo': correo,
      'fecha_registro': fechaRegistro.toIso8601String(),
    };
  }
}
```

---

## 5. Implementación del CRUD (Logic Agent)

Crearemos un servicio centralizado para manejar la relación Usuario -> Finanzas.

### Archivo: `lib/services/firebase_service.dart`
```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import '../models/gasto.dart';

class FirebaseService {
  final FirebaseFirestore _db = FirebaseFirestore.instance;

  // CREATE: Agregar gasto a un usuario específico
  Future<void> addGasto(String userId, Gasto gasto) async {
    await _db.collection('usuarios').doc(userId).collection('gastos').add(gasto.toMap());
  }

  // READ: Obtener stream de gastos
  Stream<List<Gasto>> getGastos(String userId) {
    return _db
        .collection('usuarios')
        .doc(userId)
        .collection('gastos')
        .snapshots()
        .map((snapshot) => snapshot.docs
            .map((doc) => Gasto.fromMap(doc.id, doc.data()))
            .toList());
  }

  // UPDATE
  Future<void> updateGasto(String userId, Gasto gasto) async {
    await _db
        .collection('usuarios')
        .doc(userId)
        .collection('gastos')
        .doc(gasto.id)
        .update(gasto.toMap());
  }

  // DELETE
  Future<void> deleteGasto(String userId, String gastoId) async {
    await _db
        .collection('usuarios')
        .doc(userId)
        .collection('gastos')
        .doc(gastoId)
        .delete();
  }
}
```

---

## 6. Configuración en Consola de Firebase

1.  **Crear Proyecto:** Ve a [Firebase Console](https://console.firebase.google.com/) y crea "CRUD Finanzas".
2.  **Firestore:** Crea una base de datos en **Modo Prueba**. Crea la colección principal `usuarios`.
3.  **Authentication:** Habilita el método "Correo electrónico/contraseña".
4.  **Flutterfire CLI:** (Recomendado) Ejecuta `flutterfire configure` para vincular automáticamente las credenciales.

---

## Práctica Guiada para Estudiantes

> **Paso 1:** Configuren su entorno Antigravity para que el **Agente de Modelado** genere automáticamente la clase `ingreso.dart` siguiendo el patrón de `gasto.dart`.
>
> **Paso 2:** En la consola de Firebase, observen cómo al crear un usuario en Auth, deben disparar una función (o un método en Flutter) que cree el documento correspondiente en la colección `usuarios` con su `uid`.
>
> **Paso 3:** Implementen un `ListView.builder` que consuma el `Stream` del `Logic Agent` para mostrar los datos en tiempo real.

Este flujo asegura que el código sea modular, escalable y fácil de mantener, aplicando principios de arquitectura limpia.
