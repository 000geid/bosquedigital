
¿Alguna vez abriste un archivo CSV en VS Code y pensaste que podría verse mejor? ¡A mí me pasó! Por eso creé **CSViz**, una extensión que transforma los CSV aburridos en tablas bonitas y personalizables, directamente en tu editor.
  

En este post te cuento cómo la construí, los retos que encontré y cómo puedes crear tu propio editor personalizado para VS Code.


---

## ¿Por qué hacer un visor de CSV?

Los archivos CSV están en todos lados, pero verlos como texto plano no es nada amigable. Yo quería:

- Una vista en tabla para los CSV
- Controles para cambiar tamaño y color de fuente, y el color de fondo del encabezado
- Que los estilos se guarden por archivo

---

## Estructurando la extensión

Usé el generador de extensiones de VS Code con Yeoman:

```bash
yo code
```


Elegí:
- **TypeScript** para mayor seguridad
- La API de VS Code **Custom Editor** para archivos `.csv`

  

---

  

## Registrando el editor personalizado

En `package.json` registré el editor para archivos CSV:

```json

"contributes": {

"customEditors": [

{

"viewType": "csv-viz.csvEditor",

"displayName": "CSViz",

"selector": [{ "filenamePattern": "*.csv" }],

"priority": "default"

}

]

}

```

  

---
## Implementando el Webview

El corazón de la extensión es un webview que muestra el CSV como tabla. En `src/extension.ts` implementé un `CustomTextEditorProvider`:

  
```ts

class CsvEditorProvider implements vscode.CustomTextEditorProvider {

// ...

public async resolveCustomTextEditor(document, webviewPanel, _token) {

webviewPanel.webview.options = { enableScripts: true };

webviewPanel.webview.html = this.getHtmlForWebview(webviewPanel.webview);

// Enviar datos CSV al webview

this.updateWebview(document, webviewPanel);

// Escuchar cambios y mensajes...

}

}

```

  

---

  

## Construyendo la interfaz


La UI está hecha con HTML, CSS y JavaScript en la carpeta `media/`. Incluye:

  

- Una tabla que parsea y muestra el CSV

- Controles para tamaño de fuente, color de texto y color de fondo del encabezado

- Actualización en tiempo real al cambiar los controles

  

```html

<div id="controls">

<label for="font-size">Tamaño de fuente:</label>

<input type="number" id="font-size" value="16" min="8" max="72">

<label for="font-color">Color de fuente:</label>

<input type="color" id="font-color" value="#000000">

<label for="header-bg-color">Fondo de encabezado:</label>

<input type="color" id="header-bg-color" value="#f2f2f2">

</div>

```

  

---

  

## Persistencia de estado

  

Para que la experiencia sea fluida, usé `workspaceState` de VS Code para guardar los estilos por archivo. El webview envía mensajes a la extensión cuando cambian los estilos, y la extensión los guarda:

  

```ts

webviewPanel.webview.onDidReceiveMessage(e => {

if (e.type === 'saveState') {

this.context.workspaceState.update(document.uri.toString(), e.data);

}

});

```

  

---

  

## Pruebas

  

Escribí una prueba de integración con Mocha que:

- Crea un archivo CSV de prueba

- Lo abre con el editor personalizado

- Verifica que el editor activo sea el de CSViz

  

```ts

test('Debe abrir el archivo .csv con el editor personalizado CSViz', async () => {

// ...setup...

await vscode.commands.executeCommand('vscode.openWith', uri, 'csv-viz.csvEditor');

// ...asserts...

});

```

  

---

  

## Empaquetado y publicación

  

Antes de publicar:

- Añadí una licencia MIT

- Actualicé `.vscodeignore` para excluir archivos de desarrollo y datos de ejemplo

- Añadí el campo `repository` en `package.json`

- Subí el código a [GitHub](https://github.com/000geid/csviz)

  
Antes de publicar la extensión hay que crearse una cuenta en Azure Cloud (sí, me olvido que Github pertenece a Microsoft), gracias a lo que crearemos una organización desde la cual publicar nuestra extensión.  Sugiero leer la siguiente [guía](https://code.visualstudio.com/api/working-with-extensions/publishing-extension) que explica el procedimiento a detalle. Una vez seguidos todos los pasos y obtenidos todos los permisos necesarios, publicar fue tan simple como:

  

```bash
vsce publish
```

  

---

  

## Lecciones aprendidas


  

- **Webviews son poderosos**: ¡Puedes construir casi cualquier UI!

- **Probar editores personalizados**: Las pruebas de integración dan mucha confianza.

- **Pulido para el Marketplace**: Un buen README, licencia y archivos ignore importan mucho.

  

---

  

## ¡Pruébalo!

  

Puedes ver el código y las instrucciones de instalación en [GitHub](https://github.com/000geid/csviz).

  

---

  

**¡Gracias por leer!** Si tienes dudas o quieres crear tu propia extensión para VS Code, ¡escríbeme o haz un fork del repo!

#tecnología #vscode #plugin