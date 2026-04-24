Esta es una guía estructurada para configurar tu entorno de agentes y desarrollar la aplicación **ProyectoParque**. Usaremos una arquitectura limpia y la potencia de Firebase para el backend.

---

## 1. Estructura de la Habilidad Agente Global: `.agents`

Primero, creamos la estructura de archivos para tu sistema de agentes. Esta carpeta centraliza la lógica de automatización.

```text
.agents/
├── SKILL.md              # Definición de capacidades del agente
├── scripts/              # Automatizaciones (bash/python/dart)
├── ejemplos/             # Plantillas de código UI y lógica
└── resources/            # Activos, iconos y esquemas de datos
```

### Contenido de `.agents/SKILL.md`
> **Habilidad:** Automatización de Desarrollo Flutter + Firebase
> **Capacidades:**
> * Generación de CRUDs mediante Scraping de requerimientos.
> * Diseño de UI adaptativa para Parques de Atracciones.
> * Despliegue y configuración automática de Firebase.

---

## 2. Preparación del Entorno y Requisitos

### Verificación de Herramientas
Abre tu terminal en **VS Code** o **Antigravity** y ejecuta:

1.  **Flutter:** `flutter --version` (Si no lo tienes, descárgalo en flutter.dev).
2.  **Firebase CLI:** `firebase --version`.
    * *Si no está instalado:* `npm install -g firebase-tools`.
3.  **FlutterFire CLI:** `dart pub global activate flutterfire_cli`.

### Autenticación y Proyecto
```bash
# Iniciar sesión
firebase login

# Configurar FlutterFire (Esto vincula tu app con la consola de Firebase)
flutterfire configure
```

---

## 3. Creación del Proyecto: `proyectoparque`

Ejecuta los siguientes comandos para inicializar la estructura:

```bash
flutter create proyectoparque
cd proyectoparque
```

### Configuración de `pubspec.yaml`
Añade estas dependencias necesarias para el CRUD y Firebase:

```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^3.0.0
  cloud_firestore: ^5.0.0
  cupertino_icons: ^1.0.6
```
*Corre `flutter pub get` tras guardar.*

---

## 4. Desarrollo del Proyecto (Código Fuente)

### Estructura de Carpetas Sugerida:
```text
lib/
├── main.dart
├── models/
│   └── atraccion_model.dart
├── services/
│   └── firebase_service.dart
└── screens/
    ├── home_screen.dart
    └── crud_screen.dart
```

### A. Modelo de Datos (`models/atraccion_model.dart`)
```dart
class Atraccion {
  String id;
  String nombre;
  String descripcion;
  bool operativa;

  Atraccion({required this.id, required this.nombre, required this.descripcion, required this.operativa});

  Map<String, dynamic> toMap() {
    return {'nombre': nombre, 'descripcion': descripcion, 'operativa': operativa};
  }
}
```

### B. Servicio Firestore (`services/firebase_service.dart`)
Este archivo maneja la lógica de los agentes para el Scraping y almacenamiento de datos.

```dart
import 'package:cloud_firestore/cloud_firestore.dart';

FirebaseFirestore db = FirebaseFirestore.instance;

// CREATE
Future<void> addAtraccion(String nombre, String desc) async {
  await db.collection('atracciones').add({'nombre': nombre, 'descripcion': desc, 'operativa': true});
}

// READ (Stream para cambios en tiempo real)
Stream<QuerySnapshot> getAtracciones() {
  return db.collection('atracciones').snapshots();
}

// UPDATE
Future<void> updateAtraccion(String id, String nuevoNombre) async {
  await db.collection('atracciones').doc(id).update({'nombre': nuevoNombre});
}

// DELETE
Future<void> deleteAtraccion(String id) async {
  await db.collection('atracciones').doc(id).delete();
}
```

### C. Pantalla Principal y CRUD UI (`screens/home_screen.dart`)


```dart
import 'package:flutter/material.dart';
import '../services/firebase_service.dart';

class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      app_appBar: AppBar(title: Text('Parque de Atracciones - Admin')),
      body: StreamBuilder(
        stream: getAtracciones(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return Center(child: CircularProgressIndicator());
          var docs = snapshot.data!.docs;
          return ListView.builder(
            itemCount: docs.length,
            itemBuilder: (context, index) {
              return ListTile(
                title: Text(docs[index]['nombre']),
                subtitle: Text(docs[index]['descripcion']),
                trailing: IconButton(
                  icon: Icon(Icons.delete, color: Colors.red),
                  onPressed: () => deleteAtraccion(docs[index].id),
                ),
                onTap: () {
                  // Aquí podrías implementar el Update con un Dialog
                },
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        child: Icon(Icons.add),
        onPressed: () => _mostrarDialogoAgregar(context),
      ),
    );
  }

  void _mostrarDialogoAgregar(BuildContext context) {
    // Implementar controller y showDialog para capturar nombre y descripción
  }
}
```

---

## 5. Pruebas de Verificación
1.  **Ejecución:** Conecta un dispositivo o emulador y presiona `F5` en VS Code.
2.  **Consola Firebase:** Ve a la pestaña de **Firestore Database** y verifica que al pulsar el botón `+` en la app, se cree el documento en la nube de forma instantánea.
3.  **Logs:** Revisa la terminal de depuración para confirmar que no hay errores de permisos (asegúrate de que las reglas de Firestore permitan lectura/escritura durante el desarrollo).

¿Deseas que profundice en el script de scraping para automatizar la carga de nombres de atracciones reales?
