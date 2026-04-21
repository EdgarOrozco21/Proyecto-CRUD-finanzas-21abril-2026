¡Qué tal, Edgar! Como arquitecto de software, me parece un proyecto excelente y muy ambicioso. Construir un sistema como este es el paso ideal para llevar un gestor financiero, digamos algo de la escala de un "FinanTrack", al siguiente nivel. 

Para que esta práctica guiada sea un éxito y destaque en tu grupo 6-J, vamos a estructurarla utilizando la metodología de agentes (Antigravity). Este enfoque simula un equipo de desarrollo impulsado por IA, dividiendo el trabajo en roles y habilidades específicas.

Aquí tienes el plan de trabajo completo y la arquitectura del sistema.

---

### Fase 1: Metodología Antigravity (Agentes y Flujo de Trabajo)

Para construir software complejo, dividimos el proyecto asignando "Agentes" con roles y *skills* (habilidades) específicos. Si estás guiando a otros estudiantes, puedes asignarles estos roles.

1.  **Agente Arquitecto de Datos (DBA Agent)**
    * **Rol:** Diseñar la base de datos NoSQL y la lógica de relaciones.
    * **Skills:** Modelado en Firestore, estructuración de colecciones/subcolecciones.
    * **Flujo:** Define que un `usuario` tendrá subcolecciones independientes para `gastos` e `ingresos` para escalar sin problemas.
2.  **Agente Backend y Seguridad (DevOps Agent)**
    * **Rol:** Configurar la consola de Firebase y conectar el proyecto Flutter.
    * **Skills:** Configuración de `firebase_core`, reglas de seguridad de Firestore.
    * **Flujo:** Implementa las dependencias en `pubspec.yaml` y asegura la conexión.
3.  **Agente Frontend (Flutter UI/UX Agent)**
    * **Rol:** Construir la interfaz de usuario y conectar la lógica CRUD.
    * **Skills:** Programación en Dart, manejo de estado, diseño de Widgets.
    * **Flujo:** Crea los formularios para ingresar datos y las listas para leerlos.

---

### Fase 2: Arquitectura y Estructura de Carpetas

Ejecuta en tu terminal el comando `flutter create crudfinanzas`. Una vez creado, abre el proyecto y organiza tu carpeta `lib` de esta manera para mantener el código modular:

```text
crudfinanzas/
├── lib/
│   ├── models/
│   │   ├── usuario_model.dart
│   │   ├── transaccion_model.dart
│   ├── screens/
│   │   ├── home_screen.dart
│   │   ├── crud_screen.dart
│   ├── services/
│   │   ├── firebase_service.dart
│   ├── main.dart
├── pubspec.yaml
```

---

### Fase 3: Integración de Librerías y Firebase

**1. Actualizar `pubspec.yaml`**
El *DevOps Agent* debe agregar las librerías necesarias. Abre tu archivo `pubspec.yaml` y bajo `dependencies`, agrega:

```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^2.32.0 # Inicializa Firebase
  cloud_firestore: ^4.17.5 # Base de datos
  # Nota: Para un proyecto real, también usaríamos firebase_auth
```

**2. Configurar Consola de Firebase**
1. Ve a [Firebase Console](https://console.firebase.google.com/) y haz clic en "Agregar proyecto" (nómbralo `crudfinanzas`).
2. Ve a "Compilación" > "Firestore Database" y haz clic en "Crear base de datos" (comienza en modo de prueba para la práctica).
3. Registra tu app Flutter usando el CLI de Firebase (`flutterfire configure`).

*Nota de Seguridad:* Mencionaste guardar la "contraseña" en la colección de usuarios. En el mundo real del software, **nunca guardamos contraseñas en bases de datos legibles** como Firestore. Eso lo maneja Firebase Authentication de forma encriptada. En Firestore solo guardaremos los metadatos (nombre, correo, fecha).

---

### Fase 4: Implementación del Código (Frontend Agent)

Aquí tienes el código funcional para el servicio CRUD. La mejor forma de relacionar los datos en Firestore es crear una colección raíz llamada `usuarios`, y dentro de cada documento de usuario, crear subcolecciones para sus finanzas.

**1. El Servicio CRUD (`lib/services/firebase_service.dart`)**

Este archivo maneja toda la lógica de Crear, Leer, Actualizar y Borrar.

```dart
import 'package:cloud_firestore/cloud_firestore.dart';

class FirebaseService {
  final FirebaseFirestore _db = FirebaseFirestore.instance;

  // 1. CREAR USUARIO (Metadatos)
  Future<void> crearUsuario(String uid, String nombre, String correo) async {
    await _db.collection('usuarios').doc(uid).set({
      'nombre': nombre,
      'correo': correo,
      'fecha_registro': FieldValue.serverTimestamp(),
    });
  }

  // 2. CREAR GASTO (Relacionado al UID del usuario)
  Future<void> agregarGasto(String uid, double monto, String categoria, String descripcion) async {
    await _db.collection('usuarios').doc(uid).collection('gastos').add({
      'monto': monto,
      'categoria': categoria,
      'descripcion': descripcion,
      'fecha': FieldValue.serverTimestamp(),
      'tipo': 'gasto'
    });
  }

  // 3. CREAR INGRESO
  Future<void> agregarIngreso(String uid, double monto, String fuente) async {
    await _db.collection('usuarios').doc(uid).collection('ingresos').add({
      'monto': monto,
      'fuente': fuente,
      'fecha': FieldValue.serverTimestamp(),
      'tipo': 'ingreso'
    });
  }

  // 4. LEER GASTOS (Stream para actualización en tiempo real)
  Stream<QuerySnapshot> obtenerGastos(String uid) {
    return _db.collection('usuarios').doc(uid).collection('gastos')
        .orderBy('fecha', descending: true).snapshots();
  }

  // 5. ACTUALIZAR GASTO
  Future<void> actualizarGasto(String uid, String gastoId, double nuevoMonto, String nuevaCategoria) async {
    await _db.collection('usuarios').doc(uid).collection('gastos').doc(gastoId).update({
      'monto': nuevoMonto,
      'categoria': nuevaCategoria,
    });
  }

  // 6. BORRAR TRANSACCIÓN (Funciona para ingresos o gastos)
  Future<void> borrarDocumento(String uid, String coleccion, String docId) async {
    await _db.collection('usuarios').doc(uid).collection(coleccion).doc(docId).delete();
  }
}
```

**2. Punto de Entrada (`lib/main.dart`)**

Debemos inicializar Firebase antes de arrancar la aplicación de Flutter.

```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'screens/home_screen.dart'; // Tu pantalla principal

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(); // Inicia la magia de Firebase
  runApp(const CrudFinanzasApp());
}

class CrudFinanzasApp extends StatelessWidget {
  const CrudFinanzasApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'CRUD Finanzas',
      theme: ThemeData(
        brightness: Brightness.dark, // Un toque estético oscuro
        primarySwatch: Colors.blueGrey,
      ),
      home: const HomeScreen(), // Aquí implementarás la UI que llama a FirebaseService
    );
  }
}
```

Con esta arquitectura y el servicio de Firestore listo, tienes el backend y la lógica de conexión completamente resueltos. Los "Agentes" han entregado la infraestructura.

Para completar la práctica guiada con tus compañeros, ¿te gustaría que escribamos el código exacto de la interfaz gráfica (`home_screen.dart`) donde se muestran las listas y el botón flotante para agregar los gastos e ingresos?
