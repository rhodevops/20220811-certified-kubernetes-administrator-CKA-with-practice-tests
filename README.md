# README

## Repo

`url` https://github.com/rhodevops/pandoc-latex-template-T0501.git

## Plantilla pandoc de .md a .pdf

- Se utiliza un `makefile`. Para ejecutarlo se necesita:  
    1. GNU make
    2. Pandoc
- Requiere latex, utiliza el motor `xeLatex`
- Hay que editar los archivos `make/src/changeit.md` y `make/metadata.yaml`

## Generar documento pdf

Desde el dirrectorio raiz del repo

```shell
make
```
