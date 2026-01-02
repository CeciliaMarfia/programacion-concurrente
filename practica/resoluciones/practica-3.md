# Práctica 3 – Monitores

## Ejercicio 1

**Enunciado:**  
Se dispone de un puente por el cual puede pasar un solo auto a la vez. Un auto pide permiso para pasar por el puente, cruza por el mismo y luego sigue su camino.
```
Monitor Puente
    cond cola;
    int cant= 0;

    Procedure entrarPuente ()
        while ( cant > 0) wait (cola);
        cant = cant + 1;
    end;

    Procedure salirPuente ()
        cant = cant – 1;
        signal(cola);
    end;
End Monitor;

Process Auto [a:1..M]
    Puente. entrarPuente (a);
    “el auto cruza el puente”
    Puente. salirPuente(a);
End Process;
```
a. ¿El código funciona correctamente? Justifique su respuesta.

b. ¿Se podría simplificar el programa? ¿Sin monitor? ¿Menos procedimientos? ¿Sin variable condition? En caso afirmativo, rescriba el código.

c. ¿La solución original respeta el orden de llegada de los vehículos? Si rescribió el código en el punto b), ¿esa solución respeta el orden de llegada?
 
### Respuestas:

a) A priori la lógica aplicada es correcta, aunque al invocar entrarPuente(a) se envía el parámetro "a" pero en el procedimiento no aparece dicho parámetro. Debe ser un error de tipeo de la práctica.
Lo que sucede es un proceso Auto invocará al procedimiento entrarPuente(a) donde se incrementa la variable cant y mientras la variable cant sea mayor a 0, es decir, que haya autos esperando pasar, "se dormirá" en la cola. Una vez que un auto invoque a salirPuente(a) (lo cual indica que ya pasó) decrementará la variable cant y despertará al 1er encolado el cual vuelve a competir por el acceso al monitor nuevamente.

b) Recordando que los monitores representan el recurso compartido y la EM es implícita dentro de un monitor al no poder ejecutar más de una llamada a un procedimiento a la vez hasta que no se  termina el procedure o "se duerme" en una variable condición no se libera el monitor para atender otro llamado.
Simplificando:
```
Monitor Puente{
  procedure cruzarPuente(a){
  //cruzar puente
  }
}

process Auto [a: 0..M-1]{
  Puente.cruzarPuente(a);
}
```

Sin monitor:
```
sem puente = 1;
Process Auto [a:1..M] {
	P (puente);
	// Cruzo
	V (puente);
}
```
c)No, la 1er solución no respeta el orden de llegada ya que al despertar al 1ro de la cola no garantiza el acceso inmediato ni prioritario al monitor sino que el proceso vuelve a competir por el acceso al monitor.
La solución b tampoco respeta el orden de llegada porque también van a competir por el acceso al monitor. debería utilizarse la técnica passing the condition para garantizar el orden.

## Ejercicio 2

**Enunciado:**  
Existen N procesos que deben leer información de una base de datos, la cual es administrada por un motor que admite una cantidad limitada de consultas simultáneas.

a. Analice el problema y defina qué procesos, recursos y monitores/sincronizaciones serán necesarios/convenientes para resolverlo.

b. Implemente el acceso a la base por parte de los procesos, sabiendo que el motor de base de datos puede atender a lo sumo 5 consultas de lectura simultáneas.


### Respuestas:
a) Se necesitarán N procesos lectores. El motor será el monitor ya que será quien administre el acceso a la lectura de la BD. Necesitaremos una variable entera que cuente la cantidad de consultas y una variable condition de espera.

b)
```
Process Lector [id: 0..N-1] {
	Motor.EntrarABD ();
	// leer la BD
	Motor.SalirDeBD();
}

Monitor Motor {
	cond espera;
  int cant=0;
	
	Procedure EntrarABD() {
		while (cant == 5) {
			wait (espera);
        }
        cant++;
    }

    Procedure SalirDeBD() {
        cant- –;
        signal(espera);
        }
    }
```

## Ejercicio 3

**Enunciado:**  
Existen N personas que deben fotocopiar un documento. La fotocopiadora sólo puede ser usada por una persona a la vez. Analice el problema y defina qué procesos, recursos y monitores serán necesarios/convenientes, además de las posibles sincronizaciones requeridas para resolver el problema. Luego, resuelva considerando las siguientes situaciones:

a. Implemente una solución suponiendo no importa el orden de uso. Existe una función Fotocopiar() que simula el uso de la fotocopiadora.

b. Modifique la solución de (a) para el caso en que se deba respetar el orden de llegada.

c. Modifique la solución de (b) para el caso en que se deba dar prioridad de acuerdo con la edad de cada persona (cuando la fotocopiadora está libre la debe usar la persona de mayor edad entre las que estén esperando para usarla).

d. Modifique la solución de (a) para el caso en que se deba respetar estrictamente el orden dado por el identificador del proceso (la persona X no puede usar la fotocopiadora hasta que no haya terminado de usarla la persona X-1).

e. Modifique la solución de (b) para el caso en que además haya un Empleado que le indica a cada persona cuando debe usar la fotocopiadora.

f. Modificar la solución (e) para el caso en que sean 10 fotocopiadoras. El empleado le indica a la persona cuál fotocopiadora usar y cuándo hacerlo
### Respuestas:
a)
```
Process Persona [id: 0..N-1] {
	Fotocopiadora.imprimir();
}

Monitor Fotocopiadora {
	Imprimir();
}
```
b)
```
Process Persona [id: 0..N-1] {
	txt documento, copia;

	Fotocopiadora.Pasar();
	Fotocopiar(documento, copia);
	Fotocopiadora.Salir();
}

Monitor Fotocopiadora{
    bool libre = true;
    cond cola;
    int esperando = 0;
    Procedure Pasar () { 
        if(not libre) { 
            esperando ++;
            wait(cola);
        }
        else libre = false;
    }

    Procedure Salir (){ 
        if(esperando > 0 ) { 
            esperando --;
            signal(cola);
        }
        else libre = true;
    }
}
```
c)
```
Process Persona [id: 0..N-1] {
	int edad = …;
	Fotocopiadora.Pasar(id, edad);
	Fotocopiar();
	Fotocopiadora.Salir();
}

Monitor Cajero{
    bool libre = true;
    cond espera[N];
    int idAux, esperando = 0;
    colaOrdenada fila;


    Procedure Pasar (intP, edad: in int) { 
        if(not libre) { //si está ocupado
            fila.push(idP, edad); //agrego ordenado por edad en la cola
            esperando++;
            wait (espera[idP]); //me duermo en la variable cond 
        }
        else libre = false; //actualizo el estado de la fotocopiadora a ocupada
    }

    Procedure Salir () { 
        if(esperando > 0 ) { //si hay alguien esperando
            esperando --; //decremento el contador
            fila.pop (idAux); //saco al siguiente
            signal(espera [idAux]); //despierto al desencolado
        }
        else libre = true; //actualizo a libre porque no hay nadie esperando
    }
}
```
d)
```
Process Persona [id: 0..N-1] {
	Fotocopiadora.Pasar(id);
	Fotocopiar();
	Fotocopiadora.Salir();
}

Monitor Cajero {
    int proximo = 0;
    cond espera[N];

    Procedure Pasar (idP: in int) { 
        if (idP != proximo) { //porqué if y no while: porque una vez que me desperté no tengo que volver a preguntar si me toca
            wait (espera[idP]);
        }
    }

    Procedure Salir (){ 
        proximo++;
		signal(espera[proximo]);
        }   
}
```
e)
```
Process Persona[id: 0..N-1]{
	fotocopiadora.pedido(id);
	//imprime
	fotocopiadora.terminar();
}


Process Empleado{
	for i: 1 .. N {
		fotocopiadora.sig();
	}
}

Monitor fotocopiadora(){
	Cola c;
	cond espera;
	cond hayPedido;
	cond imprimiendo;
	int esperar;

	Procedure pedido(id: in int ) {
		esperar++;
		signal(hayPedido);
		wait(espera);
	}

	Procedure sig(id: in int) {
		if(not libre){
			wait(hayPersona); //como no está libre demoro al empleado
		}
		if (esperar == 0 ){
			wait(hayPedido)
		}
		signal(espera);
		libre = false; //como hay alguien y está libre lo despierto
		esperar- -; 
}

	Procedure terminar() {
		libre = true;
		signal(hayPersona);
    }	
}
```
f) este no lo corregí, toca confiar
```
Process Persona[id: 0..N-1]{
	int idI;
	escritorio.llegue(id, idI);
    fotocopiadora[idI].imprimir();
	escritorio.terminar(idI);
}

Process Empleado{
	while(true) {
		escritorio.sig();
	}
}

Monitor escritorio(){
	Cola C;
    Cola fotocopiadoras[10];
	cond espera;
	cond personaEnEspera;
	cond esperaImpresoras;
	cond esperaAsignacion[N];
	vector personas[N];

	Procedure llegue(id: in int, idI: out int ){
		push(C, id);
		signal(personaEnEspera);
		wait(esperaAsignacion[id]);
	}

	Procedure sig(id: in int){
		if(empty(C)) {
			wait(personaEnEspera);
		}
		pop(C,id);
		if(fotocopiadoras.isEmpty()){
			wait(esperaImpresoras);
        }	
	    int i = impresoras.pop();
		personas[id]=i;
		signal(esperaAsignacion[id]);
	}

	Procedure terminar(id: in int){
		fotocopiadoras.push(id);
		signal(esperaImpresoras);
    }	
	
}

Monitor fotocopiadora()[id:0..9]{
	Procedure imprimir() {
        // Imprime
    }	
}
```
## Ejercicio 4

**Enunciado:**  
Existen N vehículos que deben pasar por un puente de acuerdo con el orden de llegada. Considere que el puente no soporta más de 50000kg y que cada vehículo cuenta con su propio peso (ningún vehículo supera el peso soportado por el puente).


### Respuestas:
este lo hice con Lauti pero creo que él tiene otra versión posible
```
Process Vehiculo [id: 0..N-1] {
	int peso = …;
	Puente.Pasar(peso);
	// Cruza el puente
	Puente.Salir(peso);
}

Monitor Puente {
	bool libre = true;
	int esperando = 0;
	double pesoTotal = 0;
	cond esperaAutos;
	cond pesoEspera;

    Procedure Pasar (peso: in double) {
        if (not libre) {
            esperando++;
            wait (esperaAutos);
        } 
        else libre = false;
        while (pesoTotal + peso > 500000) {
            wait (pesoEspera);
        }
        pesoTotal += peso;
        if (esperando > 0) {
            esperando–;
            signal(esperaAutos);
        } 
        else libre = true;
    }

    Procedure Salir (peso: in double) {
        pesoTotal -= peso;
        signal (pesoEspera);
    }
}
```
## Ejercicio 5

**Enunciado:**  
En un corralón de materiales se deben atender a N clientes de acuerdo con el orden de llegada. Cuando un cliente es llamado para ser atendido, entrega una lista con los productos que comprará, y espera a que alguno de los empleados le entregue el comprobante de la compra realizada.

a. Resuelva considerando que el corralón tiene un único empleado.

b. Resuelva considerando que el corralón tiene E empleados (E > 1). Los empleados no deben terminar su ejecución.

c. Modifique la solución (b) considerando que los empleados deben terminar su ejecución cuando se hayan atendido todos los clientes.


### Respuestas:
a)
```
process cliente[id:0..N-1]{
	txt lista, comprobante;
	gestion.llegada(id,lista,comprobante);
}

process empleado{
	int id, text lista, comprobante;
	for int i: 1 .. N {
		gestion.siguiente(id, lista);
		comprobante = generarComprobante(lista);
		gestion.atencion(id, comprobante);
	}
}

monitor gestion{
	cond hayAlguien, txt ticket[N];
	cond espera
	cola c

	procedure llegada(id: IN txt, lista: IN txt, comprobante: OUT txt){
		c.push(id,lista)
		signal(hayAlguien)
		wait(espera)
		comprobante = ticket[id];
	}

	procedure siguiente(id:OUT int, lista OUT txt){
		if(c.isEmpty()){
			wait(hayAlguien)
		}
		c.pop(id,lista)
	}

	procedure atencion(id: IN int, comprobante: IN txt){
		ticket[id] = comprobante;
		signal(espera)
	}
}
```
b)
```
process cliente[id:0..N-1]{
	txt lista, comprobante;
	gestion.llegada(id,lista,comprobante);
}

process empleado{
	int id, text lista, comprobante;
	while(true){
		gestion.siguiente(id, lista);
		comprobante = generarComprobante(lista);
		gestion.atencion(id, comprobante);
	}
}

monitor gestion{
	cond hayAlguien, txt ticket[N];
	cond espera
	cola c

	procedure llegada(id: IN txt, lista: IN txt, comprobante: OUT txt){
		c.push(id,lista)
		signal(hayAlguien)
		wait(espera)
		comprobante = ticket[id];
	}

	procedure siguiente(id:OUT int, lista OUT txt){
		while(c.isEmpty()){
			wait(hayAlguien)
		}
		c.pop(id,lista)
	}

	procedure atencion(id: IN int, comprobante: IN txt){
		ticket[id] = comprobante;
		signal(espera)
	}
}
```

c) ehh este creo que teniamos otra versión corregida por chichi pero en papel no anoté el monitor xd
```
Process Cliente[id:0..N-1]{
text comprobante;
	text productos;
	Gestion.llegue(id, productos);
	Gestion.recibirComprobante(id, comprobante);
}

Process Empleado(){
	Gestion.sig(lista);
	while(lista != null){
		text comprobante = generarC(lista);
		Gestion.darleComprobante(comprobante);
		Gestion.sig(lista);
	}
}

Monitor Gestion{
	cond espera;
	Cola C[N];
	Cola esperaComprobante;
	cond personaEsperando;
	cond colaEsperaComprobante [N];
	cond quieroComprobante;
	text comprobantes[N];
	int cantidad = N;

	Procedure llegue(id: in int, productos: in text){
		c.push(productos);
		signal(personaEsperando);
	}

	Procedure sig(productos: out text){
		if (c.isEmpty() & cant > 0) {
			wait (personaEsperando);
        } else {
	        if cant == 0 {
	            productos = null;
	            signal_all (personaEsperando);
            } else {
	            cantidad–;
				c.pop(productos);
            }
        }
	}

	Procedure recibirComprobante(id:in int , comprobante: out text) {
		esperaComprobante.push(id);
		signal (quieroComprobante);
		wait(colaEsperaComprobante[id]);
		comprobante=comprobantes[id];
	}

	Procedure darleComprobante(comprobante: in text){
		int idAux;
		while esperaComprobante.isEmpty() {
	        wait (quieroComprobante);
        }
		esperaComprobante.pop(idAux);
		comprobantes[idAux]=comprobante;
		signal(colaEsperaComprobante[idAux]);
	}
}
```

## Ejercicio 6

**Enunciado:**  
Existe una comisión de 50 alumnos que deben realizar tareas de a pares, las cuales son corregidas por un JTP. Cuando los alumnos llegan, forman una fila. Una vez que están todos en fila, el JTP les asigna un número de grupo a cada uno. Para ello, suponga que existe una función AsignarNroGrupo() que retorna un número “aleatorio” del 1 al 25. Cuando un alumno ha recibido su número de grupo, comienza a realizar su tarea. Al terminarla, el alumno le avisa al JTP y espera por su nota. Cuando los dos alumnos del grupo completaron la tarea, el JTP les asigna un puntaje (el primer grupo en terminar tendrá como nota 25, el segundo 24, y así sucesivamente hasta el último que tendrá nota 1). Nota: el JTP no guarda el número de grupo que le asigna a cada alumno.


### Respuestas: En papel lo tengo distinto pero me dio fiaca copiarlo :) pero lo tengo corregido x chichi
```
Process Alumno [id: 0..49]{
	int grupo, nota;
	Tare.llegar (id);
	Tarea.recibirGrupo (id, grupo);
	// Hacer tarea
	Tarea.recibirNota (grupo, nota); 
}

Process JTP {
	int i, grupo;
	int vecContador [25] = ([25], 0);
	int puntaje = 25;
	Tarea.esperarAlumnos();
	for i: 1 .. 50 {
		Tarea.asignarTemas(AsignarNroGrupo());
    }
    for i: 1 .. 50 {
        Tarea.corregirTarea (grupo);
        // Corregir tarea
        vecContador [grupo]++;
        if vecContador[grupo] == 2 {
            Tarea.asignarPuntaje(grupo, puntaje);
            puntaje–;
        }
    }
}

Monitor Tarea {
	Cola espera, esperandoNota;
	int esperando = 0;
	int idAux;
	int vecGrupo [50], vecNota[25];
	cond espera [50], esperaNotas[50];
	cond deboCorregir;

	Procedure llegar(id: in int) {
		esperando++;
		espera.push(id);
		if (esperando == 50) signal (esperaAsignar);
	}

	Procedure esperarAlumnos() {
	    if (esperando < 50) wait (esperaAsignar);
    }

    Procedure recibirGrupo(id: in int, grupo: out int) {
        wait (espera[id]);
        grupo = vecGrupo[id];
    }
        
    Procedure asignarTemas (grupo: in int) {
        espera.pop(idAux);
        vecGrupo [idAux] = grupo;
        signal (espera[idAux]);
    }

    Procedure corregirTarea (grupo: out int) {
        if esperandoNotas.isEmpty() {
            wait (deboCorregir);
        }
        esperandoNotas.pop(grupo);
    }

    Procedure asignarPuntaje (grupo: in int, puntaje: in int) {
        vecNota[grupo] = puntaje;
        signal_all (esperandoNotas[grupo]);
    }

    Procedure recibirNota (grupo: in int, nota: out int) {
        esperandoNota.push(grupo);
        signal(deboCorregir);
        wait (esperaNotas[grupo]);
        nota = vecNota[grupo];
    }
}
```

## Ejercicio 7

**Enunciado:**  
Se debe simular una maratón con C corredores donde en la llegada hay UNA máquina expendedoras de agua con capacidad para 20 botellas. Además, existe un repositor encargado de reponer las botellas de la máquina. Cuando los C corredores han llegado al inicio comienza la carrera. Cuando un corredor termina la carrera se dirigen a la máquina expendedora, espera su turno (respetando el orden de llegada), saca una botella y se retira. Si encuentra la máquina sin botellas, le avisa al repositor para que cargue nuevamente la máquina con 20 botellas; espera a que se haga la recarga; saca una botella y se retira. Nota: mientras se reponen las botellas se debe permitir que otros corredores se encolen.
### Respuestas:
```
Process Corredor [id: 0..C-1] {
	Carrera.llegada();
	// Correr
	Carrera.usoExpendedora(); 
	Expendedora.retirarAgua(); 
	Carrera.retirarse();
}

Process Repositor {
	while (true) {
		Expendedora.recargar();
    }
}

Monitor Carrera {
	bool libre = true;
	int cantEsperando = 0, esperando = 0;
	cond esperaCorredor,comienzoCarrera;

	Procedure llegada() { 
		cantEsperando++;
		if (cantEsperando < 20){ 
			wait (comienzoCarrera);
		}else {
			signal_all (comienzoCarrera);
        }   
    }

    Procedure usoExpendedora() {
        if (not cantEsperando ==0) && (libre) {
			libre = false;
        } else {
            cantEsperando++;
            wait (esperaCorredor);
        }
    }

    Procedure retirarse() {
        if (cantEsperando > 0) {
            cantEsperando–;
            signal (esperaCorredor);
        } else {
            libre = true;
        }
    }
}

Monitor Expendedora () {
	int cantBotellas = 20;
	cond corredor, noHayAgua;

	Procedure retirarAgua() {
	    if (cantBotellas == 0) {
        	signal(noHayAgua)
			wait (corredor);
		}
		cantBotellas --;
	}

    Procedure recargar() {
        if (cantBotellas > 0){ //sino nunca se despierta!
			wait (noHayAgua); 
        	cantBotellas = 20;
        signal (corredor);
    }
}
```

## Ejercicio 8

**Enunciado:**  
En un entrenamiento de fútbol hay 20 jugadores que forman 4 equipos (cada jugador conoce el equipo al cual pertenece llamando a la función DarEquipo()). Cuando un equipo está listo (han llegado los 5 jugadores que lo componen), debe enfrentarse a otro equipo que también esté listo (los dos primeros equipos en juntarse juegan en la cancha 1, y los otros dos equipos juegan en la cancha 2). Una vez que el equipo conoce la cancha en la que juega, sus jugadores se dirigen a ella. Cuando los 10 jugadores del partido llegaron a la cancha comienza el partido, juegan durante 50 minutos, y al terminar todos los jugadores del partido se retiran (no es necesario que se esperen para salir).


### Respuestas: 
chequear este creo que tengo la foto de chichi o Lauti lo tiene pasado
```
Process Jugador [id: 0..19] {
	int miEquipo = …;
	int numeroC;

	Equipo[miEquipo].llegada(numeroC);
	Cancha[numeroC].llegada();
}

Monitor Equipo [id: 0..3] {
	int cant = 0, numCancha;
	cond espera;

	Procedure llegada (cancha: OUT int) {
		cant ++;
        if  (cant < 4) wait (espera)
       	else  {  
            Admin.DarCancha(numCancha);
            signal_all (espera);   
        }
        cancha = numCancha;
    }   
}

Monitor Admin {  
    int cant = 0;

    Procedure DarCancha (num: OUT int) { 
        cant ++;
        if  (cant <= 2) num = 0;
        else num = 1;
    } 
}

Process Partido [id: 0..1] {  
    Cancha[id].Iniciar();
    delay (50minutos); //se juega el partido
    Cancha[id].Terminar();
}

Monitor Cancha [id: 0..1] {
    int cant = 0;
   	cond espera, inicio; 

   	Procedure llegada () { 
        cant ++;
       	if  (cant == 10) signal (inicio);      
        	wait (espera);
    }

    Procedure Iniciar () { 
        if  (cant < 10) wait (inicio);
    }
    
   	Procedure Terminar () { 
        signal_all(espera);
    }
}
```

## Ejercicio 9

**Enunciado:**  
En un examen de la secundaria hay un preceptor y una profesora que deben tomar un examen escrito a 45 alumnos. El preceptor se encarga de darle el enunciado del examen a los alumnos cuando los 45 han llegado (es el mismo enunciado para todos). La profesora se encarga de ir corrigiendo los exámenes de acuerdo con el orden en que los alumnos van entregando. Cada alumno al llegar espera a que le den el enunciado, resuelve el examen, y al terminar lo deja para que la profesora lo corrija y le envíe la nota. Nota: maximizar la concurrencia; todos los procesos deben terminar su ejecución; suponga que la profesora tiene una función corregirExamen que recibe un examen y devuelve un entero con la nota.
### Respuestas:
chequear
```
Process Alumno [id: 0..44] {
	int nota;
	text examen;

	Aula.Llegada (examen);
	// Rendir examen
	Aula.Entrega (id, examen, nota);
}

Process Profesora {
	int i, idAlumno, nota;
	text examen;
	for i: 1 .. 45 {
		Aula.Examen (idAlumno, examen);
		nota = corregirExamen(examen);
		Aula.Corregido (idAlumno, nota);
    }
}

Process Preceptor {
	text enunciado = …;
	Aula.DarEnunciado (enunciado);
}

Monitor Aula {
	int cant = 0, notas [45];
	text enunciado;
	cola C;
	cond eInicio, ePreceptor, eProfesora, eNota;
	
	Procedure Llegada (E: out text) {
		cant++;
		if (cant == 45) signal (ePreceptor);
		wait (eInicio);
		E = enunciado;
    }
	
	Procedure DarEnunciado (E: in text) {
		if (cant < 45) wait (ePreceptor);
		enunciado = E;
		signal_all (eInicio);
    }

    Procedure Entrega (id: in int, e: in text, N: out int) {
        C.push(id, e);
        signal (eProfesora);
        wait (eNota);
        N = notas [id];
    }

    Procedure Examen (idAlumno: out int, E: out text) {
        if c.isEmpty() wait (eProfesora);
        c.pop (idAlumno, E);
    }

    Procedure Corregido (id: in int, nota: in int) {
        notas[id] = nota;
        signal (eNota);
    }
}
```

## Ejercicio 10

**Enunciado:**  
En un parque hay un juego para ser usada por N personas de a una a la vez y de acuerdo al orden en que llegan para solicitar su uso. Además, hay un empleado encargado de desinfectar el juego durante 10 minutos antes de que una persona lo use. Cada persona al llegar espera hasta que el empleado le avisa que puede usar el juego, lo usa por un tiempo y luego lo devuelve. Nota: suponga que la persona tiene una función Usar_juego que simula el uso del juego; y el empleado una función Desinfectar_Juego que simula su trabajo. Todos los procesos deben terminar su ejecución.

### Respuestas:
```
Process Persona [id: 0..N-1] {
	Admin.SolicitarUso();
	Usar_Juego();
	Admin.Liberar();
}

Process Empleado {
	while (true) {
		Desinfectar_Juego();
		Admin.Listo();
    }
}

Monitor Admin {
	cond ePersona, eEmpleado;
	int esperando = 0;
	bool libre = true;

	Procedure SolicitarUso() {
		if (not libre) {
			esperando++;
			wait (esperaAutos);
        } 
        else libre = false;
    }
	
    Procedure Liberar() {
        signal (eEmpleado);
    }

    Procedure Listo () {
        if (esperando > 0) {
            esperando–;
            signal (ePersona);
        } 
        else libre = true;
        wait (eEmpleado);
    }
}
```
