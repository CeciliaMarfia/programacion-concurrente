# Práctica 2 – Semáforos

## Ejercicio 1

**Enunciado:**  
Existen N personas que deben ser chequeadas por un detector de metales antes de poder ingresar al avión.

a. Analice el problema y defina qué procesos, recursos y semáforos/sincronizaciones serán necesarios/convenientes para resolverlo.

b. Implemente una solución que modele el acceso de las personas a un detector (es decir, si el detector está libre la persona lo puede utilizar; en caso contrario, debe esperar).

c. Modifique su solución para el caso que haya tres detectores.

d. Modifique la solución anterior para el caso en que cada persona pueda pasar más de una vez, siendo aleatoria esa cantidad de veces.

### Respuesta:
a) procces Persona, sem libre.
b)
```
sem libre = 1;
process Persona[id: 0..N-1]{
P(libre)
usarDetector();
V(libre)
}
```
c)
```
sem libre = 3;
process Persona[id: 0..N-1]{
P(libre)
usarDetector();
V(libre)
}
```
d)
```
sem libre = 3;
process Persona[id: 0..N-1]{
  while (true){
    P(libre)
    usarDetector();
    V(libre)
  }
}
```

## Ejercicio 2

**Enunciado:**  
Un sistema de control cuenta con 4 procesos que realizan chequeos en forma colaborativa. Para ello, reciben el historial de fallos del día anterior (por simplicidad, de tamaño N). De cada fallo, se conoce su número de identificación (ID) y su nivel de gravedad (0=bajo, 1=intermedio, 2=alto, 3=crítico). Resuelva considerando las siguientes situaciones:

a) Se debe imprimir en pantalla los ID de todos los errores críticos (no importa el orden).

b) Se debe calcular la cantidad de fallos por nivel de gravedad, debiendo quedar los resultados en un vector global.

c) Ídem b) pero cada proceso debe ocuparse de contar los fallos de un nivel de gravedad determinado.

### Respuesta:
a)
```
Fallo historial [N];
int cantBuscar = N DIV 4;

Process P [id: 0..3]{
	int ini = id * cantBuscar; 
	int fin = (ini + cantBuscar - 1);
	for i: ini ..  fin { 
        if (historial[i].id == 3)  writeln(historial[i].id);
    }
}

```
b)
```
Fallo historial [N];
sem mutex = 1;
int cantBuscar = N DIV 4;
int vecGlobal [4] = ([4] 0);

Process P [id: 0..3]{
	int vec[4] = ([4] 0);
	int ini = id * cantBuscar;
	int fin = (ini + cantBuscar - 1);
	for i: ini ..  fin { 
        vec[historial[i].id]++;
    }
    P (mutex);
    for j: 0..3 {
        vecGlobal[j] = vecGlobal[j] + vec[i];
    }
    V (mutex);
    }
```
c)
```
int cant = 0;
Fallo historial [N];
int vec = ([4] 0);

Process P [id: 0..3]{
	int cant = 0;
    for i: 0 .. N-1 {
        if historial[cant].id == id
            cant = cant + 1;
    }
    vec [id] = cant;
}
```
## Ejercicio 3

**Enunciado:** 
Un sistema operativo mantiene 5 instancias de un recurso almacenadas en una cola. Además, existen P procesos que necesitan usar una instancia del recurso. Para eso, deben sacar la instancia de la cola antes de usarla. Una vez usada, la instancia debe ser encolada nuevamente para su reúso.

### Respuesta:
```
Cola c;
int N = 5;
sem mutex = 1; //para el acceso a la cola
sem instancias = 5; // para controlar lo que saco de la cola

Process P [id: 0..P-1] {
	Recurso recurso
while(true){
  P(instancias); //espero la confirmación de recursos disponibles
  P(mutex); //espero que la cola esté disponible
	instancia = c.pop();
	recurso = cola.pop();
	V(mutex);
	// Utilizo el recurso
	P(mutex);
	cola.push(recurso); //vuelvo a encolar la instancia
	V(mutex);
	V(instancias);
 }
}
```
## Ejercicio 4

**Enunciado:** 
Suponga que existe una BD que puede ser accedida por 6 usuarios como máximo al mismo tiempo. Además, los usuarios se clasifican como usuarios de prioridad alta y usuarios de prioridad baja. Por último, la BD tiene la siguiente restricción:

no puede haber más de 4 usuarios con prioridad alta al mismo tiempo usando la BD.
no puede haber más de 5 usuarios con prioridad baja al mismo tiempo usando la BD. Indique si la solución presentada es la más adecuada. Justifique la respuesta.

### Respuesta:
La solución planteada realiza la operación P(total) antes de ejecutar P(alta) o P(baja). Lo cual genera un inconveniente, ya que puede bloquear la ejecución de otros procesos que cumplen con las restricciones y podrían acceder a la base de datos. Generando así demora innecesaria.

## Ejercicio 5

**Enunciado:** 
En una empresa de logística de paquetes existe una sala de contenedores donde se preparan las entregas. Cada contenedor puede almacenar un paquete y la sala cuenta con capacidad para N contenedores. Resuelva considerando las siguientes situaciones:

a) La empresa cuenta con 2 empleados: un empleado Preparador que se ocupa de preparar los paquetes y dejarlos en los contenedores; un empleado Entregador que se ocupa de tomar los paquetes de los contenedores y realizar la entregas. Tanto el Preparador como el Entregador trabajan de a un paquete por vez.

b) Modifique la solución a) para el caso en que haya P empleados Preparadores.

c) Modifique la solución a) para el caso en que haya E empleados Entregadores.

d) Modifique la solución a) para el caso en que haya P empleados Preparadores y E empleadores Entregadores.

### Respuesta:
a)
```
buffer[n];// en el proceso de producto consumidor, se usa buffer que es más eficiente
sem vacio=n, lleno=0;
int ocupado=0, libre=0;

Process Preparador {
    while (true){
        P(vacio); // me fijo si tengo espacio en la sala
        buffer[libre]=paquete; // lo posiciono en ese contenedor libre
        libre=(libre+1) mod n; //posiciono a libre en la próximo contenedor libre
        v(lleno); //aviso que un contenedor está lleno
    }
}

Process Entregador {
    while (true){
        P(lleno); //me fijo si algún contenedor tiene algo
        entrega=buffer[ocupado]; //entregó el paquete
        ocupado=(ocupado+1)mod n; //saco la próxima entrega
        V(vacio); //aumento la cantidad de espacio en la sala
    }
}
```

b)

```
buffer[n]; // en el proceso de producto consumidor, se usa buffer que es más eficiente
sem vacio=n, lleno=0;
int ocupado=0,libre=0;
sem mutePreparador=1;

Process Preparador[id:0..P-1]{
    while (true){
        P(vacio); // me fijo si tengo espacio en la sala
        P(mutePreparador);
        buffer[libre]=paquete; //lo posiciono en ese contenedor libre
        libre=(libre+1) mod n; //posiciono a libre en la próximo contenedor libre
        V(mutePreparador);
        v(lleno); //aviso que un contenedor está lleno
    }
}

Process Entregador {
    while (true){
        P(lleno); //me fijo si algún contenedor tiene algo
        entrega=buffer[ocupado]; //entregó el paquete
        ocupado=(ocupado+1)mod n; //saco la próxima entrega
        V(vacio); //aumento la cantidad de espacio en la sala
    }
}
```

c)

```
buffer[n];// en el proceso de producto consumidor, se usa buffer que es más eficiente
sem vacio=n, lleno=0;
int ocupado=0,libre=0;
sem mutePreparador=1, muteEntregadores=1;

Process Preparador {
    while (true){
        P(vacio);// me fijo si tengo espacio en la sala
        buffer[libre]=paquete;//lo posiciono en ese contenedor libre
        libre=(libre+1) mod n;//posiciono a libre en la próximo contenedor libre
        v(lleno); //aviso que un contenedor está lleno
    }
}

Process Entregador [id: 0..E-1]{
    while (true){
        P(lleno); //me fijo si algún contenedor tiene algo
        P(muteEntregadores);
        entrega=buffer[ocupado]; //entregó el paquete
        ocupado=(ocupado+1)mod n; //saco la próxima entrega
        V(muteEntregadores);
        V(vacio); //aumento la cantidad de espacio en la sala
    }
}
```

d)
```
buf[n];// en el proceso de producto consumidor, se usa buffer que es más eficiente

sem vacio=n, lleno=0;
int ocupado=0,libre=0;
sem mutePreparador=1, muteEntregadores=1;

Process Preparador[id:0..P-1]{
    while (true){
        P(vacio);// me fijo si tengo espacio en la sala
        P(mutePreparador);
        buf[libre]=paquete;//lo posiciono en ese contenedor libre
        libre=(libre+1) mod n;//posiciono a libre en la próximo contenedor libre
        V(mutePreparador);
        v(lleno); //aviso que un contenedor está lleno
    }
}

Process Entregador [id: 0..E-1]{
    while (true){
        P(lleno); //me fijo si algún contenedor tiene algo
        P(muteEntregadores);
        entrega=buf[ocupado]; //entrego el paquete
        ocupado=(ocupado+1)mod n; //saco la próxima entrega
        V(muteEntregadores);
        V(vacio); //aumento la cantidad de espacio en la sala
    }
}
```
## Ejercicio 6

**Enunciado:** 
Existen N personas que deben imprimir un trabajo cada una. Resolver cada ítem usando semáforos:

a) Implemente una solución suponiendo que existe una única impresora compartida por todas las personas, y las mismas la deben usar de a una persona a la vez, sin importar el orden. Existe una función Imprimir(documento) llamada por la persona que simula el uso de la impresora. Sólo se deben usar los procesos que representan a las Personas.

b) Modifique la solución de (a) para el caso en que se deba respetar el orden de llegada.

c) Modifique la solución de (a) para el caso en que se deba respetar estrictamente el orden dado por el identificador del proceso (la persona X no puede usar la impresora hasta que no haya terminado de usarla la persona X-1).

d) Modifique la solución de (b) para el caso en que además hay un proceso Coordinador que le indica a cada persona que es su turno de usar la impresora.

e) Modificar la solución (d) para el caso en que sean 5 impresoras. El coordinador le indica a la persona cuando puede usar una impresora, y cual debe usar.

### Respuesta:
a)
```
sem mutex = 1;

Process Persona [id: 0 .. N-1] {
	Documento documento;
	P (mutex);
	Imprimir (documento);
	V (mutex);
}
```
b)

```
cola c;
sem mutex = 1, sem espera([N] 0)
boolean libre = true;


Process Persona [id: 0 .. N-1] {
	Documento documento;
	int aux;
	P (mutex);
	if (libre)  {
		libre = false;
		V (mutex);
    } else {
        c.push(id);
        V (mutex);
        P (espera [id]);
    }
    Imprimir (documento);
    P (mutex);
    if (c.isEmpty())  libre = true
    else {
        c.pop(aux);
        V (espera[aux]);
    }
    V (mutex);
}
```

c)

```

sem turno ([N] 0)

process Persona[id: 0 ..P-1]{
  if(id != 1){
    P(turno[id])
    imprimir(documento);
    V(turno[id])
  }else{
    imprimir(documento);
    V(turno[id]+1)
  }
}
```

d)
```
sem mutex = 1; sem llegada = 0; sem termine = 0;
cola c;
Process Persona [id: 0.. N-1] {
	Documento documento;
	P (mutex);
	c.push(id);
	V (mutex);
	V (llegada);
	P (turno[id]);
	imprimir (documento);
	V (termine);
}

Process Coordinador {
	int id;
	while(true) {
        P (llegada);
        P (mutex);
        c.pop(id);
        V(mutex);
        V(turno[id]);
        P (termine);
    }
}

En un problema de cliente-servidor hay que hacer la sincronización final, como en los ejercicios de los médicos o profesores tomando examen oral. Solo en este caso de administración de recurso compartido, no de espera de resultado.
```
e)
```
sem mutex = 1;
sem termine = 5; //como contador de recurso
sem llegada = 0;
sem turno ([N] 0); cola c1, c2;
sem mutex2 = 1;

Process Persona [id: 0.. N-1] {
	Documento documento;
  int imp[]; //nro. impresora
  P (mutex);
	c1.push(id);
	V (mutex);
	V (llegada);
	P (turno[id]);
	imprimir (documento);
	 P (mutex2);
   c2.push(id);
   V (mutex2);
  V (termine);
}

Process Coordinador {
	int id; int i;
	for (int 1; i < 5; i++){ //esto lo hago para tener impresora para pushear, solo la 1ra vez
     P (mutex2);
     c2.push(id);
     V (mutex2);
  }
  while(true) {
        P (termine); //espera impresora libre
        P (mutex2);
        c2.pop(i);//prox impresora
        V(mutex2);
        P(llegada);
        P(mutex);
        c1.pop(id); //a quien se la asigno
        vector[id])= i;
        V(turno[id]);
    }
}

```
## Ejercicio 7

**Enunciado:** 
Suponga que se tiene un curso con 50 alumnos. Cada alumno debe realizar una tarea y existen 10 enunciados posibles. Una vez que todos los alumnos eligieron su tarea, comienzan a realizarla. Cada vez que un alumno termina su tarea, le avisa al profesor y se queda esperando el puntaje del grupo (depende de todos aquellos que comparten el mismo enunciado). Cuando un grupo terminó, el profesor les otorga un puntaje que representa el orden en que se terminó esa tarea de las 10 posibles. Nota: Para elegir la tarea suponga que existe una función elegir que le asigna una tarea a un alumno (esta función asignará 10 tareas diferentes entre 50 alumnos, es decir, que 5 alumnos tendrán la tarea 1, otros 5 la tarea 2 y así sucesivamente para las 10 tareas).


### Respuesta:


```
int tareas[10] = ([10], 0);
int puntajes[1..10] = [(10), 0];
int contador = 0;
sem sigo [10] = ([10], 0);
sem mutex = 1;
sem barrera = 0;
sem llegue = 0;

Process Alumno [id: 0..49] {
	int numTarea = elegir();
	P (mutex);
	contador = contador + 1;
	if (contador == 50) {
		for int i = 1..50 V (barrera);
    }
	V (mutex);
	P (barrera);
	// Realizar tarea
	P (mutex);
	cola.push (numTarea);
	V (mutex);
	V (llegue);
	P (sigo[numTarea]);
	int nota = puntaje[numTarea];
}

Process Profesor {
	int numTarea;
	int puntaje = 1;
	for i: 0..49 {
		P (llegue);
		P (mutex);
		c.pop (numTarea);
		V (mutex);
		tareas [numTarea]++;
        if tareas [numTarea] == 5 {
            puntajes [numTarea] = puntaje;
            for j: 0..4 {
                V (sigo[numTarea]);
            }   
        }
        puntaje++;
    }
}
```
RECORDATORIO: siempre que agrego algo a una cola para que otro proceso lo tome DEBO AVISARLE al otro.

## Ejercicio 8

**Enunciado:** 
Una fábrica de piezas metálicas debe producir T piezas por día. Para eso, cuenta con E empleados que se ocupan de producir las piezas de a una por vez. La fábrica empieza a producir una vez que todos los empleados llegaron. Mientras haya piezas por fabricar, los empleados tomarán una y la realizarán. Cada empleado puede tardar distinto tiempo en fabricar una pieza. Al finalizar el día, se debe conocer cual es el empleado que más piezas fabricó.

a) Implemente una solución asumiendo que T > E.

b) Implemente una solución que contemple cualquier valor de T y E.

### Respuesta:
```
int cantPiezas = 0; //las variables a chequear porque me dio fiaca corregirlas (:
int contador = 0;
int max = 0;
int maxEmpleado = 0;
sem mutex = 1;
sem extraccion = 1;
sem maximo = 1;
sem barrera = 0;
int piezaXEmp([E]0);
Process Empleado [id: 0..E-1] {
    int piezas = 0;
    //llegada del empleado
    P(mutex);
    contador = contador + 1;
    if (contador == E) { 
        for int i = 1..E  V(barrera);
    }
    V(mutex);
    P(barrera);
    // Empieza a producir
	P(extraccion);
	while(cantPiezas < T){
		cantPiezas++; // 
		V(extraccion);
		// Fabrica la pieza
		piezaXEmp[id] = 	piezaXEmp[id] + 1; //incremento la cantidad hecha por cada empleado 
		P(extraccion);
	}
	V(extraccion);
	P(maximo); 
  termine ++;
	if(termine == E) {
		calcularMaximo(cantP, id,idMax,max);
    imprimir(idMax);
	}
	V(maximo);
}

```
## Ejercicio 9

**Enunciado:** 
Resolver el funcionamiento en una fábrica de ventanas con 7 empleados (4 carpinteros, 1 vidriero y 2 armadores) que trabajan de la siguiente manera:

Los carpinteros continuamente hacen marcos (cada marco es armado por un único carpintero) y los deja en un depósito con capacidad de almacenar 30 marcos.
El vidriero continuamente hace vidrios y los deja en otro depósito con capacidad para 50 vidrios.
Los armadores continuamente toman un marco y un vidrio (en ese orden) de los depósitos correspondientes y arman la ventana (cada ventana es armada por un único armador).

### Respuesta:
Hacemos cola porque son recursos compartidos!

```
Cola colaMarcos;
Cola colaVidrios;
sem cantMarcos = 30;
sem cantVidrios = 50;
sem marcosHechos = 0;
sem vidriosHechos = 0;
sem mutexMarcos = 1;
sem mutexVidrios = 1;

Process Carpintero[id: 0 .. 3] {
	Marco m;
	while(true){
		P (cantMarcos);
    m = armarMarco();
		P (mutexMarcos);
		colaMarcos.push (m);
		V (mutexMarcos);
		V(marcosHechos);
	}
}

Process Vidriero {
	Vidrio v;
	while(true) {
		P (cantVidrios);
    v = armarVidrio();
		P (mutexVidrios);
		colaVidrios.push (v);
		V (mutexVidrios);
		V(vidriosHechos);
	}
}

Process Armador[id: 0 .. 1] {
	Ventana ven;
	Vidrio v;
	Marco m;
	while(true) { 
		P(marcosHechos);
		P (mutexMarcos);
		colaMarcos.pop(m);
		V (mutexMarcos);
		V (cantMarcos);
		P (vidriosHechos);
		P (mutexVidrios);
		colaVidrios.pop (v);
		V (mutexVidrios);
		V (cantVidrios);
		ven = armarVentana (m, v); //Luego de esto el profe sugería hacer otra cola de listo pero para que quede más "prolijo"
	}
}
```
## Ejercicio 10

**Enunciado:** 
A una cerealera van T camiones a descargarse trigo y M camiones a descargar maíz. Sólo hay lugar para que 7 camiones a la vez descarguen, pero no pueden ser más de 5 del mismo tipo de cereal.

a) Implemente una solución que use un proceso extra que actúe como coordinador entre los camiones. El coordinador debe atender a los camiones según el orden de llegada. Además, debe retirarse cuando todos los camiones han descargado. 

b) Implemente una solución que no use procesos adicionales (sólo camiones). No importa el orden de llegada para descargar. Nota: maximice la concurrencia.

### Respuesta:
a)
```
sem libre = 7
sem tipo [2] = ([2] 5)
sem paso[N] = ([N] 0)
sem general = 0
sem mutex = 1
cola c;

process camion [id:0..N-1]{
	int miTipo;
	P(mutex)
	c.push(id,tipo)
	V(mutex)
	V(general) //aviso que llegué
	P(paso[id]) //espero el paso
	descargar()
	V(libre) //deja lugar
	V(llegaTipo[miTipo]) //dejo lugar para mi tipo

	process coordinador{
		for 1 to N
			P(general) //espero camión
			P(mutex)
			c.pop(id,miTipo)
			v(mutex)
			P(libre) //espero lugar
			P(llegaTipo[miTipo]) //espero lugar para mi tipo
			V(paso[id]) //habilito el paso
	}
}
```

b)
```
int cantCamionesTrigo = T;
int cantCamionesMaiz = M;
sem lugaresLibres = 7;
sem lugaresTrigo = 5;
sem lugaresMaiz = 5;

Process CamionTrigo[id = 1 .. T] {
	P(lugaresTrigo);
	P(lugaresLibres);
	// Descarga
	V(lugaresLibres);
	V(lugaresTrigo);
}

Process CamionMaiz[id = 1 .. M] {
	P(lugaresMaiz);
	P(lugaresLibres);
	// Descarga
	V(lugaresLibres);
	V(lugaresMaiz);
}
```

## Ejercicio 11

**Enunciado:**
En un vacunatorio hay un empleado de salud para vacunar a 50 personas. El empleado de salud atiende a las personas de acuerdo con el orden de llegada y de a 5 personas a la vez. Es decir, que cuando está libre debe esperar a que haya al menos 5 personas esperando, luego vacuna a las 5 primeras personas, y al terminar las deja ir para esperar por otras 5. Cuando ha atendido a las 50 personas el empleado de salud se retira. Nota: todos los procesos deben terminar su ejecución; suponga que el empleado tienen una función VacunarPersona() que simula que el empleado está vacunando a UNA persona.

### Respuesta:
```
sem mutex = 1, atender = 0, listo = 0;
sem esperaVacuna [50] = ([50], 0), esperaSalir [50] = ([50], 0);
cola c;
int cant = 0;

Process Persona [id: 0..49] {
  //llegada
	P (mutex);
	c.push (id);
  V (mutex);
  P (esperaVacuna[id]); // Espera a ser llamada 
   P (esperaSalir[id]);
  //se retira
}

Process Empleado {
	int id cola c; //cola listos
  	for (int i = 0; i < 10; i++){
      	for (int i = 0; i <= 5; i++){ //para asegurarme que hay 5 personas en la cola
        P(llegada)
        	for (int j = 0; j < 5; j++){
            P(mutex)
            c.pop(id);
            V(mutex)
            V (esperaVacuna[id]); // aviso
            vacunarPersona(id);
            listos.push(id);
            }
        for (int i = 0; i < 5; i++){
           listos.pop(id);
           V (esperaSalir[id]);
        }
}
```

## Ejercicio 12

**Enunciado:** 
Simular la atención en una Terminal de Micros que posee 3 puestos para hisopar a 150 pasajeros. En cada puesto hay una Enfermera que atiende a los pasajeros de acuerdo con el orden de llegada al mismo. Cuando llega un pasajero se dirige al Recepcionista, quien le indica qué puesto es el que tiene menos gente esperando. Luego se dirige al puesto y espera a que la enfermera correspondiente lo llame para hisoparlo. Finalmente, se retira.

a) Implemente una solución considerando los procesos Pasajeros, Enfermera y Recepcionista.

b) Modifique la solución anterior para que sólo haya procesos Pasajeros y Enfermera, siendo los pasajeros quienes determinan por su cuenta qué puesto tiene menos personas esperando.

Nota: suponga que existe una función Hisopar() que simula la atención del pasajero por parte de la enfermera correspondiente.
### Respuesta:

```
int salas [3] = ([3], 0);
int pasajeros [150] = ([150], 0);
Cola espera;
Cola vecColas [3];
sem mutexEspera =1;
sem llegue = 0;
sem salaAsignada [150] = ([150], 0);
sem mutexColaEnfermera[3]=([3], 1);
sem tengoQueAtender[3]=([3], 0);
sem podesPasar[150]=([150], 0);
sem listo [3] = ([3], 0);

Process Pasajero[id: 0..149]{
	// Llega el pasajero
	P(mutexEspera);
	espera.push(id);
	V(mutexEspera);
	V(llegue);
	P(salaAsignada[id]);
	int salaAsignada = pasajeros[id];
	P(mutexColaEnfermera[salaAsignada]);
	vecColas[salaAsignada].push(id);
	V(mutexColaEnfermera[salaAsignada]);
	V(tengoQueAtender[salaAsignada]);
	P(podesPasar[id]);
	V (listo[salaAsignada]);
	P(podesPasar[id]);
}

Process Recepcionista{
	for int i: 1 .. 150 {
		P(llegue);
		int salaMin = min(salas);
		P(mutexEspera);
		id=espera.pop();
		salas[salaMin]++;
		V(mutexEspera);
		pasajeros[id]=salaMin;
		V(salaAsignada[id]);
    }
}
		
Process Enfermera[id:0..2]{
	while(true){
		P(tengoQueAtender[id]);
		P(mutexColaEnfermera[id]);
		int persona=vecColas[id].pop();
		salas[id]--;
		V(mutexColaEnfermera[id]);
		V(podesPasar[persona]);
		P (listo[id]);
		Hisopar (persona);
		V(podesPasar[persona]);
	}
}
```
