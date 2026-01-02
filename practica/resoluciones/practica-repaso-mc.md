# Práctica de repaso - Memoria Compartida
#SEMÁFOROS

## Ejercicio 1

**Enunciado:**  
Resolver los problemas siguientes:

a. En una estación de trenes, asisten P personas que deben realizar una carga de su tarjeta SUBE en la terminal disponible. La terminal es utilizada en forma exclusiva por cada persona de acuerdo con el orden de llegada. Implemente una solución utilizando únicamente procesos Persona. Nota: la función UsarTerminal() le permite cargar la SUBE en la terminal disponible.

b. Resuelva el mismo problema anterior pero ahora considerando que hay T terminales disponibles. Las personas realizan una única fila y la carga la realizan en la primera terminal que se libera. Recuerde que sólo debe emplear procesos Persona. Nota: la función UsarTerminal(t) le permite cargar la SUBE en la terminal t.
 
### Respuestas:
a)
```
Cola c;
boolean libre = true;
sem mutex = 1;
sem espera[P] = ([P], 0);

Process Persona [id: 0..P-1] {
	int idAux;
	P (mutex);
	if not libre {
		c.push(id);
		V (mutex);
		P (espera[id]);
    } else {
        libre = false;
        V (mutex);
    }
    UsarTerminal();
    P (mutex);
    if c.isEmpty() {
        libre = true;
    } else {
        c.pop(idAux);
        V (espera[idAux]);
    }
    V (mutex);
}
```
b)
```
Cola c;              // cola de espera de personas
Cola t;              // cola de terminales libres
sem mutex1 = 1;      // protege disponible
sem mutex2 = 1;      // protege cola t
sem espera[P] = ([P], 0);
int vecTerminal[P];
int disponible;
;

Process Persona [id: 0..P-1] {
	P (mutex1);
	if (disponible == 0)){
		c.push(id);
		P(espera[id]);
   } else {
        disponible --;
        V(mutex1)
        P(mutex2);
        t.pop(nroTerminal)
        V(mutex2)
    }
    UsarTerminal (vecTerminal[id]);
    P (mutex1);
    if (c.isEmpty()) {
        disponible ++;
    } else {
        V (mutex1);
        c.pop (idAux);
        V (mutexEspera);
        vecTerminal [idAux] = vecTerminal[id];
        V (espera[idAux]);
    }
     P(mutex2)
     t.push(nroTerminal)
     V(mutex2)
     V(mutex1)
} 
//podríamos tener una opción más concurrente con un arreglo de terminales
```

## Ejercicio 2

**Enunciado:**  

 Implemente una solución para el siguiente problema. Un sistema debe validar un conjunto de 10000 transacciones que se encuentran disponibles en una estructura de datos. Para ello, el sistema dispone de 7 workers, los cuales trabajan colaborativamente validando de a 1 transacción por vez cada uno. Cada validación puede tomar un tiempo diferente y para realizarla los workers disponen de la función Validar(t), la cual retorna como resultado un número entero entre 0 al 9. Al finalizar el procesamiento, el último worker en terminar debe informar la cantidad de transacciones por cada resultado de la función de validación. Nota: maximizar la concurrencia.
### Respuestas:
```
sem mutex = 1;
sem mutexBarrera = 1;
sem contador [10] = ([10], 1);
int vectorContador [10] = ([10], 0);
int termine = 0;

Process Worker [id: 0..6] {
	text transaccion;
	int i, res;
	P (mutex);
	while (!c.isEmpty()) {
		c.pop(transaccion);
		V (mutex);
		res = Validar(transaccion);
		P (contador[res]);
		vectorContador[res]++;
		V (contador[res]);
		P (mutex);
    }
    V (mutex); //libero la SC para que otro acceda y se de cuenta que debe irse
    P (mutexBarrera);
    termine++;
    if (termine == 7) {
        for i: 0 to 9 {
            writeln(´para: ',i, ‘: ’, vectorContador[i]i;
        }
    }
    V (mutexBarrera);//NO OLVIDAR
}
```
## Ejercicio 3

**Enunciado:**  

 Implemente una solución para el siguiente problema. Se debe simular el uso de una máquina expendedora de gaseosas con capacidad para 100 latas por parte de U usuarios. Además, existe un repositor encargado de reponer las latas de la máquina. Los usuarios usan la máquina según el orden de llegada. Cuando les toca usarla, sacan una lata y luego se retiran. En el caso de que la máquina se quede sin latas, entonces le debe avisar al repositor para que cargue nuevamente la máquina en forma completa. Luego de la recarga, saca una botella y se retira. Nota: maximizar la concurrencia; mientras se reponen las latas se debe permitir que otros usuarios puedan agregarse a la fila.


### Respuestas:
```
sem mutex = 1;
sem faltanLatas = 0;
sem hayLatas = 0;
sem espera[U] = ([U], 0);
Cola c;
int cantLatas = 100;
boolean libre = true;

Process Usuario [id: 0..U-1] {
	int idAux;
	P (mutex);
	if (not libre) {
        c.push(id);
        V (mutex);
        P (espera[id]);
    } else {
        libre = false;
        V (mutex);
    }
    if (cantLatas == 0) { //IMPORTANTE: la exclusión mutua ya me lo da lo anterior y la sincronización con el repositor!
        V (faltanLatas);
        P (hayLatas);
    }
    cantLatas-–;
    retirarLata();
    P (mutex);
    if c.isEmpty() {
        libre = true;
    } else {
        c.pop(idAux);
        V (espera[idAux]);
    }
    V (mutex);
}

Process Repositor {
	while (true) {
        P (faltanLatas);
        cantLatas = 100; //recarga de la máquina
        V (hayLatas);
    }
}
```
#MONITORES
## Ejercicio 1

**Enunciado:**  
Resolver el siguiente problema. En una elección estudiantil, se utiliza una máquina para voto electrónico. Existen N Personas que votan y una Autoridad de Mesa que les da acceso a la máquina de acuerdo con el orden de llegada, aunque ancianos y embarazadas tienen prioridad sobre el resto. La máquina de voto sólo puede ser usada por una persona a la vez. Nota: la función Votar() permite usar la máquina.

 
### Respuestas:
```
Process Persona [id: 0..N-1] {
	int edad = …;
	admin.llegar(id, edad);
	Votar();
	admin.salida();	
}

// originalmente había puesto un process autoridadDeMesa pero el profe me dijo que vea el ej. 3 de práctica, y que ojo que el monitor no es la máquina!
Monitor admin {	
	ColaOrdenada c;
	cond espera[N];
	cond hayPersona;

	Procedure llegar (id: in int, edad: in int) {
		c.pushOrdenado(id, edad);
		signal (hayPersona);
		wait (espera[id]);
    }

    Procedure salida() {
        if (c.isEmpty()){
          wait (hayPersona);
        }
        c.pop(id);
        signal (espera[id]);

    }
}
```

## Ejercicio 2

**Enunciado:**  
Resolver el siguiente problema. En una empresa trabajan 20 vendedores ambulantes que forman 5 equipos de 4 personas cada uno (cada vendedor conoce previamente a qué equipo pertenece). Cada equipo se encarga de vender un producto diferente. Las personas de un equipo se deben juntar antes de comenzar a trabajar. Luego cada integrante del equipo trabaja independientemente del resto vendiendo ejemplares del producto correspondiente. Al terminar cada integrante del grupo debe conocer la cantidad de ejemplares vendidos por el grupo. Nota: maximizar la concurrencia.


 
### Respuestas:
```
Process Vendedor [id: 0..19] {
	int equipo = asignarEquipo();
	int cant = 0;
	int cantGrupo;
	Equipo[equipo].armarEquipo();
	cant = VenderEjemplar();
	Equipo[equipo].terminar(cant, cantGrupo);
}

Monitor Equipo [id: 0..4] {
	cond espera;
	int total = 0;
	int cant = 0;

	Procedure armarEquipo() {
		cant++;
		if (cant = 4) {
	        signal_all(espera);
	        cant = 0;
        } else {
	        wait (espera);
        }
    }

    Procedure terminar (cantProd: in int, cantGrupo: out int) {
        total += cantProd;
        cant++;
        if (cant = 4) {
            signal_all(espera);
        } else {
            wait(espera);
        }
        cantGrupo = total;
    }
}
```
## Ejercicio 3

**Enunciado:**  
Resolver el siguiente problema. En una montaña hay 30 escaladores que en una parte de la subida deben utilizar un único paso de a uno a la vez y de acuerdo con el orden de llegada al mismo. Nota: sólo se pueden utilizar procesos que representen a los escaladores; cada escalador usa sólo una vez el paso.
 
### Respuestas:
```
Process Escalador [id: 0..29] {
	Paso.llegar();
	// usar el paso
	Paso.salir();
}

Monitor Paso {
	cond espera;
	boolean libre = true;
	int esperando = 0;

	Procedure llegar() {
		if (not libre) {
			esperando++;
			wait (esperando);
        } else {
	        libre = false;
        }
    }

    Procedure salir() {
        if (esperando > 0) {
            esperando-–;
            signal (esperando);
        } else {
            libre = true;
        }
    }
}

```
