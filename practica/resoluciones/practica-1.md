# Práctica 1 – Variables compartidas
## Ejercicio 1

**Enunciado:**  

Para el siguiente programa concurrente suponga que todas las variables están inicializadas en 0 antes de empezar. Indique cual/es de las siguientes opciones son verdaderas:

a) En algún caso el valor de x al terminar el programa es 56.

b) En algún caso el valor de x al terminar el programa es 22.

c) En algún caso el valor de x al terminar el programa es 23.


| P1 | P2 | P3 |
|----|----|----|
| If (x = 0) then<br>y := 4 * 2;<br>x := y + 2; | If (x > 0) then<br>x := x + 1; | x := (x * 3) + (x * 2) + 1; |

### Respuesta:
```
p1:	
    1- Load y, reg1
    2- Add 2, reg1
    3- Store reg1, x	

p2:	
    4- Load x,reg2
	5- Add 1, reg2
	6- Store reg2, x

p3:	
    7- Load x, reg3
	8- Load x, reg4
	9- Multi  reg3, 3, reg 3
    10- Multi reg4, 2, reg4
    11- Add reg3, reg4, reg5
    12- Add 1, reg5
    13- Store x, reg5 
```
* a) V. Es posible que x termine con el valor 56 si se ejecuta el P1, P2 y luego el P3
* b) V. Es posible que x termine con el valor 22 si el orden de las instrucciones es 1- 2- 7- 3- 8- 9- 10- 11- 12- 13 - 4- 5- 6
* c) V. Es posible que x termine con el valor 23 si el orden de las instrucciones es 1-2-7-3-4-5-6-8-9-10-11-12-13

## Ejercicio 2

**Enunciado:**  
Realice una solución concurrente de grano grueso (utilizando <> y/o <await B; S>) para el siguiente problema. Dado un número N verifique cuántas veces aparece ese número en un arreglo de longitud M. Escriba las pre-condiciones que considere necesarias.

### Respuesta:

```
arreglo [M] int total = 0; int N;
process verificar [id: 0..P-1]{
    int comienzo = M/P *id; //para obtener la posición inicial del recorrido de c/ proceso
    int fin = M/P + comienzo; //es el pedazo a recorrer desde donde empieza
    int aux = 0;
    for (int i = comienzo; i < fin; i++){
        if (arreglo [i]== N){
            aux++;
        }
    }
    <total += aux>
}
```
## Ejercicio 3

**Enunciado:**  

Dada la siguiente solución de grano grueso:

a) Indicar si el siguiente código funciona para resolver el problema de Productor/Consumidor con un buffer de tamaño N. En caso de no funcionar, debe hacer las modificaciones necesarias.

| Variables |
|-----------|
| int cant = 0;int pri_ocupada = 0;int pri_vacia = 0;int buffer[N]; |

| Process Productor | Process Consumidor |
|-------------------|--------------------|
| while (true) {<br>produce elemento;<br>&lt;await (cant &lt; N); cant++&gt;<br>buffer[pri_vacia] = elemento;<br>pri_vacia = (pri_vacia + 1) mod N;<br>} | while (true) {<br>&lt;await (cant &gt; 0); cant--&gt;<br>elemento = buffer[pri_ocupada];<br>pri_ocupada = (pri_ocupada + 1) mod N;<br>consume elemento;<br>} |

b) Modificar el código para que funcione para C consumidores y P productores.

### Respuestas:
a)
```
int cant= 0; int pri_ocupada= 0; int pri_vacia=0; it buffer[N]; 
Process Productor::{
    //produce el elemento 
    <await (cant < N); cant++ 
    buffer[pri_vacia] = elemento;>
    pri_vacia = (pri_vacia + 1) MOD n;
    }
}
Process Consumidor::{
	while(true) {
        <await(cant > 0); cant –; 
        elemento= buffer[pri_ocupada>;
        pri_ocupada= (pri_ocupada) + 1) mod N;
        //consume elemento;
	}
}
```

b)
```
int cant= 0; int pri_ocupada= 0; int pri_vacia=0; it buffer[N]; 
Process Productor [id: 0...P-1] {
	while (true) {
        //produce el elemento 
        <await (cant < N); cant++ 
        buffer[pri_vacia] = elemento;
        pri_vacia = (pri_vacia + 1) MOD n;>
    }
}
Process Consumidor [id: 0...C-1] {
	while(true){
		{<await(cant > 0); cant –; 
		elemento= buffer[pri_ocupada];
		pri_ocupada= (pri_ocupada) + 1) mod N >
		//consume elemento;
	}
}
```

## Ejercicio 4

**Enunciado:**  

Resolver con SENTENCIAS AWAIT (<> y <await B; S>). Un sistema operativo mantiene 5 instancias de un recurso almacenadas en una cola, cuando un proceso necesita usar una instancia del recurso la saca de la cola, la usa y cuando termina de usarla la vuelve a depositar.
```
cola C;
int cantRecursos = 5
Elemento elemento
Process Proceso [id: 0..N-1] {
	<await (cantRecursos > 0); 
     elemento = c.pop();
    cantRecursos-–;>
    // Utilizar el recurso
    <c.pop(elemento)
    cantRecursos++;>
}
```

## Ejercicio 5

**Enunciado:**  
En cada ítem debe realizar una solución concurrente degrano grueso (utilizando <> y/o <await B; S>) para el siguiente problema, teniendo en cuenta las condiciones indicadas en el item. Existen N personas que deben imprimir un trabajo cada una.

a) Implemente una solución suponiendo que existe una única impresora compartida por todas las personas, y las mismas la deben usar de a una persona a la vez, sin importar el orden. Existe una función Imprimir(documento) llamada por la persona que simula el uso de la impresora. Sólo se deben usar los procesos que representan a las Personas.

b) Modifique la solución de (a) para el caso en que se deba respetar el orden de llegada.

c) Modifique la solución de (a) para el caso en que se deba respetar el orden dado por el identificador del proceso (cuando está libre la impresora, de los procesos que han solicitado su uso la debe usar el que tenga menor identificador).

d) Modifique la solución de (b) para el caso en que además hay un proceso Coordinador que le indica a cada persona que es su turno de usar la impresora.

### Respuestas:
a)

```
Process Persona [id: 0..N-1];  {
	Documento documento; 
	<imprimir(documento) >
}

// otra alternativa:

boolean libre = false Documento documento
Process Persona [id: 0..N-1];  {
		<await (libre); libre = false >
    imprimir(documento) 
	  <libre  = true>
}
```
b)
 ```
int siguiente=-1;
cola C;
Documento documento

Process Persona [id: 0..N-1];  {
	<if (siguiente = -1)  //si la cola está vacía, impresora "libre"
  siguiente = id //soy el siguiente
	else 
    c.push(id) >; //sino me encolo porque la impresora justamente está ocupada
	<await (siguiente == id) >;       //espero mi turno
	 imprimir(documento);
	<if (empty(C) 
     siguiente = -1 //si la cola está vacía (es decir, no hay nadie esperando), "reseteo" en -1 al siguiente para avisar "impresora libre"
	else 
    siguiente = C.pop()>; //la cola no está vacía, le doy al que está atrás mío el lugar
}

```

c)
```
int siguiente=-1;
colaOrdenada C; // Esta cola agrega ordenado por prioridad
Documento documento

Process Persona [id: 0..N-1];  {
	<if (siguiente = -1) 
    siguiente = id //si la impresora está "libre" soy el siguiente
	else 
    C.push(id)>; //está ocupada, me encolo
	<await (siguiente == id) >;       //espero mi turno
	imprimir(documento)
	<if (empty(C) siguiente = -1 //si se vació la cola, marco "libre" la impresora
	else 
    siguiente = C.push()>; //sino saco al siguiente
}
```



d)
```
int turno = -1;
Cola C;
int espera ([N] 0);
int aviso = 0;

Process Coordinador {
	for(int i= 0; i< N; i++) {
		<await (aviso > 0);
		C.pop(turno)
    aviso --;>
    	<await (espera[turno] = 1)>; //espero que me avisen que terminó
    }
}

Process Persona[id:0..N-1]{
	<C.push(id); //me encolo
  aviso++;> //aviso al coordinador
	<await (turno == id)>; //espero mi turno 
	imprimir(documento); 
	espera[id] = 1; //aviso de fin
}

```
## Ejercicio 6

**Enunciado:**  
Dada la siguiente solución para el Problema de la Sección Crítica entre dos procesos (suponiendo que tanto SC como SNC son segmentos de código finitos, es decir que terminan en algún momento), indicar si cumple con las 4 condiciones requeridas:
| Variables |
|-----------|
| int turno = 1; |

| Process SC1 | Process SC2 |
|-------------|-------------|
| `while (true) {`<br>`while (turno == 2)`<br>`skip;`<br>`SC;`<br>`turno = 2;`<br>`SNC;`<br>`}` | `while (true) {`<br>`while (turno == 1)`<br>`skip;`<br>`SC;`<br>`turno = 1;`<br>`SNC;`<br>`}`<br>`}` |

### Respuesta:
* Exclusión mutua: Cumple. Nunca va a pasar que los dos entren a SC porque "turno" se usa con EM.
* Ausencia de Deadlock (Livelock): Cumple. Nunca se quedan los dos bloqueados dado que el while se encarga de que alguno de los dos procesos entre a su SC mediante "turno"
* Ausencia de Demora Innecesaria: No cumple. Ej: el procesador 2 quiere entrar a su sección crítica y no puede hacerlo porque para ello el procesador 1 debe cambiar el turno, y se queda esperando en su sección no crítica.
* Eventual Entrada: Cumple, ya que son dos procesos que se intercalan.

## Ejercicio 7

**Enunciado:**  
Desarrolle una solución de grano fino usando sólo variables compartidas (no se puede usar las sentencias await ni funciones especiales como TS o FA). En base a lo visto en la clase 3 de teoría, resuelva el problema de acceso a sección crítica usando un proceso coordinador. En este caso, cuando un proceso SC[i] quiere entrar a su sección crítica le avisa al coordinador, y espera a que éste le dé permiso. Al terminar de ejecutar su sección crítica, el proceso SC[i] le avisa al coordinador. Nota: puede basarse en la solución para implementar barreras con “Flags y coordinador” vista en la teoría 3.

### Respuesta:
```
cola c;
int aviso = 0;
boolean comenzar = false; Examen examen;
notas [P-1] int cantCorregidos = P;

process alumno [id: 0..P-1{
  <aviso++>
  <await (comenzar)>
  resolverExamen();
  <c.push(id, examen)>
  <await notas[id] > 0 >
  //se retira
}


process profesores [id: 0..2]{
   <await (aviso == P)>
   comenzar = true;
   while (cantCorregidos < P){
     <await (!c.isEmpty())>
       c.pop(id,examen)>
       calificacion = corregir(id, examen);
       notas[id] = calificacion;
       <cantCorregidos ++>
  }
}
```
