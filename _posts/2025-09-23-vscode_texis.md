---
title: "Texis + VSCode: Plantilla para tesis en VSCode."
date: 2025-09-23 12:29:00 -0700
categories: [Latex, VSCode]
tags: [Latex, VSCode, Tesis]    # TAG names should always be lowercase
math: true
image:
  path: assets/img/lego_writing.jpg
  alt: jinja.
comments: true
---

* No tengo para el overleaf 🥀

* _desde IA-UNAM Ensenada._ ahora vivo en la obrera.

Notas: 
* 🍂 En Linux Mint 21.3 Cinnamon. No debería cambiar mucho en otras distribuciones basadas en Ubuntu.
* Guía para utilizar la plantilla [TEXIS](https://gaia.fdi.ucm.es/research/texis/) en VSCode.

# En VSCode

**instalar:** 

```bash
# Librerías
sudo apt-get install texlive-full
# VSCode
sudo dpkg -i code y damos tab
```


**En extensiones del VSCode:** 

- LaTeX Workshop
- LaTeX language support
- LaTeX Snippets
- latex-formatter

**Prueba**

creamos carpeta y dentro un archivo `prueba.tex`

```bash
\title{Documento en blanco}

\documentclass{article}
\usepackage[utf8]{inputenc}
\usepackage[spanish]{babel}

\begin{document}
(Escribir el contenido aquí.)
\end{document}

```

# **Consideraciones:**

> *En primer lugar, es necesario destacar que los ficheros `.tex` deben tener
codificación **ISO-8859-1**. Esto es lo que ocurre de manera predefinida en
Windows y en algunos Linux como Debian. Una excepción significativa es
el caso de **Ubuntu**, que usa de manera predeterminada **UTF-8**. En ese caso,
deberás ser cuidadoso para **asegurarte de que grabas tus ficheros con ISO-
8859-1.***
> 

### Configuración en `settings.json`:

1. Abre el archivo de configuración en VS Code:
    - Presiona **Ctrl+,** o ve a **File > Preferences > Settings**.
    - Haz clic en el ícono del archivo en la parte superior derecha para editar el archivo `settings.json`.
2. Agrega esta configuración:

```json
"[latex]": {
    "files.encoding": "iso88591"
}
```

así queda el mío: 

```bash
{
    "github.copilot.enable": {
        "*": false,
        "plaintext": false,
        "markdown": false,
        "scminput": false,
        "python": false,
        "javascript": false
    },
    "vscode-edge-devtools.webhintInstallNotification": true,
    "security.workspace.trust.untrustedFiles": "open",
    "explorer.confirmPasteNative": false,
    "makefile.configureOnOpen": true,
    "workbench.colorTheme": "One Dark Pro Darker",
    "workbench.iconTheme": "material-icon-theme",
    **"[latex]": {
        "files.encoding": "iso88591"
    }**
}

```

## compilar:

- hay que hacerlo script

```bash
pdflatex Tesis
bibtex Tesis
glosstex Tesis acronimos.gdf
makeindex Tesis.gxs -o Tesis.glx -s glosstex.ist
pdflatex Tesis
```

> *Whenever at any step. Just **press Enter** and keep holding it **until it gets unstuck**. It did work out for me. Try this method first before moving to any harsh steps like other answers are mentioning.*
> 

solo?: 

```bash
pdflatex Tesis
bibtex Tesis
pdflatex Tesis
pdflatex Tesis
```

### Crear un nuevo nivel (por ejemplo, `\subparagraph`)

Puedes definir un nivel adicional con estilo propio:

```latex
\usepackage{titlesec}

\setcounter{secnumdepth}{5} % Permite numerar hasta subparagraph
\setcounter{tocdepth}{5}    % Incluye subparagraph en el índice

\titlesec{\subparagraph}[runin] % Cambia el estilo según tu preferencia
  {\normalfont\normalsize\bfseries} % Estilo del título
  {} % Identación
  {} % Separación
  {} % Código adicional

\begin{document}

\section{Sección principal}
\subsection{Subsección}
\subsubsection{Subsubsección}
\paragraph{Nivel 4: Párrafo}
\subparagraph{Nivel 5: Subpárrafo} Este es otro nivel.

\end{document}

```