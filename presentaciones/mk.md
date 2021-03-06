## Ejemplo: quiero saber cuáles de las 170 secuencias de interés se encuentran en uno de los 10 conjuntos de datos

Supongamos que es un análisis conocido y estandarizado, que se resuelve
con los siguientes
    commandos:

    bin/get_seq seq/secuencia.txt data/datos.fastq > results/secuencia@datos.fastq.extras
    bin/clean_seq results/secuencia@datos.fastq.extras > results/secuencia@datos.fastq

. . .

Entonces sólo ejecutamos esos comandos 1700
veces.

## Descarguemos los ejercicios

    git clone https://github.com/INMEGEN/ejercicios-mk

## Ejercicio 1 ¿La secuencia P\_49627 se encuentra en el conjunto de datos C2?

. .
    .

    bin/get_seq seq/P_49627.txt data/C2.fastq > results/P_49627@C2.fastq.extras
    bin/clean_seq results/P_49627@C2.fastq.extras > results/P_49627@C2.fastq

## `mk` es una herramienta para automatizar trabajos

![`mk` genera un árbol de dependencias.](../imagenes/mk-tree.png)

## `mk` requiere un archivo de configuración que tiene la forma

`objetivo($target):ATTRIBUTOS: ingredientes($prereq)`  
`<tab> receta(instrucciones)`

Se usa `<tab>` para indicar los comandos de una *receta*.

Por defecto `mk` hace el primer
objetivo.

## Una analogía para entender `mk`: `mk` es el cocinero, `mkfile` el recetario

`mk` hace las cosas que le dices.

`mkfile` tiene las instrucciones para
hacerlas.

## Para hacer un mk tenemos que tener claro el platillo y los ingredientes

Nombrar el platillo (\$target) y los ingredientes (\$prereq):

    $ cat mkfile
    results/pato.asado:    data/pato

## Luego se agrega la receta

    $ asar --con-pimienta pato > pato.asado
    #
    # se vuelve:
    #
    $ cat mkfile
    results/pato.asado:    data/pato
        asar --con-pimiemta $prereq > $target

## Y finalmente podemos usar el recetario

    mk results/pato.asado

## Ejercicio 2: ¿Cuál es el platillo y el ingrediente para encontrar la secuencia P\_49627 en el conjunto de datos C2?

    bin/get_seq seq/P_49627.txt data/C2.fastq > results/P_49627@C2.fastq.extras
    bin/clean_seq results/P_49627@C2.fastq.extras > results/P_49627@C2.fastq

<!--
```
$ cat mkfile
results/P_49627@C2.fastq:   results/P_49627@C2.fastq.extras

results/P_49627@C2.fastq.extras:    seq/P_49627.txt data/C2.fastq
```
-->

## Ejercicio 3: rellenar las instrucciones de la receta para encontrar el resultado

Sabremos que funciona porque tendremos el resultado correcto al
ejecutar

    mk results/P_49627@C2.fastq

# Ejercicio 4: En cada una de la receta ¿Cuál es el platillo que estamos preparando (\$target)? ¿Cuáles son los ingredientes (\$prereq)?

Reemplazar dentro de la receta, el platillo y los ingredientes por
variables

    $ cat mkfile
    results/P_49627@C2.fastq:   results/P_49627@C2.fastq.extras
        bin/clean_seq results/P_49627@C2.fastq.extras > results/P_49627@C2.fastq
    
    results/P_49627@C2.fastq.extras:    seq/P_49627.txt data/C2.fastq
        bin/get_seq seq/P_49627.txt data/C2.fastq > results/P_49627@C2.fastq.extras

## En ocasiones muchas recetas comparten la misma estructura

    $ asar --con-pimienta data/pato > results/pato.asado
    $ asar --con-pimienta data/bistec > results/bistec.asado
    $ asar --con-pimienta data/pollo > results/pollo.asado
    #
    # se vuelve:
    #
    $ cat mkfile
    results/pato.asado:    data/pato
        asar --con-pimienta $prereq > $target
    
    results/bistec.asado:    data/bistec
        asar --con-pimienta $prereq > $target
    
    results/pollo.asado:    data/pollo
        asar --con-pimienta $prereq > $target

## Se puede generalizar una receta usando un comodín (%)

    $ asar --con-pimienta data/pato > results/pato.asado
    $ asar --con-pimienta data/bistec > results/bistec.asado
    $ asar --con-pimienta data/pollo > results/pollo.asado
    #
    # se vuelve:
    #
    $ cat mkfile
    results/%.asado:    data/%
        asar --con-pimienta $prereq > $target

## Ejercicio 5: Hacer que el objetivo final `.fastq` sea general:

    $ cat mkfile
    results/P_49627@C2.fastq:   results/P_49627@C2.fastq.extras
        bin/clean_seq "${prereq}" > "${target}"
    
    results/P_49627@C2.fastq.extras:    seq/P_49627.txt data/C2.fastq
        bin/get_seq ${prereq} > "${target}"

## Para hacer general el estado intermedio necesitamos un lenguaje para expresar patrones de texto

El lenguaje se llama «Expresión regular»

![](../imagenes/regexp_1.png) ![](../imagenes/regexp_2.png)
![](../imagenes/regexp_3.png)

## Ejercicio 6 (regexp): Seguir las instrucciones para cambiar las variables por expresiones regulares

    $ cat mkfile
    results/%.fastq:    results/%.fastq.extras
        bin/clean_seq "${prereq}" > "${target}"
    
    results/P_49627@C2.fastq.extras:    seq/P_49627.txt data/C2.fastq
        bin/get_seq ${prereq} > "${target}"

## Se le puede decir a mk que haga muchas cosas a la vez

    mk results/P_5029@D2.fastq results/P_5029@B1.fastq \
       results/P_5029@D1.fastq results/P_5029@B2.fastq \
       […] # otros 1696 objetivos

## Por comodidad hacemos un script que haga de mesero y le diga a mk todo lo que tiene que cocinar

    bin/targets | xargs mk
