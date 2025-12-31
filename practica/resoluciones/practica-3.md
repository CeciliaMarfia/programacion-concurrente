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

### Respuestas:

## Ejercicio 4

**Enunciado:**  

### Respuestas:

## Ejercicio 5

**Enunciado:**  

### Respuestas:


## Ejercicio 6

**Enunciado:**  

### Respuestas:

## Ejercicio 7

**Enunciado:**  

### Respuestas:
## Ejercicio 8

**Enunciado:**  

### Respuestas:
## Ejercicio 9

**Enunciado:**  

### Respuestas:
## Ejercicio 10

**Enunciado:**  

### Respuestas:
